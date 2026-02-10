# Linux 6.17 아키텍처 변경사항

> 출처: kernelnewbies.org, LWN.net, LKML, CNX Software, Phoronix
> 릴리스: Linux 6.17
> 작성일: 2026-02-10

---

## 개요

Linux 6.17 아키텍처 영역은 ARM64의 커널 라이브 패칭 지원과 GICv5 차세대 인터럽트 컨트롤러, x86의 공격 벡터 제어(AVC)와 AMD Hardware Feedback Interface(HFI), RISC-V 변경사항 거부(6.18 이월)가 핵심이다. S390의 THP 스왑/마이그레이션, LoongArch의 BPF 지원 강화도 주목할 만하다.

---

## ARM64

### 변경사항
- **커널 라이브 패칭** — 64비트 ARM 시스템에서 커널 라이브 패칭 지원 추가 (LWN.net, CNX Software). 재부팅 없이 보안 패치 적용 가능
- **GICv5 인터럽트 컨트롤러** — ARM 차세대(5세대) Generic Interrupt Controller 지원 (CNX Software). 인터럽트 라우팅, MSI, 인터럽트 변환, 유선 인터럽트 포함
- **Branch Record Buffer Extension (BRBE)** — ARM v9.2 분기 레코드 버퍼를 perf 이벤트에 통합 (LWN.net)
- **kcfi + BPF** — ARM64에서 커널 제어 흐름 무결성(kCFI)과 BPF 조합 지원
- **feat_mte_store_only** — Memory Tagging Extension 저장 전용 기능

### 배경 및 영향
- 커널 라이브 패칭은 서버/클라우드 환경에서 필수 기능으로, ARM64 서버(AWS Graviton, Ampere Altra 등)의 프로덕션 환경 적용 가능성 대폭 향상
- GICv5는 ARM 서버/모바일의 차세대 인터럽트 처리 아키텍처. 현재 하드웨어는 미출시이나 소프트웨어 선행 준비
- BRBE는 perf 기반 분기 프로파일링으로, AutoFDO(Automatic Feedback-Directed Optimization) 등 PGO 활용에 핵심
- kcfi + BPF는 ARM64에서의 BPF 보안 강화. 제어 흐름 무결성과 BPF 프로그램의 공존 보장

---

## x86

### 변경사항
- **공격 벡터 제어(AVC)** — CPU 취약점 완화를 공격 벡터 단위로 관리하는 커맨드라인 프레임워크 (kernelnewbies.org, LWN.net)
- **AMD Hardware Feedback Interface (HFI)** — AMD 하이브리드 프로세서의 하드웨어 피드백 인터페이스로 태스크 배치 최적화 (LWN.net, Phoronix)
- **Intel Wildcat Lake PCI** — 차세대 Intel 플랫폼 지원
- **Intel Bartlett Lake-S** — 새 데스크톱 플랫폼 지원

### 배경 및 영향
- AVC는 Spectre/Meltdown/MDS 등 다수의 CPU 취약점 완화 파라미터를 통합 관리. 새 취약점 발견 시 관리자 개입 최소화
- AMD HFI는 Intel Thread Director에 대응하는 AMD의 하드웨어 기반 태스크 스케줄링 힌트. big.LITTLE 유사 하이브리드 아키텍처에서 효율적 태스크 배치
- AMD HFI와 sched_ext의 결합으로 하이브리드 CPU의 에너지 효율적 스케줄링 가능

---

## RISC-V

### 변경사항
- **변경사항 거부** — Linus Torvalds가 RISC-V 아키텍처 변경사항을 "garbage" pull request로 지적하고 거부 (Phoronix). 머지 윈도우 마감 직전 금요일에 제출된 점도 문제
- 6.18로 이월

### 배경 및 영향
- RISC-V 커뮤니티의 코드 품질 및 제출 프로세스에 대한 경고. 6.17에서 RISC-V 아키텍처 변경은 없음
- 다만 RISC-V KVM 개선(더티 메모리 추적, 중첩 가상화 MMU)은 별도 서브시스템을 통해 포함
- 디바이스 트리 업데이트 일부는 포함: Sophgo SG2042/SG2044, SpacemiT K1, StarFive JH7110, Andes QiLai 등

---

## LoongArch

### 변경사항
- **BPF 지원 강화** — 동적 코드 수정, BPF trampolines, struct_ops 프로그램 지원 (LWN.net, CNX Software)

### 배경 및 영향
- LoongArch(중국 Loongson 아키텍처)의 BPF 생태계가 x86/ARM64 수준으로 발전
- BPF trampoline은 고성능 BPF 프로그램 실행의 핵심 인프라

---

## S390

### 변경사항
- **투명 대형 페이지(THP) 스왑 및 마이그레이션** — S390에서 THP의 스왑 아웃 및 마이그레이션 지원

### 배경 및 영향
- IBM Z 시리즈 메인프레임의 메모리 관리 효율 향상. 대형 워크로드의 스왑 성능 개선

---

## SoC 플랫폼 지원 (ARM)

### 변경사항

#### Allwinner
- A523 GPU/efuse, Display Engine 3.3 (DE33) 지원 (H616, H618, H700, T507)
- Orange Pi 4A (T527 SoC) 신규 보드

#### Rockchip
- RK3528 GPU 지원 (Lima, Mali-450 MP2)
- PinePhone Pro 카메라 지원
- PCIe 링크 속도 최적화, 레이스 컨디션 방지
- 신규 보드: FriendlyElec NanoPi M5, Firefly ROC-RK3588S-PC, Luckfox Omni3576

#### Samsung/Exynos
- **Exynos 2200 (Galaxy S22)** — 핀/클럭 컨트롤러, USB 초기화
- Google GS101 (Pixel 6): 타이머 전환, MAX77759 PMIC, 향상된 리부트 핸들러
- Exynos990 usbdrd PHY, Exynos7870 MIPI PHY

#### Qualcomm
- Milos/SM8750 PHY, M31 eUSB2 PHY
- Milos: 글로벌/디스플레이/GPU/비디오/카메라/tcsr/rpmh 클럭 컨트롤러
- SA8775P CPU OPP 테이블, L3 인터커넥트, DSI, 비디오 인코더/디코더
- **ASUS Zenbook A14 (Snapdragon X1)** 신규 플랫폼

#### MediaTek
- MT6572 레거시 칩 업스트림, MT8189 pinctrl
- MT7988 CPU DVFS, Airoha EN7581 이더넷
- Steelix Squirtle Chromebook (MT8186) 신규

#### NVIDIA
- Tegra264 (Thor) — Jetson T5000 모듈

#### 기타
- Axiado AX3000 (Cortex-A53, 10Gbps 이더넷, 4TOPS NPU)
- CIX P1 (12코어 A720/A520, ARMv9.2)
- Sophgo SG2000 ARM 지원 추가

### 배경 및 영향
- Exynos 2200/Google GS101 지원은 Samsung/Google 모바일 SoC의 메인라인 커널 통합 진행
- Qualcomm Milos/SM8750은 차세대 Snapdragon 플랫폼의 준비
- MediaTek MT6572 업스트림은 레거시 SoC의 커널 지원 유지

---

## 이전 버전과의 연속성

- **ARM64 엔터프라이즈 기능**: 6.16 lazy preemption/SME → 6.17 라이브 패칭/GICv5로 서버급 기능 대폭 확대
- **x86 보안**: 6.16 Intel APX 레지스터 → 6.17 AVC/AMD HFI로 보안 및 스케줄링 최적화
- **RISC-V**: 6.16 KVM 정식 전환 → 6.17 아키텍처 변경 거부 (6.18 이월)
- **SoC 플랫폼**: 매 릴리스 Samsung, Qualcomm, MediaTek, Rockchip 등 모바일 SoC 지원 확대

---

## 참고 자료
- [kernelnewbies.org Linux 6.17](https://kernelnewbies.org/Linux_6.17)
- [LWN.net 6.17 release](https://lwn.net/Articles/1038266/)
- [CNX Software Linux 6.17 ARM/RISC-V](https://www.cnx-software.com/2025/09/29/linux-6-17-release-main-changes-arm-risc-v-and-mips-architectures/)
- [Phoronix RISC-V rejection](https://www.phoronix.com/news/Linux-6.17-RISC-V-Rejected)
