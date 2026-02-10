# Linux 6.15 보안 변경사항

> 출처: kernelnewbies.org, LWN.net, kernel.org
> 릴리스: Linux 6.15
> 작성일: 2026-02-10

---

## 개요

Linux 6.15 보안 영역은 Runtime Verification(RV) 스케줄러 모니터 추가, UBSAN overflow 패턴 제외, iommufd의 PASID/vIOMMU 인프라 확장, io_uring 보안 훅 도입이 핵심이다. fwctl 서브시스템의 보안 모델도 주목할 만하다.

---

## Runtime Verification (RV)

### 변경사항
- **스케줄러 스펙 모니터 추가** — 스케줄러 동작에 대한 형식 스펙을 기반으로 런타임 검증 수행
- RV 프레임워크의 모니터 범위를 스케줄러 서브시스템으로 확장

### 배경 및 영향
- RV는 시스템의 실제 실행 트레이스를 형식 스펙과 비교하여 정확성을 검증하는 경량 기법이다. 전통적 정적 검증(model checking)보다 실용적이며, 런타임 동작에 대한 정확한 정보를 제공
- RV 모니터는 참조 모델, 인스턴스(per-cpu, per-task 등), 그리고 트레이스 포인트와 연결하는 글루 함수로 구성
- 예상치 못한 이벤트에 대한 반응: 로깅, 올바른 동작 강제, 또는 시스템 중단까지 다양한 수준의 리액터를 제공
- 2025년에는 Linear Temporal Logic(LTL) 확장이 Nam Cao에 의해 개발되어 linux-next에 포함됨 (6.17에서 본격 도입). 이는 더 복잡한 시간적 속성을 RV로 검증할 수 있게 하는 기반 작업
- Daniel Bristot de Oliveira가 개척한 오토마타 기반 접근법 위에, 2025년 9월 Gabriele Monaco(Red Hat)가 RV 공동 메인테이너로 합류

---

## UBSAN (Undefined Behavior Sanitizer)

### 변경사항
- **Overflow 패턴 제외 지원** — 패턴 기반으로 특정 정수 오버플로우 경고를 선택적으로 억제
- 세밀한 경고 제어로 의도적인 오버플로우(예: 해시 함수)와 실제 버그를 구분 가능

### 배경 및 영향
- 커널 코드에는 의도적으로 정수 오버플로우를 활용하는 패턴이 다수 존재 (래핑 연산, 해시 계산 등)
- 기존에는 이러한 의도적 패턴도 UBSAN이 경고를 발생시켜, 실제 버그 경고가 노이즈에 묻히는 문제가 있었다
- 패턴 제외로 의도적 사용은 필터링하고 진짜 undefined behavior만 탐지하여 UBSAN의 실효성 향상

---

## iommufd (IOMMU File Descriptor)

### 변경사항
- **PASID attach/replace 지원** — Process Address Space ID를 iommufd를 통해 디바이스에 부착/교체 가능
- **vIOMMU 인프라 확장** — vEVENTQ(가상 이벤트 큐)를 통한 하드웨어 이벤트 리포팅 경로 추가
- **Nested SMMU MSI 매핑 지원** — 중첩 ARM SMMUv3 환경에서의 MSI(Message Signaled Interrupt) 매핑

### 배경 및 영향

**PASID (Process Address Space ID)**:
- PASID와 PRI(Page Request Interface)는 Shared Virtual Addressing(SVA)을 구현하는 핵심 PCI 기능으로, 디바이스의 DMA가 프로세스 가상 메모리 주소로 직접 전달될 수 있게 한다
- iommufd를 통한 PASID attach는 가상머신 내에서 vSVA를 구현하기 위한 기반

**vIOMMU**:
- `IOMMUFD_OBJ_VIOMMU`는 `IOMMU_VIOMMU_ALLOC` uAPI를 통해 생성되며, dev_id와 hwpt_id를 필요로 한다
- vIOMMU 드라이버는 vPASID와 vPRI를 구현하여 VM 내에서 가상 SVA를 달성
- vEVENTQ는 stage-1 이벤트를 threaded IRQ 핸들러에서 리포팅하여, 호스트-게스트 간 IOMMU 이벤트 전달 메커니즘을 제공
- ARM SMMUv3 중첩 환경에서의 MSI 매핑은 디바이스 패스스루 시나리오에서 인터럽트 라우팅의 정확성을 보장

---

## io_uring 보안 훅

### 변경사항
- **io_uring 서브시스템 전용 보안 훅 추가** — 보안 모듈(특히 SELinux)이 io_uring에서 읽는 데이터 종류에 대해 정책 제어를 적용할 수 있게 함

### 배경 및 영향
- Linus Torvalds의 비판에도 불구하고 6.15에 머지됨
- io_uring은 시스콜을 거치지 않는 비동기 I/O 경로를 제공하므로, 전통적 시스콜 기반 보안 모듈의 정책 적용이 우회될 수 있었다
- 이 훅으로 SELinux 등이 io_uring 연산에 대해서도 일관된 보안 정책을 적용 가능

---

## fwctl 보안 모델

### 변경사항
- **fwctl 서브시스템 도입** — 펌웨어 인터페이스를 유저스페이스에 노출하되, 정의된 보안 모델 내에서만 RPC 실행을 허용
- 초기 드라이버: CXL 디바이스, mlx5 네트워크 어댑터, AMD/Pensando 분산 서비스 카드

### 배경 및 영향
- lockdown 커널 활성화 환경이 증가함에 따라, 기존 `/sys/../resource0` 또는 PCI config space 접근에 의존하던 디바이스 관리 도구가 작동하지 않게 됨
- fwctl은 OS와 FW 간의 합의된 보안 규칙 내에서 유저스페이스가 FW RPC를 안전하게 실행할 수 있도록 표준화
- 핵심 제약: fwctl은 다른 커널 서브시스템의 핵심 기능(LBA 읽기/쓰기, 네트워크 패킷 송수신, 가속기 데이터 플레인)과 중복되어서는 안 됨
- CXL은 "command effects" 모델로 명령의 보안 속성을 사전에 학습; mlx5는 ConnectX 및 Bluefield SmartNIC 계열 지원

---

## Training Solo (Spectre v2 변종) 완화

### 변경사항
- **Training Solo 공격 완화 패치** — 분기 예측기를 도메인 내에서 악용하는 Spectre v2 변종에 대한 AMD 및 Intel 프로세서용 패치 포함

### 배경 및 영향
- 하드웨어 취약점에 대한 지속적인 커널 수준 완화 작업의 일환
- 6.17에서는 CPU 버그 완화 선택을 공격 벡터 기반으로 간소화하는 인터페이스가 도입될 예정

---

## 이전 버전과의 연속성

- **RV**: 6.5에서 초기 인프라 도입 이후, 6.15에서 스케줄러 모니터 추가. 6.17에서 LTL 확장 본격 도입 예정
- **iommufd**: 6.2에서 서브시스템 도입 이후 매 릴리스마다 기능 확장. PASID/vIOMMU는 기밀 컴퓨팅 및 디바이스 가상화의 핵심 인프라
- **기밀 컴퓨팅**: 6.13에서 ARM64 CCA, 6.15에서 iommufd PASID/vIOMMU 강화, 6.16에서 Intel TDX 초기화, 6.18에서 PSP 암호화, 6.19에서 PCIe 링크 암호화로 이어지는 장기 트렌드

---

## 참고 자료

- [kernelnewbies.org — Linux 6.15](https://kernelnewbies.org/Linux_6.15)
- [LWN.net — Introduce fwctl subsystem](https://lwn.net/Articles/976833/)
- [LWN.net — Extending run-time verification for the kernel](https://lwn.net/Articles/1030685/)
- [kernel.org — Runtime Verification Documentation](https://docs.kernel.org/trace/rv/runtime-verification.html)
- [kernel.org — fwctl subsystem Documentation](https://docs.kernel.org/userspace-api/fwctl/fwctl.html)
- [kernel.org — IOMMUFD Documentation](https://docs.kernel.org/userspace-api/iommufd.html)
- [Phoronix — New FWCTL Subsystem Submitted For Linux 6.15](https://www.phoronix.com/news/Linux-6.15-fwctl)
