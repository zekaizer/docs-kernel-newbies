# Linux 6.17 드라이버 변경사항

> 출처: kernelnewbies.org, LWN.net, Phoronix, CNX Software, OMG Ubuntu
> 릴리스: Linux 6.17
> 작성일: 2026-02-10

---

## 개요

Linux 6.17 드라이버 영역은 Intel Xe3 Panther Lake GPU 기본 활성화, AMD SmartMux 하이브리드 GPU, Apple M1/M2 SMC 재부팅 지원, Raspberry Pi 5 RP1 I/O 칩, Qualcomm Iris HEVC/VP9 디코더, Intel IPU7 웹캠, Samsung Exynos 2200 Galaxy S22, NVIDIA Tegra264 Jetson T5000이 핵심이다.

---

## GPU / 디스플레이

### Intel 그래픽
- **Xe3 Panther Lake 기본 활성화** — Intel 차세대 노트북 GPU(Xe3 통합 그래픽)가 기본으로 활성화 (Phoronix, OMG Ubuntu). 하드웨어 공식 출시 전 소프트웨어 선행 준비
- **Wildcat Lake 초기 GPU 드라이버** — 차세대 Intel 플랫폼 GPU 지원 추가
- **Intel Arc Pro B-Series SR-IOV** — 전문 그래픽 카드의 SR-IOV 가상화 기능

### AMD 그래픽
- **SmartMux** — 하이브리드 GPU 노트북에서 통합/디스크리트 그래픽 간 자동 전환 지원 (LWN.net, Phoronix). 워크로드에 따른 동적 GPU 선택

### NVIDIA
- **Tegra264 (Thor)** — Jetson T5000 모듈 지원. 자율주행/로보틱스 플랫폼

### 배경 및 영향
- Intel Xe3 기본 활성화는 2025-2026년 Panther Lake 노트북 출시에 대비한 소프트웨어 성숙도 확보
- AMD SmartMux는 Linux 노트북의 GPU 전환 경험을 Windows 수준으로 향상
- Tegra264는 NVIDIA의 자동차/로보틱스 전략과 연계. Android Automotive 플랫폼에 관련 가능

---

## 웹캠 / 카메라

### 변경사항
- **Intel IPU7** — Lunar Lake(Core Ultra Series 2) 및 Panther Lake 웹캠 지원 (OMG Ubuntu)
- **Qualcomm Iris 디코더** — HEVC(H.265) 및 VP9 코덱 초기 지원 (LWN.net)
- **PinePhone Pro 카메라** — Rockchip 기반 PinePhone Pro 카메라 지원 (CNX Software)

### 배경 및 영향
- Qualcomm Iris 디코더는 Snapdragon SoC의 하드웨어 비디오 디코더 메인라인 통합. Android 미디어 프레임워크와 직접 관련
- Intel IPU7은 최신 Intel 노트북의 카메라 기능 정상 동작에 필수

---

## 노트북 / 데스크톱 하드웨어

### Apple Silicon
- **M1/M2 SMC 드라이버** — Apple System Management Controller 업스트림으로 메인라인 커널 재부팅 가능 (LWN.net, OMG Ubuntu)
- **Touch Bar 입력 개선** — Intel MacBook Pro Touch Bar 입력 처리 향상

### Lenovo
- **WMI Gaming Series** — Legion Go, Legion Go S 핸드헬드 드라이버 (OMG Ubuntu)

### 기타
- **ASUS Zenbook A14** — Snapdragon X1 Plus/Elite 메인라인 지원 (CNX Software)
- **Dell/Alienware 성능 부스트 키** — 키보드 성능 부스트 키 표준화
- **Flydigi Apex 5** — 게임 컨트롤러 지원
- **F13-F24 키보드 매핑** — 확장 기능키 지원
- **Intel Touch Host Controller** — Wake-on-Touch, 오버레이 객체 지원

### 배경 및 영향
- Apple M1/M2 재부팅 지원은 Linux on Apple Silicon 사용자 경험의 중요한 마일스톤
- Snapdragon X1 메인라인 지원은 ARM Windows/Linux 노트북 생태계 확대

---

## 임베디드 / SBC

### 변경사항
- **Raspberry Pi 5 RP1 I/O 칩** — 멀티펑션 I/O 칩 드라이버 + pinctrl/clk 프레임워크 통합 (LWN.net, CNX Software)
- **Corsair HX1200i PSU 모니터링** — hwmon 기반 전원 공급 장치 모니터링
- **Raspberry Pi 7" LCD** — 720×1280 디스플레이 지원 (ilitek-ili9881c)
- **RGB161616/BGR161616 DRM FourCC** — 새 DRM 픽셀 포맷

### 배경 및 영향
- RPi5 RP1 드라이버는 Raspberry Pi 5의 완전한 메인라인 커널 지원을 위한 핵심 구성 요소. GPIO, SPI, I2C, UART 등 모든 주변 장치 접근에 필요

---

## 오디오

### 변경사항
- **AMD ACP7.2 플랫폼 오디오** — 새 AMD 오디오 코프로세서 플랫폼 지원
- **Framework Laptop 13 HD Audio** — AMD Ryzen AI 300 모델 지원
- **ASUS/HP 상업 모델 HD Audio** — 비즈니스 노트북 오디오 지원
- **USB 오디오 오프로드** — Fairphone 4 등 모바일 기기 USB 오디오 오프로드

### 배경 및 영향
- USB 오디오 오프로드 확장은 6.16에서 도입된 기능의 기기 지원 확대. Android 스마트폰의 USB-C 오디오 배터리 효율 개선

---

## 네트워킹 드라이버

### 변경사항
- **MT7925 Bluetooth** — MediaTek Wi-Fi/BT 칩 ID 추가
- **Airoha EN7581 이더넷** — 임베디드 네트워크 프로세서 지원

### 배경 및 영향
- MediaTek MT7925는 Wi-Fi 7 + Bluetooth 5.4 칩으로, 최신 노트북/태블릿에 탑재

---

## 전력 관리

### 변경사항
- **AMD Hardware Feedback Interface (HFI)** — 하이브리드 CPU에서 태스크 배치 최적화 힌트 제공
- **Turbostat L3 캐시 토폴로지** — L3 캐시 정보 표시 추가

### 배경 및 영향
- AMD HFI는 Intel Thread Director의 AMD 대응. 스케줄러가 하드웨어 정보를 기반으로 효율적 태스크 배치

---

## 스토리지 드라이버

### 변경사항
- UFS, NVMe 관련 변경사항은 [블록 레이어 상세](block-layer.md) 참조

---

## 이전 버전과의 연속성

- **Intel GPU**: 6.16 LOBF eDP 절전 → 6.17 Xe3 Panther Lake 기본 활성화
- **AMD GPU**: 6.16 사용자 모드 큐 실험적 → 6.17 SmartMux 하이브리드 GPU
- **모바일 SoC 지원**: 매 릴리스 Samsung, Qualcomm, MediaTek, Rockchip 드라이버 확대
- **Apple Silicon**: 6.16 M2 PCIe → 6.17 M1/M2 SMC 재부팅으로 기본 플랫폼 기능 완성
- **Qualcomm 미디어**: 6.17에서 Iris 디코더 HEVC/VP9 초기 지원. 메인라인 비디오 디코딩 시작
- **USB 오디오**: 6.16 오프로드 도입 → 6.17 기기 지원 확대

---

## 참고 자료
- [kernelnewbies.org Linux 6.17](https://kernelnewbies.org/Linux_6.17)
- [Phoronix Linux 6.17 Features](https://www.phoronix.com/news/Linux-6.17-Features-Reminder)
- [OMG Ubuntu Linux 6.17](https://www.omgubuntu.co.uk/2025/09/linux-kernel-6-17-new-features)
- [CNX Software Linux 6.17 ARM/RISC-V](https://www.cnx-software.com/2025/09/29/linux-6-17-release-main-changes-arm-risc-v-and-mips-architectures/)
