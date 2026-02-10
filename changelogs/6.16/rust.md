# Linux 6.16 Rust 지원 변경사항

> 출처: kernelnewbies.org, LWN.net, LKML
> 릴리스: Linux 6.16
> 작성일: 2026-02-10

---

## 개요

Linux 6.16의 Rust 지원은 cpufreq/OPP/clk/cpumask 전력 관리 추상화, DRM 그래픽 드라이버 인프라, nova GPU 드라이버 초기 통합, XArray 바인딩, auxiliary bus 지원이 핵심이다. Rust 기반 커널 드라이버 작성에 필요한 하드웨어 추상화가 대폭 확대된 릴리스다.

---

## 전력 관리 추상화 (cpufreq, OPP, clk, cpumask)

### 변경사항
- **cpufreq 추상화** — CPU 주파수 제어 인터페이스의 Rust 바인딩
- **OPP (Operating Performance Points) 추상화** — 전력/성능 동작점 관리 인터페이스
- **clk 추상화** — 클럭 프레임워크의 Rust 바인딩
- **cpumask 추상화** — CPU 마스크 조작 API의 Rust 바인딩
- **cpuid** — CPU 식별 기능의 Rust 바인딩

### 배경 및 영향
- 이 추상화들은 Rust로 전력 관리 관련 드라이버를 작성하기 위한 핵심 인프라
- cpufreq + OPP 조합은 SoC의 DVFS(Dynamic Voltage/Frequency Scaling) 드라이버 구현에 필수
- ARM SoC 환경에서 Rust 기반 cpufreq 드라이버 작성이 이론적으로 가능해진 첫 릴리스
- clk 프레임워크 바인딩은 SoC의 클럭 트리 관리 드라이버에 필요

---

## DRM (Direct Rendering Manager) 추상화

### 변경사항
- **DRM 그래픽 드라이버 인프라 바인딩** — GPU 드라이버 개발을 위한 DRM 서브시스템의 Rust 추상화
- **configfs 지원** — Rust 모듈에서 configfs 사용 가능

### 배경 및 영향
- DRM 추상화는 nova GPU 드라이버의 기반이 되는 핵심 인프라
- 6.15에서 DRM Rust 추상화 추가가 시작되었고, 6.16에서 본격 확장
- 향후 Rust 기반 GPU 드라이버(NVIDIA nova, 잠재적으로 다른 GPU)의 개발 가속화

---

## nova GPU 드라이버

### 변경사항
- **nova 드라이버 초기 통합 진행** — NVIDIA 오픈소스 커널 드라이버의 Rust 구현 진행 (LWN.net: stub 드라이버 계속 업스트림 통합)

### 배경 및 영향
- nova는 NVIDIA GPU를 위한 순수 Rust 기반 오픈소스 커널 드라이버
- 6.15에서 초기 stub 코드가 머지된 이후, 6.16에서 DRM 추상화와 함께 진전
- 장기적으로 nouveau 드라이버를 보완/대체하는 것이 목표
- Rust의 메모리 안전성을 GPU 드라이버 개발에 적용하는 사례로 주목

---

## XArray 바인딩

### 변경사항
- **최소 XArray 바인딩** — XArray 자료구조의 기본 Rust 바인딩 추가

### 배경 및 영향
- XArray는 커널 내에서 배열 기반 데이터 저장소로 광범위하게 사용 (페이지 캐시, 디바이스 관리 등)
- Rust 드라이버가 커널의 XArray를 안전하게 사용할 수 있게 됨
- 추후 더 완전한 XArray API가 Rust로 추가될 기반

---

## Auxiliary Bus

### 변경사항
- **auxiliary bus 디바이스 바인딩 프레임워크** — Rust에서 auxiliary bus를 통한 디바이스 바인딩 지원

### 배경 및 영향
- Auxiliary bus는 하나의 물리 디바이스에서 다수의 기능(function)을 별도 드라이버로 분리하는 프레임워크
- 현대 SoC와 복합 디바이스(네트워크+RDMA, GPU+비디오 등)에서 활용
- Rust 드라이버가 auxiliary bus를 통해 다른 드라이버와 상호작용 가능

---

## io polling 및 기타

### 변경사항
- **io polling 추상화** — 비동기 I/O 폴링의 Rust 바인딩
- **KUnit 통합 강화** — Rust assertion과 테스트 결과를 KUnit으로 통합하여 커널 테스트 프레임워크와의 일관성 향상

### 배경 및 영향
- io polling은 고성능 스토리지/네트워크 드라이버의 핵심 패턴
- KUnit 통합은 Rust 커널 코드의 테스트 커버리지와 CI 통합에 필수

---

## tools/nolibc

### 변경사항
- **m68k 아키텍처 지원** — nolibc에 Motorola 68000 아키텍처 추가
- **다수의 표준 라이브러리 함수 추가** — Rust 커널 개발에 필요한 기본 함수 확장

### 배경 및 영향
- nolibc는 커널 빌드 환경에서 libc 없이 기본 C/Rust 함수를 제공하는 경량 라이브러리

---

## 컴파일러 요구사항

### 변경사항
- **최소 GCC 8.1, binutils 2.30** — 모든 아키텍처에서 GCC 8.1과 binutils 2.30을 최소 버전으로 설정
- 여러 deprecated 플러그인 제거 (SANCOV, structleak)

### 배경 및 영향
- 오래된 컴파일러 버전 지원 제거로 코드베이스 정리 및 최신 컴파일러 기능 활용 가능
- Rust 커널 코드는 이미 특정 최소 Rust 컴파일러 버전을 요구하므로, C 측도 버전 기준 상향

---

## 이전 버전과의 연속성

- **Rust 추상화 확대**: 6.13 기초 인프라 → 6.14 네트워크 관련 → 6.15 pin-init/hrtimer/mm → 6.16 cpufreq/OPP/DRM/XArray로 드라이버 작성 기반 대폭 확대
- **nova GPU**: 6.15 stub 코드 머지 → 6.16 DRM 추상화와 함께 진전. 매 릴리스마다 기능 추가 예상
- **Rust 드라이버 생태계**: 6.16에서 전력 관리(cpufreq/OPP/clk) + GPU(DRM) + 버스(auxiliary bus) 추상화가 동시에 추가되어, 실질적인 하드웨어 드라이버 작성이 가능해지는 전환점

---

## 참고 자료

- [kernelnewbies.org — Linux 6.16](https://kernelnewbies.org/Linux_6.16)
- [LWN.net — The first half of the 6.16 merge window](https://lwn.net/Articles/1022512/)
- [Phoronix — Linux 6.16 Features](https://www.phoronix.com/news/Linux-6.16-Features-Early-Look)
