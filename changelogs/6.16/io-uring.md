# Linux 6.16 io_uring 변경사항

> 출처: kernelnewbies.org, LWN.net, LKML
> 릴리스: Linux 6.16
> 작성일: 2026-02-10

---

## 개요

Linux 6.16의 io_uring은 다중 네트워크 큐 지원, DMABUF 통합 zero-copy RX 확장, 파이프 생성 기능 추가가 핵심이다. 6.15의 zero-copy 수신 기반 위에 다중 큐 아키텍처를 도입하여 네트워크 I/O 확장성을 크게 높였다.

---

## 다중 네트워크 큐 (Multiple ifqs)

### 변경사항
- **io_uring 컨텍스트당 다중 네트워크 큐 지원** — 6개 커밋으로 io_uring에서 다중 네트워크 인터페이스 큐를 사용하는 아키텍처 구현 (kernelnewbies.org)
- 단일 io_uring 컨텍스트에서 여러 네트워크 큐를 동시에 관리 가능

### 배경 및 영향
- 고성능 네트워크 서버에서 다중 NIC 큐(RSS/RFS)를 io_uring으로 효율적으로 처리
- 기존에는 io_uring 컨텍스트 하나당 하나의 네트워크 큐만 사용 가능했으나, 다중 큐로 네트워크 처리량 확장
- DPDK 대비 커널 네트워크 스택의 경쟁력 강화

---

## DMABUF Zero-Copy RX (zcrx) 확장

### 변경사항
- **DMABUF 통합 zero-copy 수신 확장** — 6.15에서 도입된 zero-copy RX 기능을 DMABUF과 통합하여 디바이스 메모리에서의 직접 수신 확장

### 배경 및 영향
- GPU/가속기 메모리에서 네트워크 데이터를 직접 수신하는 경로의 DMABUF 통합
- 6.16에서 DMABUF TCP TX도 추가되어 양방향 디바이스 메모리 네트워킹이 완성
- AI/ML 학습 클러스터에서 GPU 간 네트워크 통신 시 CPU 메모리 복사 제거

---

## 파이프 생성

### 변경사항
- **io_uring에서 파이프 생성** — io_uring 컨텍스트 내에서 파이프를 생성하는 기능 추가 (LWN.net)

### 배경 및 영향
- 기존에는 pipe() 시스콜을 별도로 호출해야 했으나, io_uring 내에서 비동기로 파이프 생성 가능
- 프로세스 간 통신 파이프의 설정을 io_uring의 submission queue로 통합하여 시스콜 오버헤드 감소
- io_uring이 파일시스템 연산(open, close, stat 등)에 이어 IPC 프리미티브까지 비동기 범위를 확장하는 트렌드

---

## 이전 버전과의 연속성

- **io_uring zero-copy**: 6.12 DMABUF TCP RX 도입 → 6.15 io_uring zero-copy 수신 → 6.16 다중 ifq + DMABUF zcrx 확장 + TX 완성
- **io_uring 기능 확장**: 매 릴리스마다 새 연산 추가. 6.15 epoll 이벤트 읽기 → 6.16 파이프 생성으로 비동기 범위 계속 확대
- **io_uring 네트워킹**: 6.16에서 다중 네트워크 큐 아키텍처 도입으로 고성능 네트워크 서버에서의 io_uring 활용 기반 확립

---

## 참고 자료

- [kernelnewbies.org — Linux 6.16](https://kernelnewbies.org/Linux_6.16)
- [LWN.net — The first half of the 6.16 merge window](https://lwn.net/Articles/1022512/)
