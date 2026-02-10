# Linux 6.17 네트워킹 변경사항

> 출처: kernelnewbies.org, LWN.net, LKML, CNX Software
> 릴리스: Linux 6.17
> 작성일: 2026-02-10

---

## 개요

Linux 6.17 네트워킹은 DualPI2 혼잡 제어 프로토콜의 메인라인 통합, MCTP 게이트웨이 라우팅, 멀티패스 TCP의 TCP_MAXSEG 옵션 지원, IPv6 per-interface force_forwarding, RFC 6675 손실 감지 제거가 핵심이다. 6.16의 DMABUF zero-copy 완성에 이어 6.17에서는 지연 최적화(DualPI2) 방향으로 전환했다.

---

## DualPI2 혼잡 제어

### 변경사항
- **DualPI2 메인라인 통합** — L4S(Low Latency, Low Loss, Scalable Throughput) 기반의 이중 PI(Proportional Integral) 큐 혼잡 제어 프로토콜이 메인라인에 포함 (kernelnewbies.org, LWN.net)
- Classic 트래픽과 L4S 트래픽을 단일 큐 내에서 분리 관리하는 AQM(Active Queue Management) 메커니즘

### 배경 및 영향
- DualPI2는 IETF RFC 9332의 구현으로, 기존 TCP와 새로운 L4S(scalable congestion control) 트래픽이 공존할 수 있도록 설계
- 게이밍, 화상회의, 실시간 스트리밍 등 지연 민감 애플리케이션의 네트워크 큐 지연 감소
- ACC-ECN(Accurate ECN)은 2/3 완성(6.18), TCP-Prague는 ACC-ECN 완성 대기 중. 6.17의 DualPI2는 L4S 생태계의 첫 메인라인 구성 요소
- ISP 및 데이터센터의 AQM 정책에서 활용 가능

---

## MCTP 게이트웨이 라우팅

### 변경사항
- **MCTP 게이트웨이 라우팅 지원** — Management Component Transport Protocol에 게이트웨이 라우팅 기능 추가

### 배경 및 영향
- MCTP는 서버/임베디드 플랫폼의 관리 컴포넌트(BMC, 센서, NVMe-MI) 간 통신 프로토콜
- 게이트웨이 라우팅으로 MCTP 네트워크 세그먼트 간 패킷 포워딩 가능
- 데이터센터 서버 관리, 임베디드 시스템 관리에 활용

---

## 멀티패스 TCP

### 변경사항
- **TCP_MAXSEG 소켓 옵션 지원** — 멀티패스 TCP에서 TCP_MAXSEG 소켓 옵션 사용 가능

### 배경 및 영향
- TCP_MAXSEG로 최대 세그먼트 크기를 명시적으로 제어하여, MPTCP 서브플로우의 MTU 관리 최적화
- 모바일 기기의 Wi-Fi + LTE 동시 사용 시나리오에서 최적 세그먼트 크기 제어

---

## IPv6 포워딩 개선

### 변경사항
- **force_forwarding sysctl** — 인터페이스별 IPv6 포워딩을 강제 활성화하는 sysctl 추가

### 배경 및 영향
- 기존의 글로벌 포워딩 설정을 인터페이스 단위로 세분화
- 라우터, VPN 게이트웨이, 컨테이너 네트워크 환경에서 더 유연한 포워딩 정책 설정

---

## TCP 수신 윈도우 제한

### 변경사항
- **TCP 수신 윈도우 제한 적용** — TCP 수신 윈도우 크기 제한 강화

### 배경 및 영향
- 메모리 과다 사용 방지 및 네트워크 공정성 개선

---

## 레거시 프로토콜 정리

### 변경사항
- **RFC 6675 손실 감지 제거** — 2018년 이후 기본값으로 사용되지 않았던 구식 TCP 손실 감지 메커니즘 완전 제거

### 배경 및 영향
- 코드 정리 차원의 변경으로, 현대 TCP 손실 감지(RACK, BBR 등)가 대체

---

## 이전 버전과의 연속성

- **네트워크 최적화 방향 전환**: 6.15 zero-copy 수신 → 6.16 DMABUF TCP TX zero-copy 완성 → 6.17 DualPI2 지연 최적화 (대역폭 → 지연)
- **L4S 생태계 구축**: 6.17 DualPI2 → 6.18 ACC-ECN (부분) → 향후 TCP-Prague로 L4S 스택 완성 예정
- **MPTCP 진화**: 매 릴리스마다 소켓 옵션, 경로 관리 등 MPTCP 기능 확대
- **프로토콜 정리**: 6.16 DCCP 제거 → 6.17 RFC 6675 제거로 레거시 코드 정리 지속

---

## 참고 자료
- [kernelnewbies.org Linux 6.17](https://kernelnewbies.org/Linux_6.17)
- [LWN.net 6.17 merge window](https://lwn.net/Articles/1031713/)
- [CNX Software Linux 6.17 release](https://www.cnx-software.com/2025/09/29/linux-6-17-release-main-changes-arm-risc-v-and-mips-architectures/)
- [GitHub L4STeam/linux (DualPI2)](https://github.com/L4STeam/linux)
