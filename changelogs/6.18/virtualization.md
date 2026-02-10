# Linux 6.18 가상화 변경사항

> 출처: kernelnewbies.org, Phoronix, CNX Software
> 릴리스: Linux 6.18
> 작성일: 2026-02-10

---

## 개요

Linux 6.18 가상화는 KVM CET(Control-flow Enforcement Technology) 가상화, AMD AVIC 기본 활성화, SEV-SNP CipherText Hiding/Secure TSC, guest_memfd 사용자 공간 매핑이 핵심이다. ARM64, RISC-V, LoongArch KVM도 각각 의미있는 개선을 받았다.

---

## KVM CET 가상화

### 변경사항
- **Intel CET 가상화** — Shadow Stack + Indirect Branch Tracking을 KVM 게스트에서 활성화 가능
- **AMD CET 가상화** — Shadow Stack 가상화 지원 (Indirect Branch Tracking은 미포함)
- **호스트/게스트 CET 상태 전환** — VM 진입/탈출 시 CET 레지스터 상태 저장/복원

### 배경 및 영향
- CET는 ROP(Return-Oriented Programming)/JOP(Jump-Oriented Programming) 공격을 하드웨어 수준에서 방어하는 기술
- Shadow Stack: 반환 주소의 별도 보호 스택 유지. 반환 주소 변조 감지
- Indirect Branch Tracking: 간접 분기 대상의 유효성 검증
- 기존에는 호스트에서만 CET 사용 가능했으나, 6.18부터 VM 내부에서도 CET 보호 활성화
- 클라우드 환경에서의 VM 보안 수준 향상에 기여

---

## AMD AVIC 기본 활성화

### 변경사항
- **Zen 4+ x2AVIC 기본 활성화** — x2AVIC를 지원하는 Zen 4 이후 프로세서에서 AVIC가 기본으로 활성화
- **Secure AVIC** — SEV-SNP VM에서의 보안 가상 인터럽트 컨트롤러

### 배경 및 영향
- AVIC(Advanced Virtual Interrupt Controller)은 가상 인터럽트 전달의 하드웨어 가속
- x2AVIC는 AVIC의 확장 버전으로, 더 많은 vCPU 지원 및 낮은 인터럽트 지연
- 기본 활성화로 Zen 4+ 서버의 VM 인터럽트 성능이 추가 설정 없이 향상
- Secure AVIC는 SEV-SNP의 기밀 컴퓨팅 환경에서도 안전한 인터럽트 전달 보장

---

## SEV-SNP 개선

### 변경사항
- **CipherText Hiding** — SNP 게스트의 비공개 메모리 암호문에 대한 비인가 CPU 접근 차단. EPYC 시스템에서 오프라인 공격 방어 (opt-in)
- **Secure TSC** — 비신뢰 호스트가 게스트의 Timestamp Calibration 주파수를 변조하는 것을 방지

### 배경 및 영향
- CipherText Hiding은 물리적 서버 접근이 가능한 공격자로부터 VM 메모리 암호문을 보호하는 추가 방어 계층
- Secure TSC는 타이밍 기반 사이드채널 공격 방지에 기여
- 6.16의 SEV-SNP SVSM vTPM + TSM-MR 통합 ABI에 이어 기밀 컴퓨팅 보안 강화 지속

---

## guest_memfd 매핑

### 변경사항
- **호스트 사용자 공간 매핑** — KVM_MEMORY_ATTRIBUTE_PRIVATE를 지원하지 않는 VM 타입에서 guest_memfd 메모리의 호스트 mmap() 지원
- **커널 직접 맵에서 게스트 메모리 제거** 방향으로 진전

### 배경 및 영향
- guest_memfd는 게스트 전용 메모리를 관리하기 위한 새 메커니즘
- 커널 직접 맵에서 게스트 메모리를 제거하면 호스트 커널의 취약점을 통한 게스트 메모리 접근 방지
- 기밀 컴퓨팅(TDX, SEV-SNP)의 메모리 격리 요구사항 충족을 위한 점진적 개선

---

## ARM64 KVM

### 변경사항
- **FF-A 1.2 보안 메모리 도관** — pKVM(Protected KVM)에서 FF-A 1.2를 보안 메모리 컨디트로 지원
- **마이그레이션 개선** — VM 마이그레이션 관련 인프라 개선
- **EL2 기능 비활성화 인프라** — 특정 EL2 기능을 선택적으로 비활성화할 수 있는 기반 구축

### 배경 및 영향
- FF-A(Firmware Framework for Arm)는 Secure World/Normal World 간 통신 표준. 1.2 지원은 보안 파티션과의 메모리 공유 개선
- pKVM은 Android의 보호된 가상화 기술로, FF-A 1.2 통합은 Android pKVM의 보안 아키텍처 강화
- 6.16의 ARM64 KVM THP/중첩 가상화에 이어 보안 및 마이그레이션 중심 개선

---

## RISC-V KVM

### 변경사항
- **SBI FWFT 확장** — Firmware Feature (FWFT) SBI 확장 지원
- **Zicbop 확장** — Cache Block Operation (Prefetch) 확장 게스트 지원
- **bfloat16 확장** — bfloat16 부동소수점 확장 게스트 지원

### 배경 및 영향
- RISC-V KVM은 6.16에서 실험적 → 정식 전환된 이후 기능 확대 중
- bfloat16은 머신러닝 워크로드에 중요한 데이터 타입으로, VM 내 ML 추론 워크로드에 유용

---

## LoongArch KVM

### 변경사항
- **페이지 테이블 워크(PTW) 감지** — 새 하드웨어에서의 PTW 기능 감지 지원
- **인커널 IPI 에뮬레이션 개선** — 프로세서 간 인터럽트 에뮬레이션 성능 향상
- **PCH-PIC 에뮬레이션 개선** — 플랫폼 인터럽트 컨트롤러 에뮬레이션 개선

---

## FreeBSD Bhyve 하이퍼바이저

### 변경사항
- **Bhyve 하이퍼바이저 감지** — Linux 커널이 FreeBSD Bhyve 하이퍼바이저 위에서 게스트로 실행될 때 감지 가능

### 배경 및 영향
- Bhyve는 FreeBSD의 타입 2 하이퍼바이저. Linux가 Bhyve 게스트로 실행되는 환경에서의 최적화 기반

---

## Dibs (Direct Internal Buffer Sharing)

### 변경사항
- **하이퍼바이저-게스트 메모리 버퍼 공유** — 하이퍼바이저와 Linux 인스턴스 간 메모리 버퍼 직접 공유 메커니즘

### 배경 및 영향
- 호스트-게스트 간 zero-copy 데이터 전달을 위한 기반 인프라
- 가상화 환경에서의 I/O 오버헤드 감소 가능

---

## 이전 버전과의 연속성

- **기밀 컴퓨팅**: 6.15 iommufd PASID/vIOMMU → 6.16 Intel TDX KVM + Hyper-V VTL + SEV-SNP vTPM → 6.18 KVM CET + AMD Secure AVIC + SEV-SNP CipherText Hiding/Secure TSC. 보안 범위가 메모리 암호화에서 제어 흐름 보호까지 확대
- **AMD SEV-SNP**: 매 릴리스마다 기능 추가. 6.18에서 CipherText Hiding과 Secure TSC로 공격 표면 축소
- **guest_memfd**: 게스트 메모리 격리 메커니즘이 점진적으로 발전. 커널 직접 맵 제거를 향한 장기 프로젝트
- **ARM64 KVM**: 6.16 THP/중첩 가상화 → 6.18 FF-A 1.2/마이그레이션/EL2 제어. pKVM 보안 아키텍처 강화
- **RISC-V KVM**: 6.16 정식 전환 → 6.18 FWFT/Zicbop/bfloat16. 정식 전환 이후 기능 확대 가속

---

## 참고 자료

- [Phoronix — KVM Virtualization Sees Several Exciting Improvements For AMD & Intel In Linux 6.18](https://www.phoronix.com/news/Linux-6.18-KVM)
- [kernelnewbies.org — Linux 6.18](https://kernelnewbies.org/Linux_6.18)
- [CNX Software — Linux 6.18 release](https://www.cnx-software.com/2025/12/01/linux-6-18-release-main-changes-arm-risc-v-and-mips-architectures/)
