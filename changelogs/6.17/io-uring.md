# Linux 6.17 io_uring 변경사항

> 출처: kernelnewbies.org, LWN.net, LKML
> 릴리스: Linux 6.17
> 작성일: 2026-02-10

---

## 개요

Linux 6.17의 io_uring은 전송 타임스탬프 명령, 벡터화된 전송(vectorized send), multishot 수신 길이 제한, 테스트용 mock 파일 인프라가 핵심이다. 6.16의 대규모 네트워크 확장(다중 큐, DMABUF zcrx)에 비해 안정화 및 기능 보완 중심의 릴리스이다.

---

## 전송 타임스탬프 (Transmit Timestamp)

### 변경사항
- **전송 타임스탬프 명령** — io_uring에서 네트워크 패킷 전송 시 하드웨어/소프트웨어 타임스탬프 수집 지원

### 배경 및 영향
- 네트워크 지연 측정, PTP(Precision Time Protocol), 고빈도 트레이딩 등 정밀 타이밍이 필요한 워크로드에 활용
- 기존 sendmsg + SO_TIMESTAMPING 조합의 비동기 대안으로, io_uring의 zero-copy 전송 경로와 결합 시 효과적

---

## 벡터화된 전송 (Vectorized Send)

### 변경사항
- **vectorized send 지원** — 다수의 버퍼를 단일 전송 연산으로 처리하는 scatter-gather 전송

### 배경 및 영향
- writev()/sendmsg()의 비동기 io_uring 대안. 프로토콜 헤더와 페이로드를 별도 버퍼로 관리하면서 단일 시스콜로 전송
- 네트워크 서버, 프록시, 스트리밍 서비스의 성능 최적화에 활용

---

## Multishot 수신 길이 제한

### 변경사항
- **multishot receive length cap** — multishot 수신 연산에 최대 길이 제한 설정 가능

### 배경 및 영향
- Multishot 수신은 단일 SQE(Submission Queue Entry)로 다수의 수신 완료를 처리하는 고효율 메커니즘
- 길이 제한 추가로 메모리 과다 사용 방지 및 더 세밀한 리소스 관리 가능

---

## Mock 파일 인프라

### 변경사항
- **테스트용 mock 파일 인프라** — io_uring 테스트를 위한 가상 파일 인프라 도입

### 배경 및 영향
- io_uring의 테스트 커버리지 향상을 위한 인프라. 실제 디바이스 없이 다양한 I/O 시나리오 테스트 가능
- 커널 CI/CD에서의 io_uring 회귀 테스트 안정성 향상

---

## NOP_TW 완료 플래그

### 변경사항
- **NOP_TW completion flag** — task_work 기반 NOP 완료 플래그 추가

### 배경 및 영향
- io_uring 내부 메커니즘 최적화. task_work 완료 경로의 효율성 향상

---

## 이전 버전과의 연속성

- **io_uring 네트워크 확장**: 6.15 zero-copy 수신/vectored registered buffer → 6.16 다중 큐/DMABUF zcrx/파이프 생성 → 6.17 전송 타임스탬프/vectorized send (안정화 중심)
- **io_uring 기능 범위**: 6.17은 대규모 기능 추가보다 기존 기능의 완성도 향상에 초점
- **테스트 인프라**: mock 파일 도입으로 향후 기능 추가 시 품질 보증 강화

---

## 참고 자료
- [kernelnewbies.org Linux 6.17](https://kernelnewbies.org/Linux_6.17)
- [LWN.net 6.17 merge window](https://lwn.net/Articles/1031713/)
