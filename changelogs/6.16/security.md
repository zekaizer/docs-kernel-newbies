# Linux 6.16 보안 변경사항

> 출처: kernelnewbies.org, LWN.net, LKML
> 릴리스: Linux 6.16
> 작성일: 2026-02-10

---

## 개요

Linux 6.16 보안 영역은 SELinux 디렉터리 접근 캐시(Android 부팅 ~15% 단축), wildcard genfscon 지원, AF_UNIX 기반 coredump 소켓, Intel TDX 기밀 컴퓨팅 초기화, TPM kexec 측정이 핵심이다. EFI SecureBoot .sbat 폐지 섹션 지원과 randstruct GCC 플러그인 복원도 주목할 만하다.

---

## SELinux

### 변경사항
- **디렉터리 접근 결정 캐시** — per-task 캐싱으로 디렉터리 접근 결정을 캐시하여 Android 부팅 시간 약 15% 단축 (kernelnewbies.org)
- **Wildcard genfscon 매칭** — genfscon 정책 명세에서 와일드카드를 사용한 sysfs 경로 매칭 지원

### 배경 및 영향
- **디렉터리 접근 캐시**: Android 부팅 과정에서 SELinux 정책 확인은 상당한 오버헤드를 차지. 디렉터리 접근에 대한 결정을 태스크별로 캐시함으로써 반복적인 정책 조회를 제거
  - 15% 부팅 시간 단축은 Android 환경에서의 측정값으로, SELinux 정책이 복잡한 시스템에서 효과가 클수록 극대화
  - per-task 캐싱은 태스크 간 격리를 유지하면서도 동일 태스크의 반복 접근에 대해 캐시 적중률 향상
- **Wildcard genfscon**: 기존에는 각 sysfs 경로를 개별적으로 지정해야 했으나, 와일드카드로 패턴 매칭이 가능해져 정책 작성의 유연성과 유지보수성 향상
  - Android의 복잡한 sysfs 트리에서 특히 유용. 벤더별 sysfs 경로를 패턴으로 일괄 커버 가능

---

## Coredump 소켓

### 변경사항
- **AF_UNIX 소켓을 통한 코어 덤프 전달** — 기존의 파일 기반 또는 사용자 모드 헬퍼 방식 대신 Unix 도메인 소켓으로 코어 덤프를 전달
- 특권 헬퍼 프로세스 없이 유저스페이스에서 코어 덤프 처리 가능

### 배경 및 영향
- 기존 core_pattern의 파이프 방식(|)은 특권 프로세스를 호출해야 하는 보안 위험이 있었음
- AF_UNIX 소켓 기반은 비특권 데몬이 소켓을 수신하여 덤프를 처리할 수 있으므로 공격 표면 감소
- systemd-coredump, crash reporting 서비스 등이 이 새 인터페이스를 활용 가능

---

## TPM 및 Attestation

### 변경사항
- **kexec 이벤트 측정** — kexec load/execute 사이의 이벤트를 TPM으로 측정. 커널 전환 과정의 무결성 검증 강화
- **IMA kexec 측정 지속** — IMA 측정 데이터가 kexec를 통한 커널 전환 시에도 유지. `IMA_KEXEC_EXTRA_MEMORY_KB`로 메모리 예약 설정 가능
- **TSM 측정 레지스터** — sysfs를 통해 Trusted Virtual Machine의 하드웨어 attestation 측정 데이터 노출

### 배경 및 영향
- kexec를 활용한 빠른 커널 업데이트/재부팅 시 측정 체인의 연속성 보장
- 기밀 컴퓨팅(TDX, SEV-SNP) 환경에서 attestation 흐름의 핵심 인프라
- TSM-MR(Measurement Register) ABI의 통합은 다양한 기밀 컴퓨팅 플랫폼 간 attestation 인터페이스 표준화

---

## EFI SecureBoot

### 변경사항
- **.sbat UEFI SecureBoot 폐지 섹션 지원** — EFI 코드에서 .sbat 섹션을 처리하여 UEFI SecureBoot 폐지 메커니즘 지원

### 배경 및 영향
- SBAT(Secure Boot Advanced Targeting)은 특정 부트로더/shim 버전을 선택적으로 폐지하는 메커니즘
- 기존의 dbx(블랙리스트) 방식보다 세밀한 폐지가 가능하여, 취약한 부트로더를 정밀하게 차단
- 보안 부팅 체인의 관리 편의성 향상

---

## 커널 구조체 보호

### 변경사항
- **randstruct GCC 플러그인 복원** — 커널 구조체의 필드 순서를 랜덤화하는 보안 플러그인을 테스트 커버리지와 함께 복원
- **ARM_SSP_PER_TASK 플러그인 제거** — ARM 스택 보호 플러그인 퇴역
- **모듈 .static_call_sites 읽기 전용** — 초기화 이후 정적 호출 사이트를 읽기 전용으로 전환

### 배경 및 영향
- randstruct는 메모리 손상 공격(버퍼 오버플로우로 구조체 필드 조작)을 어렵게 만드는 방어 기법
- .static_call_sites 읽기 전용 전환은 커널 모듈의 함수 포인터 변조 공격 방지

---

## 모듈 export 제한

### 변경사항
- **명시적 export 허용 목록** — 모듈이 접근할 수 있는 커널 심볼을 명시적 허용 목록으로 제한 (kernelnewbies.org: "abuse potential/risk is greatly reduced")

### 배경 및 영향
- 기존에는 EXPORT_SYMBOL로 내보낸 모든 심볼에 모든 모듈이 접근 가능했음
- 명시적 허용 목록으로 필요한 심볼만 접근하도록 제한하여, 악의적 또는 잘못 작성된 모듈의 커널 내부 남용 방지
- GKI(Generic Kernel Image) 환경에서 벤더 모듈의 심볼 접근 관리와 유사한 목적

---

## Intel TDX (기밀 컴퓨팅)

### 변경사항
- **KVM에서 TDX 게스트 VM/VCPU 생성** — Intel Trusted Domain Extensions를 통한 기밀 가상머신 초기 지원
- 암호화된 메모리를 통한 게스트 VM 보호

### 배경 및 영향
- AMD SEV-SNP에 대응하는 Intel의 기밀 컴퓨팅 기술
- 6.15의 iommufd PASID/vIOMMU 인프라에 이어 6.16에서 TDX 초기화로 실질적 기밀 VM 생성 가능
- 상세: [가상화 상세](virtualization.md)

---

## 이전 버전과의 연속성

- **SELinux**: 6.16의 디렉터리 접근 캐시와 wildcard genfscon은 성능과 정책 유연성의 동시 개선. Android 생태계에 직접적 영향
- **기밀 컴퓨팅**: 6.13 ARM CCA → 6.15 iommufd PASID/vIOMMU → 6.16 Intel TDX KVM 초기화 → 6.18 PSP 암호화 → 6.19 PCIe 링크 암호화의 장기 트렌드 지속
- **Coredump**: 전통적 파일/파이프 기반에서 소켓 기반으로의 전환은 보안 모델의 근본적 변화
- **모듈 보안**: export 허용 목록은 GKI의 KMI 제어와 유사한 방향으로, 커널-모듈 인터페이스의 보안 강화

---

## 참고 자료

- [kernelnewbies.org — Linux 6.16](https://kernelnewbies.org/Linux_6.16)
- [LWN.net — The first half of the 6.16 merge window](https://lwn.net/Articles/1022512/)
- [LWN.net — The second half of the 6.16 merge window](https://lwn.net/Articles/1023075/)
- [LWN.net — The 6.16 kernel is out](https://lwn.net/Articles/1031534/)
