# Linux 6.18 Rust 지원 변경사항

> 출처: kernelnewbies.org, LWN.net, Phoronix, Collabora
> 릴리스: Linux 6.18
> 작성일: 2026-02-10

---

## 개요

Linux 6.18은 Rust 커널 지원의 결정적 전환점이다. Android Binder 드라이버의 Rust 재작성 병합, Tyr Arm Mali GPU Rust DRM 드라이버 도입, LKMM atomics 바인딩, USB 드라이버 프레임워크 등이 추가되었다. 2025년 12월 커널 개발자 서밋에서 Rust가 실험적 상태에서 커널 핵심 언어로 공식 승격되었다.

---

## Rust Binder 드라이버

### 변경사항
- **Android Binder IPC 드라이버 Rust 재작성** — Linux v6.18-rc1에서 병합. Android의 핵심 프로세스 간 통신 메커니즘의 Rust 구현
- **C 드라이버와 공존** — 빌드 시 C/Rust 드라이버 중 선택 가능. 수 릴리스 동안 검증 후 C 코드 제거 예정
- **원저자: Wedson Almeida Filho, 현재 유지보수자: Alice Ryhl (Google)**

### 배경 및 영향
- Binder는 Android 보안 모델의 핵심. 모든 앱 간 IPC가 Binder를 통과하므로 메모리 안전성이 극히 중요
- Rust의 소유권 시스템과 borrow checker를 통해 use-after-free, 버퍼 오버플로 등의 메모리 취약점 구조적 방지
- **CVE-2025-68260**: Rust Binder에서 발견된 첫 CVE. 경쟁 조건으로 인한 커널 패닉(DoS 등급). 메모리 안전성은 확보되었으나 논리적 경쟁 조건은 Rust로도 완전 방지 불가능함을 보여줌
- C Binder에서 발견되었던 수십 건의 메모리 취약점 대비 구조적 안전성 향상

---

## Tyr — Arm Mali GPU Rust DRM 드라이버

### 변경사항
- **CSF 기반 Arm Mali GPU 드라이버** — Collabora, Arm, Google 공동 개발. 기존 C 기반 Panthor 드라이버의 Rust 포팅
- **RK3588 지원** — Rockchip RK3588의 Mali GPU 전원 관리 및 디바이스 쿼리 가능
- **DRM_IOCTL_PANTHOR_DEV_QUERY 호출 지원** — GPU ROM 정보를 유저스페이스에 노출
- **실험적 상태** — 업스트림은 기본 프로빙/쿼리만 구현. GPUVM 추상화 등 추가 작업 필요

### 배경 및 영향
- 다운스트림 프로토타입은 GNOME, Weston, SuperTuxKart 등에서 Panthor 대비 동등한 성능으로 동작 확인 (Collabora 블로그)
- PanVK Vulkan 드라이버와의 통합을 목표로 동일한 유저스페이스 API 유지
- 장기적으로 Panthor를 대체할 계획이나, 현재 Panthor가 OpenGL ES 3.1 인증 완료 상태이므로 프로덕션에서는 Panthor 계속 사용
- Android 디바이스의 Mali GPU 드라이버 Rust 전환 가능성 시사

---

## LKMM Atomics

### 변경사항
- **Linux Kernel Memory Model 원자적 연산 Rust 바인딩** — C와 Rust 간 통일된 메모리 모델로 원자적 연산 수행 가능
- **Rust-C 상호운용 메모리 모델 통합** — 혼합 코드에서의 메모리 순서 보장

### 배경 및 영향
- LKMM은 커널의 동시성 모델을 정의. Rust 바인딩은 C-Rust 혼합 코드에서의 데이터 레이스 방지에 필수
- 기존에는 Rust 코드가 C 원자적 연산과의 상호운용에서 불확실성이 있었으나, 이제 LKMM 기반의 명확한 의미론 보장

---

## USB 드라이버 프레임워크

### 변경사항
- **USB 드라이버 Rust 바인딩 초기 프레임워크** — USB 서브시스템의 Rust 드라이버 작성 기반 (Phoronix)
- **USB 드라이버 샘플** — Rust 기반 USB 드라이버 예제 코드 포함

### 배경 및 영향
- USB는 가장 다양한 하드웨어를 지원하는 서브시스템 중 하나. Rust 바인딩은 USB 드라이버의 안전성 향상 잠재력
- 6.16의 cpufreq/OPP/DRM 추상화에 이어 USB까지 Rust 생태계 확장

---

## 커널 인프라 추상화

### 변경사항
- **debugfs 바인딩** — 커널 디버그 파일시스템 인터페이스
- **sg_table 및 scatterlist 인프라** — DMA 분산/수집 리스트 바인딩
- **hrtimer 추가 및 시간 지원** — 고해상도 타이머 확장
- **kernel::sync::Refcount** — 참조 카운트 구현
- **struct iov_iter 지원** — I/O 벡터 이터레이터 바인딩
- **정렬 타입 및 비트맵 API** — 메모리 정렬 타입 및 비트맵 조작
- **ID 풀 및 바인딩** — 리소스 ID 관리
- **request_irq 추상화** — 인터럽트 요청 등록 바인딩
- **Maple tree 추상화** — 메모리 관리 핵심 데이터 구조 바인딩 (Nouveau/Nova 드라이버에 필요)
- **PCI Class/Vendor 지원** — PCI 디바이스 식별 지원
- **Box::pin_slice() 구현** — 핀된 슬라이스 할당
- **Pin-init 필드 참조** — 핀 초기화 인프라 확장
- **Instant/Delta 산술 연산** — 시간 관련 타입의 산술 연산
- **BorrowedPage, AsPageIter, VmallocPageIter** — 페이지 관련 추상화

### 배경 및 영향
- 이 추상화들은 Rust 드라이버가 커널의 핵심 인프라에 접근하기 위한 필수 빌딩 블록
- Maple tree 추상화는 Nouveau(NVIDIA) 및 Nova(NVIDIA 차세대) GPU 드라이버 개발에 직접 필요
- 6.16의 cpufreq/OPP/DRM/XArray에 이어 인프라 추상화 범위가 대폭 확대

---

## Rust의 커널 핵심 언어 승격

### 변경사항
- **2025년 12월 커널 개발자 서밋 결정** — Rust가 실험적(experimental) 상태에서 커널의 핵심 언어(core language)로 공식 승격
- **커널의 공식 핵심 언어: C, 어셈블리, Rust**

### 배경 및 영향
- 2022년 6.1에서 초기 Rust 인프라가 도입된 이후 약 3년 만의 공식 승격
- 이 결정은 향후 새 커널 서브시스템이 Rust로 작성될 수 있음을 의미
- Binder와 Tyr의 성공적 병합이 승격 결정의 주요 근거

---

## 이전 버전과의 연속성

- **인프라 성장**: 6.15 pin-init/hrtimer/mm → 6.16 cpufreq/OPP/DRM/XArray/auxiliary bus → 6.18 USB/debugfs/sg_table/iov_iter/maple tree/request_irq. 매 릴리스마다 추상화 범위 확대
- **프로덕션 드라이버**: 6.15~6.16 인프라 준비 → 6.18 실제 프로덕션 드라이버(Binder, Tyr) 병합. 양적에서 질적 전환
- **GPU 드라이버**: 6.16 DRM 추상화 + nova 진행 → 6.18 Tyr Mali Rust DRM 업스트림. ARM Mali GPU 생태계에서 Rust 드라이버 현실화
- **언어 지위**: 6.1 초기 도입 → 6.18 핵심 언어 승격. C와 동등한 지위 확보
- **첫 Rust CVE**: CVE-2025-68260 (Binder)로 Rust가 만능이 아님을 확인. 메모리 안전성은 확보되나 논리적 버그는 별개 문제

---

## 참고 자료

- [kernelnewbies.org — Linux 6.18](https://kernelnewbies.org/Linux_6.18)
- [Collabora — Kernel 6.18: Tyr advances Rust in Linux](https://www.collabora.com/news-and-blog/news-and-events/kernel-618-tyr-advances-rust-in-linux.html)
- [Collabora — Racing karts on a Rust GPU kernel driver](https://www.collabora.com/news-and-blog/news-and-events/racing-karts-on-a-rust-gpu-kernel-driver.html)
- [Rust for Linux — Android Binder Driver](https://rust-for-linux.com/android-binder-driver)
- [Rust for Linux — Tyr GPU Driver](https://rust-for-linux.com/tyr-gpu-driver)
- [Phoronix — Linux 6.18 Lands Initial Framework For USB Driver Rust Bindings](https://www.phoronix.com/news/Linux-6.18-Rust-USB-Framework)
- [CVE-2025-68260 Analysis](https://www.penligent.ai/hackinglabs/cve-2025-68260-the-linux-kernels-first-rust-code-cve-in-rust_binder-root-cause-exposure-checks-and-fix-strategy/)
