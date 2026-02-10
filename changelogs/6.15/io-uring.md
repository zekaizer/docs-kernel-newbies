# Linux 6.15 io_uring 변경사항

> 출처: kernelnewbies.org, LWN.net, Phoronix
> 릴리스: Linux 6.15
> 작성일: 2026-02-10

---

## 개요

Linux 6.15는 io_uring의 네트워킹 성능을 근본적으로 향상시키는 릴리스다. Zero-copy 수신(zcrx) 도입으로 단일 CPU 코어에서 200Gbps 링크 포화가 가능해졌고, epoll 이벤트 reaping으로 레거시 이벤트 루프의 점진적 마이그레이션 경로를 제공하며, vectored registered buffer로 ublk/stripe 워크로드 성능을 개선했다.

---

## Zero-Copy 수신 (zcrx)

### 변경사항
- **네트워크 zero-copy 수신 지원** — NIC에서 DMA를 통해 애플리케이션 메모리로 직접 패킷 데이터를 전달하여 커널-유저 메모리 복사를 제거
- 유저스페이스에서 호스트 메모리 영역을 등록하여 NIC가 직접 DMA로 데이터를 수신
- 6.15에서는 호스트 메모리만 지원; GPU 메모리(DMA-BUF)는 6.16에서 추가 예정

### 배경 및 영향
- io_uring zcrx의 핵심 목표는 고속 네트워킹에서 커널 메모리 복사 오버헤드를 완전히 제거하는 것이다
- **성능 벤치마크** (LWN, AMD EPYC 9454 + Broadcom BCM957508 200G NIC 기준):
  - io_uring zcrx: **116.2 Gbps**
  - epoll 기반: **82.2 Gbps**
  - **+41% 성능 향상** (애플리케이션 스레드와 net rx softirq가 다른 코어에 핀된 경우)
- Jens Axboe(Meta)는 "커널 메모리에서 데이터를 복사할 필요 없이, 애플리케이션 메모리로 직접 대량 수신이 가능"하다고 설명
- 단일 CPU 코어에서 200G 링크를 포화시키는 시연이 이루어짐
- 기존 zero-copy 솔루션과 달리 메모리 정렬이나 복잡한 mmap 연산이 불필요하여 사용이 용이
- 일부 사용 사례에서 DPDK를 대체할 수 있는 잠재력 — 커널을 우회하지 않으면서도 동등한 성능 제공

### 주요 기여자
- Pavel Begunkov, David Wei, Jakub Kicinski

---

## Epoll 이벤트 Reaping

### 변경사항
- **io_uring를 통한 epoll 이벤트 읽기 지원** — `IORING_OP_EPOLL_WAIT` 연산 추가
- 기존 epoll 이벤트 루프를 io_uring의 completion 모델로 부분 전환 가능
- readiness 기반 이벤트 타입은 여전히 epoll로 reap하되, 전체 대기는 io_uring이 담당

### 배경 및 영향
- 많은 기존 애플리케이션이 epoll 기반 이벤트 루프를 사용 중이며, io_uring으로의 전면 전환은 대규모 리팩토링을 요구
- epoll reaping 지원으로 **점진적 마이그레이션 경로** 제공 — 레거시 readiness 기반 이벤트와 io_uring completion 모델의 공존 가능
- 이는 io_uring 채택의 실질적 장벽을 낮추는 전략적 기능

---

## Vectored Registered Buffer

### 변경사항
- **커널 bvec용 vectored fixed buffer 지원** — scatter/gather I/O를 위한 등록된 버퍼의 벡터화 연산 지원
- **ublk zero-copy 지원** — ublk 및 stripe 워크로드에서 vectored fixed buffer 활용
- 커널 공간 버퍼 벡터 관리 개선

### 배경 및 영향
- Vectored registered buffer는 단일 I/O 연산에서 여러 비연속 메모리 영역을 효율적으로 처리
- ublk(userspace block device)는 io_uring 기반 블록 장치 프레임워크로, zero-copy 지원으로 불필요한 데이터 복사 제거
- 등록된 버퍼 사용 시 벤치마크에서 최대 **2.05배** 성능 향상이 보고됨 (PostgreSQL case study: 11-15% throughput 향상)

---

## 이전 버전과의 연속성

| 버전 | io_uring 주요 변화 |
|------|-------------------|
| 6.13 | Ring resizing, fixed wait regions, hybrid IO polling, static NAPI |
| 6.14 | FUSE io_uring 통신, 컨텍스트 스위치 감소 |
| **6.15** | **zero-copy 수신 (zcrx), epoll reaping, vectored registered buffer** |
| 6.16 | DMA-BUF zcrx 확장 예정 |

- 6.13의 ring resizing과 fixed wait regions이 io_uring의 유연성을 높였고, 6.14에서 FUSE와의 통합으로 활용 범위를 확대
- 6.15의 zero-copy 수신은 io_uring을 고성능 네트워킹의 핵심 인프라로 격상시킨 전환점
- epoll reaping은 io_uring 채택을 가속화하기 위한 호환성 전략

---

## 참고 자료

- [kernelnewbies.org — Linux 6.15](https://kernelnewbies.org/Linux_6.15)
- [LWN.net — io_uring zero copy rx](https://lwn.net/Articles/994603/) — 벤치마크 수치 (116.2 Gbps vs 82.2 Gbps)
- [LWN.net — Zero copy Rx using io_uring](https://lwn.net/Articles/965214/) — 패치 시리즈 분석
- [Phoronix — IO_uring Network Zero-Copy Receive Lands In Linux 6.15](https://www.phoronix.com/news/Linux-6.15-IO_uring)
- [Phoronix — IO_uring ZCRX DMA-BUF for 6.16](https://www.phoronix.com/news/IO_uring-ZCRX-DMA-BUF)
- [Linux Kernel Documentation — iou-zcrx](https://docs.kernel.org/networking/iou-zcrx.html)
