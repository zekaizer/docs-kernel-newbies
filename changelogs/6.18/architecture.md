# Linux 6.18 아키텍처 변경사항

> 출처: kernelnewbies.org, Phoronix, CNX Software
> 릴리스: Linux 6.18
> 작성일: 2026-02-10

---

## 개요

Linux 6.18 아키텍처 영역은 ARM64에서 Apple M2 Pro/Max/Ultra Device Tree 업스트림, Qualcomm Glymur/SM8750 지원, MediaTek MT8196/MT6991 확장이 핵심이다. RISC-V는 RPMI 플랫폼 관리 인터페이스와 ESWIN EIC7700 SoC 지원을 추가했다. x86은 microcode 커맨드라인 옵션과 AMD Zen 6 / Intel Wildcat Lake 준비를 진행했다.

---

## ARM / ARM64

### Apple Silicon
- **M2 Pro/Max/Ultra Device Tree** — Apple M2 Pro, M2 Max, M2 Ultra 칩의 Device Tree 업스트림. Asahi Linux의 메인라인 통합 노력의 결과
- **M2 워크스테이션 지원** — M2 기반 워크스테이션 하드웨어 지원 확대

### Qualcomm
- **Glymur 플랫폼** — 새 Qualcomm 플랫폼의 global/display/rpmh 클럭 컨트롤러 도입
- **SM8750** — QMP PCIe PHY, 광범위한 Device Tree 업데이트
- **IPQ5424 APSS 클럭** — 네트워크 SoC 클럭 지원
- **MSM8916→MSM8937 확장** — 레거시 SoC 지원 확대
- **PCIe 개선** — 8.0/32.0 GT/s 이퀄라이제이션, ECAM 메커니즘, 슬롯 전원 제어
- **Snapdragon X 노트북** — Dell Inspiron 14 Plus, Dell Latitude 7455, HP OmniBook X14, Lenovo ThinkBook T16 Device Tree 추가
- **새 디바이스** — Hamoa IoT SOM/EVK, Samsung Galaxy S22/S20, Fairphone 5

### MediaTek
- **MT8196 Chromebook** — MT8196 기반 Chromebook 지원
- **MT6991 (Dimensity 9400)** — I2C 지원 추가
- **MT8196 클럭 컨트롤러** — 클럭 드라이버 추가
- **PCIe Gen3 컨트롤러** — MT8196/MT6991 PCIe 지원

### Samsung / Exynos
- **Axis ARTPEC-8** — Samsung 설계 기반 SoC 클럭/핀 컨트롤러 드라이버
- **Exynos990** — PERIC 클럭 컨트롤러 확장
- **Google GS101 CPU Idle** — ACPM 펌웨어 기반 CPU Idle 지원
- **Exynos850/990/2200** — Device Tree 확장
- **Samsung Exynos 7870** — 10년 된 14nm SoC 지원 추가

### Allwinner
- **A523 SoC** — GMAC200, MCU PRCM 클럭, 리셋 컨트롤러, Vivante GC9000 NPU 지원
- **Orange Pi Zero** — 애드온 보드 오버레이 지원

### Rockchip
- **RK3588** — MIPI CSI-2 DPHY, DPTX 출력, NPU 드라이버
- **RK3528** — combphy PHY 지원
- **새 보드** — NanoPi Zero2, ArmSoM Sige1, Radxa ROCK 2A/2F, Firefly ROC-RK3588-RT

### Raspberry Pi
- **RP1 이더넷 컨트롤러** — RPi5 이더넷 지원
- **RPi5** — 핀 컨트롤러, GPIO 추가, 두 번째 SDHCI (Wi-Fi)
- **KVM VGIC** — RPi5 인터럽트 라인 지원
- **drm/v3d** — PREEMPT_RT 안정성을 위한 fence lock 분리

### NXP
- **i.MX91** — 단일 코어 Cortex-A55 SoC 지원
- **TQ Embedded 모듈** — 산업용 모듈 지원

### Renesas
- **RZ/T2H, RZ/N2H** — 산업용 SoC 지원 추가

### 기타 ARM
- **Broadcom bcm4708** — 네트워크 라우터 보드
- **Amlogic A113L2** — SFC 플래시 컨트롤러 포함 SoC 지원

### 배경 및 영향
- Apple M2 Pro/Max/Ultra DT 업스트림은 Asahi Linux 프로젝트의 중요한 마일스톤. 고성능 Apple Silicon Mac에서의 메인라인 Linux 지원 확대
- Qualcomm Snapdragon X 노트북 지원은 ARM 기반 Windows/Linux 노트북 생태계 확대
- Samsung Galaxy S22/S20 Device Tree 추가는 메인라인 커널로의 Android 디바이스 지원 확대 트렌드

---

## x86

### 변경사항
- **microcode= 커맨드라인 옵션** — x86 마이크로코드 로더 동작을 커맨드라인에서 제어
- **AMD Zen 6 준비** — 차세대 AMD 프로세서 플랫폼 초기 준비
- **AMD EPYC Venice** — EDAC 드라이버에서 16채널 메모리 Venice 프로세서 인식 (EPYC 8004 후속)
- **Intel Wildcat Lake** — Xe3 그래픽 통합 디스플레이 지원
- **Intel Panther Lake** — 전력 슬라이더(thermal management) 지원
- **AMD ABMC** — Assignable Bandwidth Monitoring Counters (EPYC 서버용)
- **Zhaoxin CPU** — KVM에서 Centaur CPU leaves 허용

### 배경 및 영향
- microcode 커맨드라인 옵션은 보안 패치 적용 및 성능 트레이드오프 관리에 유용
- AMD Zen 6 준비는 2026년 출시 예상 프로세서의 초기 지원
- Intel Wildcat Lake/Panther Lake은 차세대 인텔 플랫폼 준비

---

## RISC-V

### 변경사항
- **RPMI (RISC-V Platform Management Interface)** — ARM SCMI에 대응하는 플랫폼 관리 인터페이스. irqchip, mailbox, clk 드라이버 포함
- **MPXY SBI 확장** — S-mode OS와 M-mode 펌웨어 간 공유 메모리 mailbox
- **ESWIN EIC7700 SoC** — SiFive HiFive Premier P550 보드에 사용되는 SoC 초기 지원
- **ioremap_wc() 도입** — Write-Combining 메모리 매핑 지원
- **ACPI 부트 로고** — RISC-V에서 ACPI 부트 로고 지원
- **__ASSEMBLY__ → __ASSEMBLER__ 정리** — 어셈블리 헤더 매크로 표준화
- **엔디안 스왑 매크로** — RISC-V 명령어를 활용한 바이트 순서 변환
- **새 SoC/보드**: Alibaba T-Head TH1520 (IMG BXM-4-64 GPU), Microchip PolarFire, Sophgo CV1800/SG2042, SpacemiT P1/K1, StarFive JH7110

### 배경 및 영향
- RPMI는 RISC-V 생태계의 표준 플랫폼 관리 인터페이스로, ARM의 SCMI와 유사한 역할
- ESWIN EIC7700은 고성능 RISC-V SoC로, Linux 메인라인 지원은 RISC-V 서버/데스크톱 생태계 확대에 기여
- 6.16의 KVM 정식 전환에 이어 플랫폼 인프라(RPMI, MPXY) 확충으로 RISC-V 성숙도 향상

---

## LoongArch

### 변경사항
- **KVM PTW 감지** — 새 하드웨어에서 페이지 테이블 워크 기능 감지
- **IPI 에뮬레이션 개선** — 인커널 프로세서 간 인터럽트 에뮬레이션 성능 향상
- **PCH-PIC 에뮬레이션 개선** — 플랫폼 인터럽트 컨트롤러 에뮬레이션 개선

---

## MIPS

### 변경사항
- **Loongson32 Device Tree 이전** — MIPS_GENERIC 프레임워크로 이전
- **내장 DTB** — Loongson 내장 DTB 지원
- **strcpy() 비사용화** — 다수 플랫폼에서 strcpy() → 안전한 대체 함수 전환

---

## 이전 버전과의 연속성

- **ARM64**: 6.16 ARM64 lazy preemption + Scalable Matrix Extension → 6.18 Apple M2 Pro/Max/Ultra DT + Qualcomm SM8750 + MediaTek MT8196. 플랫폼 지원 폭 확대
- **RISC-V**: 6.16 KVM 정식 전환 + getrandom vDSO → 6.18 RPMI + ESWIN EIC7700 + MPXY SBI. 생태계 인프라 구축
- **x86**: 6.16 APX 32 레지스터 + CONFIG_X86_NATIVE_CPU → 6.18 microcode 커맨드라인 + Zen 6 준비. 차세대 플랫폼 대비
- **Apple Silicon**: 6.18에서 M2 Pro/Max/Ultra DT 업스트림은 Asahi Linux의 가장 큰 메인라인 통합 진전 중 하나

---

## 참고 자료

- [CNX Software — Linux 6.18 release: ARM, RISC-V, MIPS](https://www.cnx-software.com/2025/12/01/linux-6-18-release-main-changes-arm-risc-v-and-mips-architectures/)
- [kernelnewbies.org — Linux 6.18](https://kernelnewbies.org/Linux_6.18)
- [Phoronix — Linux 6.18 Features](https://www.phoronix.com/news/LInux-6.18-Features-Reminder)
- [Asahi Linux — Progress Report: Linux 6.18](https://asahilinux.org/2025/12/progress-report-6-18/)
