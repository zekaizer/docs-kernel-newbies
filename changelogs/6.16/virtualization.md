# Linux 6.16 가상화 변경사항

> 출처: kernelnewbies.org, LWN.net, LKML
> 릴리스: Linux 6.16
> 작성일: 2026-02-10

---

## 개요

Linux 6.16 가상화는 Intel TDX 기밀 게스트 초기 지원, Hyper-V Virtual Trust Level 부팅, ARM64 KVM의 THP/중첩 가상화 진전, RISC-V KVM 정식 전환, 새 virtio_rtc 디바이스가 핵심이다. 기밀 컴퓨팅 인프라가 Intel TDX까지 확대된 중요한 릴리스다.

---

## Intel TDX (Trusted Domain Extensions)

### 변경사항
- **KVM TDX 게스트 VM/VCPU 생성** — Intel TDX를 통한 기밀 가상머신 초기화 지원. 암호화된 메모리를 통해 호스트/하이퍼바이저로부터 게스트 VM을 보호

### 배경 및 영향
- TDX는 AMD SEV-SNP에 대응하는 Intel의 기밀 컴퓨팅 기술
- 하이퍼바이저가 게스트 메모리에 접근할 수 없도록 하드웨어 수준에서 격리
- 클라우드 환경에서 다중 테넌트 간 보안 격리의 핵심 기술
- 6.15의 iommufd PASID/vIOMMU 인프라에 이어 TDX 게스트 실행까지 가능해짐
- 초기 지원이므로 기능이 제한적이나, 향후 릴리스에서 라이브 마이그레이션, attestation 등 확장 예상

---

## Hyper-V Virtual Trust Level (VTL)

### 변경사항
- **Virtual Trust Level Boot 지원** — 11개 커밋으로 Hyper-V의 VTL 부팅 인프라 구현

### 배경 및 영향
- VTL은 Hyper-V의 보안 격리 메커니즘으로, 가상머신 내에서 서로 다른 신뢰 수준의 실행 환경을 분리
- Azure의 기밀 컴퓨팅(Confidential VM)과 관련
- Linux 게스트가 Hyper-V VTL 환경에서 올바르게 부팅되고 실행될 수 있도록 지원

---

## ARM64 KVM

### 변경사항
- **비보호 게스트에서 THP (Transparent Huge Pages) 지원** — 보호(protected) KVM에서 비보호 게스트의 THP 사용 허용
- **중첩 가상화 (nested virtualization)** — ARM64에서의 중첩 가상화 진전 (기본 비활성)

### 배경 및 영향
- THP 지원은 가상머신의 메모리 접근 성능을 향상. Stage-2 페이지 테이블에서 대형 페이지 사용으로 TLB 미스 감소
- 중첩 가상화는 VM 내에서 VM을 실행하는 기능. 개발/테스트 및 클라우드 환경에서 유용
- ARM64에서의 중첩 가상화는 장기 프로젝트로, 기본 비활성 상태로 머지되어 안정성 확보 후 활성화 예정

---

## RISC-V KVM

### 변경사항
- **실험적 상태에서 정식 전환** — RISC-V KVM 지원이 실험적(experimental) 태그를 제거하고 정식 기능으로 전환 (LWN.net)

### 배경 및 영향
- RISC-V 아키텍처의 가상화 생태계 성숙을 나타내는 중요한 이정표
- 향후 RISC-V 기반 서버/클라우드 플랫폼에서의 KVM 활용 가능성 확대

---

## SEV/SNP

### 변경사항
- **기능 초기화를 KVM으로 이전** — SEV/SNP 초기화 코드가 KVM 서브시스템으로 이동
- **SVSM vTPM enlightenment** — Secure VM Service Module을 통한 가상 TPM 지원

### 배경 및 영향
- 초기화 코드의 KVM 이전은 코드 구조의 논리적 정리. 기밀 컴퓨팅 초기화가 가상화 서브시스템에 집중
- SVSM vTPM은 기밀 VM 내에서 TPM 기반 attestation/sealing 기능을 제공

---

## virtio_rtc

### 변경사항
- **새 virtio 실시간 시계 디바이스** — 가상머신에서 호스트의 실시간 시계에 접근하는 새 virtio 디바이스 모듈

### 배경 및 영향
- 가상머신의 시간 동기화 정확도 향상에 기여
- 기존 kvm-clock 외에 virtio 표준을 따르는 시계 디바이스로, 다양한 하이퍼바이저 간 호환성 제공

---

## TSM-MR (Measurement Register ABI)

### 변경사항
- **통합 측정 레지스터 ABI** — Trusted Virtual Machine의 측정 레지스터에 대한 통합 인터페이스 (sysfs)

### 배경 및 영향
- TDX, SEV-SNP, ARM CCA 등 다양한 기밀 컴퓨팅 플랫폼의 attestation 측정 데이터에 대한 통합 접근 인터페이스
- 플랫폼에 독립적인 attestation 워크플로우 구현 가능

---

## 이전 버전과의 연속성

- **기밀 컴퓨팅**: 6.13 ARM CCA → 6.15 iommufd PASID/vIOMMU 강화 → 6.16 Intel TDX KVM 초기화 + Hyper-V VTL + TSM-MR 통합. 멀티 벤더 기밀 컴퓨팅 지원이 본격화
- **ARM64 KVM**: 중첩 가상화가 점진적으로 진전. 6.16에서 비보호 게스트 THP 추가로 성능 기반 확보
- **RISC-V KVM**: 실험적 → 정식 전환은 아키텍처 성숙도의 중요한 이정표
- **SEV/SNP**: 지속적 코드 정리 및 vTPM 기능 추가로 AMD 기밀 컴퓨팅 생태계 강화

---

## 참고 자료

- [kernelnewbies.org — Linux 6.16](https://kernelnewbies.org/Linux_6.16)
- [LWN.net — The first half of the 6.16 merge window](https://lwn.net/Articles/1022512/)
- [LWN.net — The second half of the 6.16 merge window](https://lwn.net/Articles/1023075/)
- [Phoronix — Linux 6.16 Features](https://www.phoronix.com/news/Linux-6.16-Features-Early-Look)
