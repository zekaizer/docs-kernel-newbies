# Linux 6.18 드라이버 변경사항

> 출처: kernelnewbies.org, Phoronix, CNX Software
> 릴리스: Linux 6.18
> 작성일: 2026-02-10

---

## 개요

Linux 6.18 드라이버 영역은 Tyr Rust Mali GPU 드라이버 도입, Nouveau의 GSP 펌웨어 기본화, Rockchip NPU 가속기 업스트림, Google 햅틱 터치패드 초기 지원이 핵심이다. AMDGPU, Intel Xe, PowerVR 등 GPU 드라이버와 다수의 SoC 플랫폼 드라이버에도 의미있는 개선이 포함되었다.

---

## GPU / 그래픽

### Tyr — Rust Arm Mali GPU DRM 드라이버
- **CSF 기반 Mali GPU의 Rust DRM 드라이버** — Collabora, Arm, Google 공동 개발
- **Panthor 드라이버의 Rust 포팅** — 동일한 유저스페이스 API(DRM_IOCTL_PANTHOR_DEV_QUERY) 유지
- **RK3588 초기 지원** — GPU 전원 관리 및 디바이스 정보 쿼리 가능
- **실험적 상태** — 업스트림은 프로빙/쿼리만 구현. GPUVM 추상화 등 추가 작업 필요
- 다운스트림 프로토타입은 GNOME, Weston, SuperTuxKart에서 Panthor 대비 동등 성능 확인

### Nouveau (NVIDIA 오픈소스)
- **GSP 펌웨어 기본 사용** — Turing(GTX 16xx/RTX 20xx) 및 Ampere(RTX 30xx) GPU에서 NVIDIA GSP(GPU System Processor) 펌웨어를 기본으로 사용
- 기존 수동 활성화에서 자동 활성화로 전환하여 드라이버 성능 및 기능 향상

### AMDGPU
- **GCN 1.0/1.1 개선** — 초기 GCN 아키텍처 지원 개선
- **체크포인트/복원 기능** — GPU 상태 체크포인트 및 복원 지원

### Intel Xe
- **SR-IOV 준비 작업** — 가상화를 위한 Single Root I/O Virtualization 인프라 준비
- **Wildcat Lake 디스플레이** — Xe3 통합 그래픽 디스플레이 지원

### PowerVR
- **RISC-V 아키텍처 지원** — Imagination PowerVR GPU 드라이버의 RISC-V 플랫폼 지원 추가

### Rockchip NPU
- **RK3588 NPU 가속기 드라이버** — Rockchip NPU(Neural Processing Unit) 드라이버 업스트림

### Pixpaper
- **새 DRM 드라이버** — 새로운 Direct Rendering Manager 드라이버 추가

### VESA DRM
- **8비트 컬러 팔레트 모드** — VESA 호환 드라이버에서 8비트 팔레트 모드 지원

### v3d (Raspberry Pi)
- **PREEMPT_RT fence lock 분리** — 실시간 커널 안정성을 위한 fence lock 구조 변경

---

## 입력 디바이스

### Google 햅틱 터치패드
- **초기 햅틱 터치패드 지원** — Elan 2703 기반 햅틱 터치패드 드라이버 업스트림
- Google이 Chromebook 등에서 사용하는 햅틱 피드백 터치패드의 메인라인 지원

### Sony DualSense
- **오디오 잭 핸들링** — DualSense 컨트롤러의 3.5mm 오디오 잭 지원

### Xiaomi Redmibook
- **키보드 지원 개선** — Xiaomi Redmibook 노트북 키보드 드라이버 개선

### GPD 핸드헬드
- **센서 드라이버** — GPD 핸드헬드 게이밍 디바이스 센서 지원

---

## USB

### Intel USBIO
- **USB IO Expander 드라이버** — Intel USB IO Expander 칩 지원 메인라인화

### eUSB2V2
- **임베디드 USB2 v2.0 웹 카메라** — eUSB2V2 웹 카메라 지원

### USB Rust 바인딩
- **USB 드라이버 프레임워크** — Rust 기반 USB 드라이버 작성을 위한 초기 바인딩 및 샘플 드라이버

---

## 오디오

### TASCAM US-144MKII
- **USB 오디오 인터페이스** — TASCAM US-144MKII 오디오 인터페이스 드라이버

---

## 네트워크 드라이버

### Raspberry Pi RP1
- **이더넷 컨트롤러** — RPi5의 RP1 이더넷 컨트롤러 메인라인 지원

### Qualcomm PPE
- **Packet Processing Engine** — Qualcomm PPE 네트워크 드라이버

### Microchip LAN969x
- **SoC 네트워킹** — Microchip LAN969x SoC 네트워크 지원

---

## 전력 관리 / 열 관리

### Intel Panther Lake
- **전력 슬라이더** — 열 관리를 위한 전력 슬라이더 지원

### Intel SLPC
- **그래픽 절전 옵션** — Intel 그래픽 드라이버의 새 절전 모드

---

## 센서 / 모니터링

### ASUS WRX90E-SAGE SE
- **센서 모니터링** — 하드웨어 모니터링 센서 지원

### AMD SBTSI
- **온도 모니터링** — 동결 감지 포함 온도 모니터링

### AMD ABMC
- **대역폭 모니터링 카운터** — EPYC 서버용 Assignable Bandwidth Monitoring Counters

---

## 스토리지 드라이버

### AMD-Xilinx Versal
- **TRNG 드라이버** — True Random Number Generator 드라이버
- **DDR EDAC 드라이버** — DDR 메모리 에러 감지/수정 드라이버

### ASpeed AST2700
- **BMC 준비 작업** — 기판 관리 컨트롤러 초기 지원

---

## 기타 드라이버 서브시스템

- **SPI**: 새 디바이스 지원 및 드라이버 개선
- **Watchdog**: 다수 새 워치독 드라이버
- **Serial**: 시리얼 포트 드라이버 업데이트
- **CPU Frequency**: 주파수 스케일링 드라이버 개선
- **Regulators**: 전압 레귤레이터 드라이버 추가
- **RTC**: 실시간 클럭 드라이버 업데이트
- **Pin Controllers**: 핀 컨트롤러 드라이버 확장
- **MMC**: MMC/SD 카드 드라이버 개선
- **MTD**: 플래시 메모리 드라이버 업데이트
- **IIO**: 산업용 I/O 드라이버 추가
- **MFD**: 다기능 디바이스 드라이버
- **I2C/I3C**: I2C/I3C 버스 드라이버 개선
- **hwmon**: 하드웨어 모니터링 드라이버 확장
- **GPIO**: GPIO 드라이버 추가
- **LEDs**: LED 드라이버 업데이트
- **DMA**: DMA 엔진 드라이버 개선
- **Crypto**: 암호화 가속기 드라이버
- **PCI**: PCIe 드라이버 개선 (Qualcomm 8.0/32.0 GT/s 이퀄라이제이션 포함)
- **NTB**: Non-Transparent Bridge 드라이버
- **Clock**: 클럭 드라이버 (Allwinner, Qualcomm, MediaTek, Samsung 등)
- **PHY**: PHY 드라이버 (Rockchip combphy, Qualcomm QMP PCIe, Sophgo USB 등)
- **EDAC**: 에러 감지/수정 드라이버 (AMD EPYC Venice, ARM Cortex-A72, Xilinx Versal)

---

## 이전 버전과의 연속성

- **Rust GPU 드라이버**: 6.16 DRM 추상화 + nova 진행 → 6.18 Tyr Mali Rust DRM 업스트림. 첫 Rust GPU 드라이버 메인라인 진입
- **NVIDIA Nouveau**: GSP 펌웨어 기본화로 Nouveau의 현대 NVIDIA GPU 지원 품질 향상
- **Rockchip**: 6.18에서 NPU 드라이버 + RK3588 CSI-2 DPHY + DPTX 출력으로 RK3588 생태계 완성도 향상
- **입력 디바이스**: 햅틱 터치패드 초기 지원은 차세대 노트북 입력 장치의 메인라인 지원 시작
- **ARM Cortex-A72 EDAC**: 임베디드/서버 ARM 플랫폼의 메모리 안정성 모니터링 강화

---

## 참고 자료

- [kernelnewbies.org — Linux 6.18](https://kernelnewbies.org/Linux_6.18)
- [Phoronix — Linux 6.18 Features](https://www.phoronix.com/news/LInux-6.18-Features-Reminder)
- [CNX Software — Linux 6.18 release](https://www.cnx-software.com/2025/12/01/linux-6-18-release-main-changes-arm-risc-v-and-mips-architectures/)
- [Collabora — Kernel 6.18: Tyr advances Rust in Linux](https://www.collabora.com/news-and-blog/news-and-events/kernel-618-tyr-advances-rust-in-linux.html)
