# Linux 6.16 네트워킹 변경사항

> 출처: kernelnewbies.org, LWN.net, LKML
> 릴리스: Linux 6.16
> 작성일: 2026-02-10

---

## 개요

Linux 6.16 네트워킹은 DMABUF 메모리에서의 TCP zero-copy 전송 완성, OpenVPN DCO 커널 드라이버 도입, BPF 기반 qdisc 구현, 그리고 DCCP 프로토콜 제거가 핵심이다. 디바이스 메모리와 네트워크 스택의 통합이 양방향으로 완성된 중요한 릴리스다.

---

## DMABUF TCP TX (Zero-Copy 전송)

### 변경사항
- **DMABUF 메모리에서 TCP zero-copy 전송** — GPU/가속기 DMABUF 영역에서 TCP 패이로드를 zero-copy로 전송. 헤더는 별도 커널 버퍼에서 처리 (kernelnewbies.org)
- 6.12에서 도입된 DMABUF TCP RX(수신) 경로에 이어 TX(송신) 경로 추가로 양방향 zero-copy 완성

### 배경 및 영향
- GPU 연산 결과나 가속기 출력을 네트워크로 전송할 때 CPU 메모리 복사 없이 직접 전송 가능
- AI/ML 학습 클러스터, RDMA 대체 시나리오, GPU 렌더링 결과 스트리밍 등에서 레이턴시 및 CPU 오버헤드 대폭 감소
- 패이로드와 헤더 분리 설계로 네트워크 스택의 프로토콜 처리를 유지하면서도 데이터 복사 제거

---

## OpenVPN DCO (Data Channel Offload)

### 변경사항
- **OpenVPN 커널 오프로드 드라이버** — OpenVPN 데이터 채널 처리를 커널 내에서 수행하는 가상 네트워크 디바이스
- 유저스페이스에서의 암호화/복호화 오버헤드를 커널로 이전하여 VPN 성능 향상

### 배경 및 영향
- WireGuard가 커널 내장 VPN으로 존재하는 것처럼, OpenVPN도 커널 오프로드를 통해 성능 개선
- 기존 OpenVPN은 유저스페이스에서 모든 패킷 처리를 수행하여 컨텍스트 스위칭 오버헤드가 컸음
- DCO로 데이터 경로가 커널에서 처리되어 처리량 향상 및 레이턴시 감소

---

## BPF qdisc

### 변경사항
- **BPF struct_ops 기반 트래픽 제어 큐잉 디시플린** — 10개 커밋으로 BPF 프로그램이 커널의 qdisc를 구현할 수 있는 프레임워크 추가
- 기존 커널 내장 qdisc(fq, htb, tbf 등)와 동등한 수준의 트래픽 제어를 BPF로 프로그래밍 가능

### 배경 및 영향
- 네트워크 트래픽 제어의 프로그래밍 가능성을 크게 확대. 커널 재빌드 없이 커스텀 스케줄링 알고리즘 배포 가능
- 클라우드 환경에서의 네트워크 QoS 정책을 동적으로 적용하는 데 유용
- sched_ext가 CPU 스케줄러를 BPF로 확장한 것처럼, qdisc 영역에서의 유사한 확장

---

## DCCP 제거

### 변경사항
- **DCCP 프로토콜 지원 완전 제거** — 2005년 도입된 Datagram Congestion Control Protocol이 20년 만에 제거

### 배경 및 영향
- DCCP는 실시간 스트리밍을 위한 혼잡 제어 데이터그램 프로토콜로 설계되었으나, 실제 채택이 거의 없었음
- 커널 코드 정리 및 공격 표면 감소에 기여
- 장기간 deprecated 상태였으므로 실질적 영향은 미미

---

## 기타 네트워킹 변경사항

### 변경사항
- **AFS GSSAPI 인증** — AFS 파일시스템에서 YFS/OpenAFS 서버 연결을 위한 GSSAPI 인증 지원
- **Unix-domain 소켓 FD 전송 옵트아웃** — 프로그램이 파일 디스크립터 전송을 거부하여 DoS 방지 가능
- **TCP/UDP 최적화** — 다수의 프로토콜 스택 개선 및 드라이버 API 업데이트
- **NFS I/O 크기 확대** — 읽기/쓰기 크기 제한 4MB (네트워크 파일시스템 관련)

### 배경 및 영향
- Unix-domain 소켓 FD 옵트아웃은 권한 있는 소켓 서버가 악의적 FD flooding 공격을 방지하는 보안 기능
- AFS GSSAPI는 대규모 분산 파일시스템 환경에서의 인증 인프라 현대화

---

## 이전 버전과의 연속성

- **DMABUF TCP**: 6.12 RX zero-copy 도입 → 6.16 TX zero-copy 완성으로 양방향 디바이스 메모리 네트워킹 실현
- **BPF 네트워킹**: XDP → TC BPF → 6.16 qdisc BPF로 네트워크 스택 프로그래밍 범위 계속 확대
- **VPN 커널 통합**: WireGuard(5.6) → OpenVPN DCO(6.16)로 주요 VPN 솔루션의 커널 오프로드 확산

---

## 참고 자료

- [kernelnewbies.org — Linux 6.16](https://kernelnewbies.org/Linux_6.16)
- [LWN.net — The second half of the 6.16 merge window](https://lwn.net/Articles/1023075/)
- [Phoronix — Linux 6.16 Features](https://www.phoronix.com/news/Linux-6.16-Features-Early-Look)
