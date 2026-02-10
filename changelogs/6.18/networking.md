# Linux 6.18 네트워킹 변경사항

> 출처: kernelnewbies.org, LWN.net, Phoronix
> 릴리스: Linux 6.18
> 작성일: 2026-02-10

---

## 개요

Linux 6.18 네트워킹은 Google PSP 암호화 프로토콜 도입, TCP Accurate ECN 초기 지원, UDP 수신 50% 성능 향상이 핵심이다. NFS는 서버 I/O 캐싱 비활성화 프로토타입으로 확장성 문제를 해결하고, RWF_DONTCACHE 플래그와 pnfs 대형 extent 배열을 추가했다.

---

## PSP (Protocol Security Protocol)

### 변경사항
- **Google PSP 프로토콜 구현** — TCP 연결에 대한 새 암호화 프로토콜 전체 구현. 문서화 및 보안 고려사항 포함
- **다중 운영 모드** — 터널링 모드를 포함한 여러 운영 모드 지원
- **하드웨어 오프로드 최적화** — IPsec/TLS 대비 우수한 하드웨어 오프로드 능력

### 배경 및 영향
- PSP는 Google이 대규모 데이터센터 네트워크에서 사용하기 위해 개발한 프로토콜
- IPsec보다 유연한 키 관리, TLS보다 낮은 프로토콜 오버헤드를 목표
- 하드웨어 NIC에서의 암호화/복호화 오프로드가 핵심 설계 목표
- Azure/Google Cloud 등 대규모 클라우드 환경에서의 네트워크 암호화 표준화에 기여 가능

---

## TCP Accurate ECN

### 변경사항
- **RFC 9768 기반 초기 지원** — Accurate Explicit Congestion Notification의 초기 구현
- **라운드트립당 다중 혼잡 피드백** — 기존 ECN의 RTT당 단일 피드백 제한을 극복하여 RTT당 다중 혼잡 신호 전달

### 배경 및 영향
- 기존 ECN은 RTT당 최대 하나의 혼잡 신호만 전달 가능하여 고속 네트워크에서 혼잡 응답의 정밀도가 제한적
- Accurate ECN은 데이터센터 네트워크(DCTCP 등)에서의 혼잡 제어 정밀도를 크게 향상
- 초기 지원 단계로, 후속 릴리스에서 기능 확장 예상

---

## UDP 수신 성능 향상

### 변경사항
- **50% 추가 성능 개선** — UDP 수신 경로의 대폭적 최적화 (kernelnewbies.org)
- **경합 감소** — 락 경합 및 경쟁 조건 감소
- **NUMA 인식 락킹** — NUMA 토폴로지를 고려한 락 배치
- **데이터 구조 레이아웃 최적화** — 캐시 라인 정렬 및 데이터 구조 바이너리 레이아웃 재구성
- Phoronix는 최대 47%의 수신 성능 향상을 보고

### 배경 및 영향
- UDP는 DNS, QUIC, 게이밍, VoIP, 스트리밍 등 저지연 통신의 핵심 프로토콜
- NUMA 인식 락킹은 멀티소켓 서버에서의 UDP 성능 병목을 해소
- 극한 조건(extreme conditions)에서 50% 향상으로, 일반적 사용에서도 유의미한 개선 기대

---

## NFS 확장성

### 변경사항
- **I/O 캐싱 비활성화 프로토타입** — NFS 서버에서 I/O 캐싱을 비활성화하여 저메모리 시스템에서 운용 가능하게 하고 대규모 워크로드의 캐시 쓰래싱 감소
- **RWF_DONTCACHE 플래그** — 클라이언트에서 preadv2()/pwritev2()에 캐시 회피 플래그 지원
- **Direct I/O 정렬** — LOCALIO의 정렬 불일치 DIO 연산 처리 개선
- **STATX_DIOALIGN 지원** — NFSD filecache에서 Direct I/O 정렬 요구사항 노출
- **pnfs 대형 extent 배열** — 대규모 파일 레이아웃 처리를 위한 확장
- **flexfiles 스트라이프 레이아웃** — flexfiles 레이아웃 드라이버에서 스트라이프 구성 지원

### 배경 및 영향
- NFS 서버의 I/O 캐싱 비활성화는 접근 방식의 근본적 전환. 대규모 데이터셋에서 캐시가 오히려 성능을 저하시키는 문제 해결
- 6.16의 4MB I/O 크기 확대에 이어 서버 확장성에 초점을 맞춘 개선

---

## SMB/CIFS 네트워크 파일시스템

### 변경사항
- **smbdirect 소켓 통합** — 클라이언트/서버 간 RDMA 소켓 코드 대규모 리팩토링
- **ksmbd max IP connections** — 서버의 IP별 최대 연결 수 제한

### 배경 및 영향
- smbdirect 통합은 RDMA 네트워크 지원의 유지보수성 향상
- 연결 제한은 리소스 관리 및 보안에 기여

---

## 이전 버전과의 연속성

- **네트워크 암호화**: 6.16 DMABUF TCP TX zero-copy → 6.18 PSP 프로토콜. 데이터센터 네트워크 암호화의 새로운 접근
- **UDP 성능**: 지속적인 UDP 스택 최적화. 6.18의 50% 향상은 단일 릴리스에서의 최대 개선 중 하나
- **혼잡 제어**: TCP Accurate ECN 초기 지원으로 차세대 혼잡 제어 메커니즘 기반 마련
- **NFS**: 6.16의 4MB I/O + localio sysfs → 6.18 서버 캐싱 비활성화 + RWF_DONTCACHE + pnfs 확장. 서버 확장성에 초점 전환

---

## 참고 자료

- [kernelnewbies.org — Linux 6.18](https://kernelnewbies.org/Linux_6.18)
- [Phoronix — Linux 6.18 Released](https://www.phoronix.com/news/Linux-6.18-Released)
- [CNX Software — Linux 6.18 release](https://www.cnx-software.com/2025/12/01/linux-6-18-release-main-changes-arm-risc-v-and-mips-architectures/)
