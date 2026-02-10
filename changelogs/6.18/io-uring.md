# Linux 6.18 io_uring 변경사항

> 출처: kernelnewbies.org, Phoronix
> 릴리스: Linux 6.18
> 작성일: 2026-02-10

---

## 개요

Linux 6.18 io_uring은 mixed-size CQE(가변 크기 완료 큐 엔트리), uring_cmd multishot + provided buffers, zcrx 업데이트, io_uring 기능 조회, 요청 포이즈닝 등을 추가했다. 6.16의 다중 네트워크 큐 + DMABUF zcrx에 이어 기능 확장이 지속된다.

---

## Mixed-size CQE

### 변경사항
- **가변 크기 완료 큐 엔트리** — 동일 io_uring 인스턴스에서 서로 다른 크기의 CQE를 혼합하여 사용 가능

### 배경 및 영향
- 기존에는 인스턴스 생성 시 CQE 크기를 고정(일반 16바이트 또는 확장 32바이트)으로 결정
- Mixed-size CQE로 필요에 따라 연산별로 다른 크기의 완료 데이터를 반환 가능
- 확장 CQE가 필요한 일부 연산과 일반 CQE로 충분한 연산을 동일 링에서 효율적으로 처리

---

## uring_cmd Multishot

### 변경사항
- **uring_cmd multishot + provided buffers** — uring_cmd에서 multishot 모드와 provided buffers 조합 지원

### 배경 및 영향
- uring_cmd는 드라이버별 커스텀 명령을 io_uring으로 실행하는 인터페이스
- Multishot은 단일 SQE(제출 큐 엔트리)로 다수의 완료를 생성
- provided buffers와 결합하여 사용자가 미리 등록한 버퍼에 반복적으로 결과를 기록
- NVMe passthrough, 커스텀 디바이스 드라이버 등에서 활용 가능

---

## zcrx (Zero-Copy Receive) 업데이트

### 변경사항
- **zcrx 기능 확장** — io_uring zero-copy 수신 경로 업데이트

### 배경 및 영향
- zcrx는 6.15에서 도입되어 6.16에서 DMABUF 통합으로 확장된 zero-copy 네트워크 수신 메커니즘
- 6.18에서 추가 안정화 및 기능 개선
- DPDK 등 유저스페이스 네트워킹 대비 커널 기반 zero-copy의 경쟁력 강화

---

## io_uring 기능 조회

### 변경사항
- **io_uring 쿼리 기능** — io_uring이 지원하는 기능을 런타임에 조회하는 인터페이스

### 배경 및 영향
- 애플리케이션이 커널의 io_uring 지원 수준을 동적으로 확인 가능
- 커널 버전별로 다른 io_uring 기능 가용성에 대한 안전한 기능 검출(feature detection)
- 라이브러리(liburing) 및 애플리케이션의 호환성 관리 용이

---

## 요청 포이즈닝

### 변경사항
- **요청 포이즈닝(request poisoning)** — 완료된 요청의 메모리를 의도적으로 무효화하여 재사용 방지

### 배경 및 영향
- 보안 및 디버깅 목적. 완료된 요청을 재사용하려는 버그를 조기 감지
- use-after-free 유형의 버그 방지에 기여

---

## 이전 버전과의 연속성

- **io_uring 네트워크**: 6.15 zero-copy 수신 → 6.16 다중 네트워크 큐 + DMABUF zcrx → 6.18 zcrx 업데이트. zero-copy 네트워크 경로 안정화
- **CQE 진화**: 6.18 mixed-size CQE로 완료 큐의 유연성 향상
- **드라이버 통합**: uring_cmd multishot + provided buffers로 드라이버 커스텀 명령의 효율적 다중 완료 지원
- **기능 조회**: io_uring 기능 조회 인터페이스로 버전 호환성 관리 체계화

---

## 참고 자료

- [kernelnewbies.org — Linux 6.18](https://kernelnewbies.org/Linux_6.18)
- [Phoronix — Linux 6.18 Features](https://www.phoronix.com/news/Linux-6.18-Features-Expected)
