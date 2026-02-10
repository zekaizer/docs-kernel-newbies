# Linux 6.17 Rust 지원 변경사항

> 출처: kernelnewbies.org, LWN.net, LKML
> 릴리스: Linux 6.17
> 작성일: 2026-02-10

---

## 개요

Linux 6.17의 Rust 지원은 regulator 서브시스템, firmware properties, I/O 리소스/메모리, ACPI 매치 테이블 등 하드웨어 플랫폼 추상화의 지속적 확대가 핵심이다. 6.16의 cpufreq/OPP/DRM 추상화에 이어, 6.17에서는 전압 레귤레이터와 디바이스 트리/ACPI 프로퍼티 접근으로 실질적 SoC 드라이버 작성 기반을 강화했다.

---

## Regulator 서브시스템

### 변경사항
- **regulator 추상화** — 전압/전류 레귤레이터 서브시스템의 Rust 바인딩 (LWN.net, CNX Software)
- 레귤레이터 활성화/비활성화, 전압 설정, 상태 조회 등 기본 API

### 배경 및 영향
- SoC 드라이버에서 전원 레일(power rail) 관리는 필수. PMIC(Power Management IC) 제어의 Rust 기반 구현 가능
- 6.16의 cpufreq/OPP/clk에 이어 regulator 추가로, Rust 기반 전력 관리 드라이버의 전체 스택이 거의 완성
- Android SoC에서 PMIC 드라이버 개발 시 Rust 활용 가능성 확대

---

## Firmware Properties

### 변경사항
- **디바이스 프로퍼티 읽기 바인딩** — 디바이스 트리(DT) 및 ACPI 기반 디바이스 프로퍼티 접근 API (kernelnewbies.org)

### 배경 및 영향
- 커널 드라이버에서 하드웨어 설정 정보(GPIO 번호, 클럭 주파수, 인터럽트 번호 등)를 읽는 핵심 API
- DT 프로퍼티 접근은 ARM SoC 드라이버의 기본 요소. 이 바인딩 없이는 실질적 SoC 드라이버 작성이 불가능
- ACPI 매치 테이블과 결합하여 x86/ARM 플랫폼 모두에서 디바이스 식별 가능

---

## I/O 리소스 및 I/O 메모리

### 변경사항
- **플랫폼 I/O 지원** — I/O 리소스(port I/O, MMIO) 접근을 위한 Rust 바인딩 (kernelnewbies.org, CNX Software)
- **I/O 메모리 매핑** — ioremap 등 메모리 매핑 I/O 추상화

### 배경 및 영향
- 하드웨어 레지스터 접근(MMIO readl/writel)은 모든 디바이스 드라이버의 기본. 이 바인딩은 Rust 드라이버의 물리적 하드웨어 제어를 가능하게 함
- 6.16의 DRM/cpufreq 추상화가 상위 수준 프레임워크였다면, I/O 리소스는 가장 기본적인 하드웨어 접근 레이어

---

## ACPI 매치 테이블

### 변경사항
- **ACPI 디바이스 매치 테이블** — ACPI 기반 디바이스 식별을 위한 매치 테이블 지원 (kernelnewbies.org)

### 배경 및 영향
- ACPI 매치 테이블은 x86 플랫폼에서 디바이스를 드라이버에 바인딩하는 기본 메커니즘
- ARM 서버 및 ACPI 지원 ARM 플랫폼(일부 서버급 SoC)에서도 활용

---

## 유틸리티 및 기타

### 변경사항
- **bits/genmask 매크로** — 비트 필드 접근을 위한 기본 매크로 지원
- **Borrow/BorrowMut 구현** — Rust 표준 차용 트레이트 구현
- **hrtimer Instant/Delta 변환** — 고해상도 타이머의 시간 표현 변환
- **strncpy_from_user** — 유저스페이스에서 커널로 문자열 복사 uaccess 추가
- **Bug/warn 추상화** — 커널의 BUG()/WARN() 매크로 Rust 래퍼

### 배경 및 영향
- bits/genmask는 하드웨어 레지스터의 비트 필드 조작에 필수적인 매크로
- strncpy_from_user는 유저스페이스와의 데이터 교환에 필요한 uaccess 기본 함수
- Bug/warn 추상화로 Rust 드라이버에서도 커널의 에러 보고 체계 활용 가능

---

## 이전 버전과의 연속성

- **Rust 추상화 확대 경로**:
  - 6.15: pin-init 독립 크레이트, hrtimer API, mm_struct/vm_area_struct/mmap
  - 6.16: cpufreq/OPP/clk/cpumask (전력 관리), DRM (GPU), XArray, auxiliary bus
  - 6.17: regulator (전압 관리), firmware properties (DT/ACPI), I/O 리소스/메모리 (하드웨어 접근), ACPI 매치 테이블
- **SoC 드라이버 작성 가능성**: 6.17 기준으로 cpufreq + OPP + clk + regulator + firmware properties + I/O 메모리가 모두 갖춰져, 간단한 SoC 플랫폼 드라이버를 Rust로 작성하는 것이 이론적으로 가능해짐
- **향후 전망**: nova GPU 드라이버의 실질적 기능 구현, 네트워킹/스토리지 서브시스템 추상화 확대 예상

---

## 참고 자료
- [kernelnewbies.org Linux 6.17](https://kernelnewbies.org/Linux_6.17)
- [LWN.net 6.17 merge window](https://lwn.net/Articles/1031713/)
- [CNX Software Linux 6.17 release](https://www.cnx-software.com/2025/09/29/linux-6-17-release-main-changes-arm-risc-v-and-mips-architectures/)
