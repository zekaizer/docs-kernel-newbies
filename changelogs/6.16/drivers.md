# Linux 6.16 드라이버 변경사항

> 출처: kernelnewbies.org, LWN.net, Phoronix
> 릴리스: Linux 6.16
> 작성일: 2026-02-10

---

## 개요

Linux 6.16 드라이버 영역은 USB 오디오 오프로드(2년+ 개발), NVIDIA Hopper/Blackwell GPU nouveau 지원, AMDGPU 사용자 모드 큐, Intel QAT GEN6 암호화 가속, Realtek RTL8127A 10GbE, Apple M2 PCIe 지원이 핵심이다. 전반적으로 GPU와 네트워킹 드라이버에 가장 많은 변경이 이루어졌다.

---

## USB 오디오 오프로드

### 변경사항
- **USB 오디오 스트림 오프로드** — 시스템 슬립 중에도 USB 오디오 스트림을 유지하는 기능. 온보드 사운드 칩의 오프로드와 유사한 기능을 USB 오디오로 확장

### 배경 및 영향
- Greg Kroah-Hartman에 따르면 "30개 이상 패치 시리즈, 2년 이상 개발 기간"이 소요된 기능
- 배터리 구동 장치에서 USB 헤드셋/DAC 사용 시 시스템을 저전력 상태로 전환하면서도 오디오 재생 유지 가능
- Qualcomm USB 오디오 오프로드도 관련 패치 포함 — Android 스마트폰에서의 활용 가능성
- 임베디드/모바일 환경에서 오디오 전력 효율의 중요한 진전

---

## GPU 드라이버

### NVIDIA
- **Hopper/Blackwell GPU PCI ID 추가** — nouveau 드라이버에 NVIDIA Hopper(H100 등) 및 Blackwell GPU PCI ID 추가로 기본 지원
- **nova 드라이버 진행** — Rust 기반 NVIDIA 오픈소스 커널 드라이버 개발 계속

### AMD
- **AMDGPU 사용자 모드 큐** — 실험적 사용자 모드 큐 지원 제출. GPU 작업 제출을 유저스페이스에서 직접 수행하여 오버헤드 감소
- **AMD 가상 TPM 드라이버** — AMD SoC에서의 가상 TPM 지원
- **AMD 리셋 이유 리포팅** — Zen 시스템의 리셋/리부팅 원인을 커널에서 리포팅

### Intel
- **Link Off Between Frames (LOBF)** — eDP 디스플레이에서 프레임 간 링크를 끄는 전력 절약 기능
- **Xe3 Panther Lake 활성화** — 차세대 Intel GPU 지원
- **DRM IN_FORMATS_ASYNC** — 비동기 플리핑을 위한 포맷 지원
- **Xe 드라이버 팬 속도 보고** — Intel Arc GPU의 팬 속도 sysfs 노출

### 기타 GPU
- **TI AM68 GPU** — PowerVR DRM 드라이버 내 TI AM68 GPU 지원
- **Mediatek HDMI 2.0 준비** — MT8195/MT8188에서의 HDMI 2.0 DRM 준비 작업
- **Asahi Apple Silicon** — 그래픽 UAPI 헤더 추가 (커널 드라이버는 아직 미포함)

---

## 네트워크 드라이버

### 변경사항
- **Realtek RTL8127A 10GbE** — 새 10GbE 이더넷 컨트롤러 지원
- **OpenVPN DCO** — OpenVPN 커널 데이터 채널 오프로드 가상 드라이버
- 다수의 Ethernet/PHY 드라이버 추가: Renesas, Broadcom, AMD, Qualcomm, Mediatek 플랫폼

---

## 플랫폼/SoC 드라이버

### 변경사항
- **Apple M2 PCIe 컨트롤러** — Apple Silicon M2 Pro의 PCIe 컨트롤러 지원
- **Apple Magic Mouse 2 USB-C** — USB-C 연결 지원
- **Qualcomm Snapdragon X 노트북** — ASUS Zenbook A14, HP EliteBook Ultra G1q, ThinkPad T14s, XPS 13 9345 외부 DisplayPort 지원
- **Intel 플랫폼**: Wildcat Lake 오디오, PTC(Platform Temperature Control) 서멀, OverClock Watchdog 드라이버
- **intel_pstate EAS** — 하이브리드 플랫폼(SMT 미지원)에서 Energy-Aware Scheduling을 위한 에너지 모델 등록

### SoC 관련
- **NXP S32G** — 타이머 지원
- **Rockchip RK3528** — 핀 제어 지원
- **Samsung Exynos Auto v920** — 입력 디바이스 지원
- **Allwinner H6/H616** — 전력 관리 도메인
- **Dimensity 1200 MT6893** — Mediatek SoC 전력 도메인

---

## 오디오 드라이버

### 변경사항
- **AMD ACP 7.x** — 차세대 AMD 오디오 코프로세서 지원
- **Intel Wildcat Lake** — Intel WCL 오디오 지원
- **NVIDIA Tegra264** — Tegra 오디오 지원
- **Cirrus Logic CS35L63/CS48L32** — 앰프 및 코덱 드라이버
- **Everest ES8375/ES8389** — 오디오 코덱
- **Rockchip SAI** — Serial Audio Interface
- **Loongson AC'97** — 레거시 오디오 지원

---

## 센서 및 모니터링

### 변경사항
- **ASUS ROG Z690 센서** — 마더보드 센서 모니터링
- **Intel PTC 서멀** — 플랫폼 온도 제어 인터페이스
- **KEBA 컨트롤러** — 산업용 컨트롤러
- **MAX77705 IC** — PMIC 모니터링
- **입력 디바이스**: ByoWave Proteus 컨트롤러, AMD HID2, Renesas RZ/G3E

---

## 암호화 및 보안

### 변경사항
- **Intel QAT GEN6** — 6세대 QuickAssist Technology 드라이버
- **dm-crypt 래핑 인라인 암호화 키** — 하드웨어 인라인 암호화 엔진 활용
- **Turris ECDSA 서명** — keyctl() 기반 ECDSA 서명 지원

---

## 미디어 드라이버

### 변경사항
- **ST VD55G1/VD56G3** — 이미지 센서 드라이버
- **OmniVision OV02C10** — 카메라 센서 지원
- **MT8192 Spherion, MT8186 Corsola** — Mediatek SoC 미디어 지원

---

## 이전 버전과의 연속성

- **GPU 드라이버**: 6.15 NOVA 초기 stub → 6.16 DRM 추상화 확장 + NVIDIA Hopper/Blackwell PCI ID. AMDGPU 사용자 모드 큐는 오래된 AMD 로드맵 항목의 실현
- **USB 오디오**: 2년 이상 개발 끝에 6.16에서 최종 머지. 모바일/임베디드 오디오의 중요한 이정표
- **SoC 드라이버**: 매 릴리스마다 Qualcomm, Mediatek, Samsung, Rockchip 등 모바일 SoC 지원 확대
- **인라인 암호화**: Android FBE의 하드웨어 암호화 지원이 dm-crypt 레벨에서 계속 확장

---

## 참고 자료

- [kernelnewbies.org — Linux 6.16](https://kernelnewbies.org/Linux_6.16)
- [LWN.net — The first half of the 6.16 merge window](https://lwn.net/Articles/1022512/)
- [LWN.net — The second half of the 6.16 merge window](https://lwn.net/Articles/1023075/)
- [Phoronix — Linux 6.16 Features](https://www.phoronix.com/news/Linux-6.16-Features-Early-Look)
- [linuxiac — Linux Kernel 6.16 Released](https://linuxiac.com/linux-kernel-6-16-released-this-is-whats-new/)
