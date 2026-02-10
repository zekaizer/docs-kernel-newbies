# Linux 6.15 Rust 지원 변경사항

> 출처: kernelnewbies.org, Phoronix, LWN.net, rust.docs.kernel.org
> 릴리스: Linux 6.15
> 작성일: 2026-02-10

---

## 개요

Linux 6.15는 Rust 커널 인프라의 구조적 전환점이다. pin-init이 독립 크레이트로 분리되어 모듈성이 향상되었고, hrtimer Rust API로 타이머 프로그래밍이 가능해졌으며, mm_struct/vm_area_struct 바인딩으로 메모리 관리 서브시스템에 대한 Rust 접근이 열렸다. 또한 NOVA DRM 드라이버의 초기 코드가 메인라인에 머지되어, Rust 기반 디바이스 드라이버의 실질적 사례가 등장했다.

---

## pin-init 독립 크레이트

### 변경사항
- **pin-init을 독립 크레이트로 리팩터링** — 기존 커널 크레이트 내부에 있던 pin 초기화 인프라를 별도 크레이트로 분리
- 커널 외부에서도 테스트 및 활용 가능한 구조로 전환

### 배경 및 영향
- pin-init은 `Mutex<T>`와 같은 주소 민감(address-sensitive) 타입의 안전한 초기화를 처리하는 핵심 라이브러리다. `struct mutex` 등 C 커널 구조체를 래핑하는 대부분의 sync 모듈 타입이 자기참조(self-referential) 구조를 포함하므로, pinning이 필수적
- `pin_init!` 매크로는 pinned 구조체의 in-place 초기화자를 생성하며, 기본적으로 `Infallible` 에러 타입을 사용. fallible 초기화가 필요한 경우 `try_pin_init!` 사용
- 큰 구조체의 in-place 초기화를 지원하여 스택 오버플로우를 방지
- 독립 크레이트 전환으로:
  - 커널 외부에서의 단위 테스트가 용이해짐
  - Rust 커널 생태계의 모듈화 촉진
  - 다른 크레이트에서의 의존성 관리가 명확해짐
- Rust Linux 메인테이너 Miguel Ojeda가 처음으로 다른 기여자의 pull request를 직접 받아들인 릴리스로, 서브트리 관리 체계 정비의 시작

---

## hrtimer Rust API

### 변경사항
- **고해상도 타이머(hrtimer) Rust 바인딩** — C `struct hrtimer`를 안전하게 래핑하는 Rust API 제공
- `Arc::as_ptr` 추가, `Arc`에 대한 `HrTimerPointer` 구현
- 타이머 핸들러 내에서 타이머 재시작 가능
- `UnsafeHrTimerPointer`, `ScopedHrTimerPointer` 트레이트 추가
- `Pin<&T>` 및 `Pin<&mut T>`에 대한 `UnsafeHrTimerPointer` 구현

### 배경 및 영향
- hrtimer는 커널에서 나노초 정밀도의 타이머를 제공하는 핵심 인프라로, 드라이버와 서브시스템에서 광범위하게 사용
- `HrTimer` 구조체는 `bindings::hrtimer_setup`으로 초기화되며, `Pin<&HasHrTimer>` 또는 `Pin<&mut HasHrTimer>` 핸들이 존재하는 동안 타이머가 실행 중일 수 있음
- Andreas Hindborg가 개발한 이 패치 시리즈는 Rust 커널 드라이버가 타이밍 기능을 안전하게 사용할 수 있는 기반을 제공
- 6.17에서 `Instant`/`Delta` 타입으로 더 고수준의 시간 API가 추가될 예정

---

## 메모리 관리 바인딩

### 변경사항
- **mm_struct Rust 바인딩** — 프로세스의 메모리 디스크립터를 Rust에서 접근 가능
- **vm_area_struct 바인딩** — 가상 메모리 영역(VMA) 구조체에 대한 Rust 인터페이스
- **mmap 연산 바인딩** — 메모리 매핑 연산을 Rust에서 수행 가능

### 배경 및 영향
- mm_struct와 vm_area_struct는 커널 메모리 관리의 핵심 데이터 구조로, 이 바인딩은 Rust 드라이버가 프로세스 메모리 관리에 참여할 수 있는 기반을 마련
- 특히 GPU 드라이버(NOVA 등)에서 메모리 매핑 관리가 필수적이므로, 이 바인딩은 Rust 기반 GPU 드라이버 개발의 전제 조건
- mmap 연산 지원으로 유저스페이스와 커널 간 공유 메모리 설정이 Rust에서 가능해짐

---

## NOVA DRM 드라이버 (초기 코드)

### 변경사항
- **NOVA DRM 드라이버 초기 코드 메인라인 머지** — Rust 기반 NVIDIA 커널 그래픽/디스플레이 드라이버의 최초 코드
- NVIDIA RTX 2000 "Turing" 시리즈 이후 GPU를 대상으로 하며, 기존 Nouveau 드라이버의 후계자를 목표

### 배경 및 영향
- 메인라인 커널에 Rust 기반 DRM 드라이버 코드가 포함된 최초의 릴리스
- 6.15 시점에서는 아직 초기 단계로 실제 사용은 불가하나, Rust 커널 드라이버의 가능성을 실증
- 6.16에서 DRM Rust 추상화가 확장되며 NOVA 개발이 본격화될 예정

---

## 기타 변경사항

### 변경사항
- DMA 모듈 추가 — 커널 크레이트에 DMA 관련 Rust 모듈 도입
- 에러 핸들링 문서 확충 — Rust 커널 코드의 에러 처리 가이드라인 추가

---

## 이전 버전과의 연속성

| 버전 | Rust 주요 변화 |
|------|---------------|
| 6.1  | Rust 인프라 최초 도입 (최소 지원) |
| 6.12 | sched_ext BPF와 함께 Rust 생태계 확대 |
| 6.13 | PidNamespace, Binder file ops, tracepoints, PCI/platform 드라이버 |
| 6.14 | Extended modversions (Rust + MODVERSIONS 동시 빌드) |
| **6.15** | **pin-init 독립 크레이트, hrtimer API, mm_struct/mmap 바인딩, NOVA 초기 코드** |
| 6.16 | clk, cpumask, cpufreq, DRM 추상화, NOVA 확장 |
| 6.17 | ACPI, platform I/O, hrtimer Instant/Delta |
| 6.18 | debugfs, sg_table, iov_iter, maple tree, PCI, USB |
| 6.19 | Bounded ints, PWM, I2C 드라이버 |

- 6.15는 Rust 커널 인프라의 **구조적 전환점**: pin-init 독립화로 모듈성, hrtimer로 시간 관리, mm 바인딩으로 메모리 관리 영역 진입
- NOVA 초기 코드의 머지는 "Rust가 실제 프로덕션 드라이버에 사용될 수 있다"는 증거
- 매 릴리스마다 Rust 추상화 범위가 확대되는 일관된 트렌드가 계속됨

---

## 참고 자료

- [kernelnewbies.org — Linux 6.15](https://kernelnewbies.org/Linux_6.15)
- [Phoronix — Many Rust Changes Submitted For Linux 6.15](https://www.phoronix.com/news/Linux-6.15-Rust)
- [LWN.net — hrtimer Rust API](https://lwn.net/Articles/1004824/)
- [rust.docs.kernel.org — HrTimer struct](https://rust.docs.kernel.org/kernel/time/hrtimer/struct.HrTimer.html)
- [rust.docs.kernel.org — pin_init macro](https://rust.docs.kernel.org/kernel/prelude/macro.pin_init.html)
- [rust.docs.kernel.org — hrtimer module](https://rust.docs.kernel.org/kernel/time/hrtimer/index.html)
