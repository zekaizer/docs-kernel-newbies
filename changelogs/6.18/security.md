# Linux 6.18 보안 변경사항

> 출처: kernelnewbies.org, Phoronix, LWN.net
> 릴리스: Linux 6.18
> 작성일: 2026-02-10

---

## 개요

Linux 6.18 보안은 BPF 프로그램 서명 인프라, KVM CET 가상화를 통한 제어 흐름 보호, AMD SEV-SNP CipherText Hiding/Secure TSC, KASAN hw-tags write_only 모드가 핵심이다. fanotify 퍼미션 이벤트 워치독과 out-of-tree 비GPL 모듈 제한 강화도 포함된다.

---

## BPF 프로그램 서명

### 변경사항
- **암호화 서명 인프라** — BPF 프로그램에 대한 디지털 서명 메커니즘 도입
- **보안 정책 적용 기반** — LSM을 통한 서명 검증 정책 구현 가능
- **비특권 로딩 경로** — 서명 검증을 통과한 BPF 프로그램의 비특권 사용자 로딩 기반

### 배경 및 영향
- 커널 모듈 서명과 유사한 보안 수준을 BPF 프로그램에도 적용
- 엔터프라이즈 환경에서 BPF 프로그램의 출처 증명(provenance verification) 가능
- Microsoft Azure의 80만 eBPF 네트워킹 카드 보안 요구사항이 주요 동기
- Android의 eBPF 기반 네트워크 정책에도 서명 검증 적용 가능성

---

## KVM CET (Control-flow Enforcement Technology)

### 변경사항
- **Intel CET 가상화** — Shadow Stack + Indirect Branch Tracking을 VM 게스트에서 활성화
- **AMD CET 가상화** — Shadow Stack 지원

### 배경 및 영향
- ROP/JOP 공격을 하드웨어 수준에서 방어하는 기술이 VM 내부에서도 사용 가능
- 클라우드 환경의 VM 보안 수준 향상
- 호스트와 게스트 모두에서 CET 보호 적용으로 공격 표면 축소

---

## AMD SEV-SNP 보안 강화

### 변경사항
- **CipherText Hiding** — SNP 게스트의 비공개 메모리 암호문에 대한 비인가 CPU 접근 차단 (opt-in)
- **Secure TSC** — 비신뢰 호스트의 게스트 TSC 주파수 변조 방지
- **Secure AVIC** — SEV-SNP VM에서의 보안 가상 인터럽트 컨트롤러

### 배경 및 영향
- CipherText Hiding은 물리적 서버 접근을 가진 공격자에 대한 추가 방어 계층
- Secure TSC는 타이밍 사이드채널 공격 방지
- Secure AVIC는 인터럽트 경로를 통한 정보 유출 차단
- 기밀 컴퓨팅의 보안 깊이(defense-in-depth)가 매 릴리스마다 강화

---

## KASAN hw-tags 개선

### 변경사항
- **write_only 옵션** — 하드웨어 태그 기반 KASAN에서 쓰기 연산만 감시하는 모드 추가

### 배경 및 영향
- KASAN(Kernel Address Sanitizer)의 hw-tags 모드는 ARM MTE를 활용한 메모리 오류 감지
- write_only 모드는 읽기 검사를 스킵하여 성능 오버헤드를 줄이면서 쓰기 관련 메모리 오류(use-after-free, 버퍼 오버플로)는 계속 감지
- 프로덕션 환경에서의 KASAN 활용 가능성 향상. Android에서 특히 관련성 높음

---

## fanotify 퍼미션 이벤트 워치독

### 변경사항
- **퍼미션 이벤트 워치독** — fanotify 퍼미션 이벤트에 대한 워치독 타이머 추가

### 배경 및 영향
- fanotify 퍼미션 이벤트는 파일 접근 정책 결정을 위해 사용자 공간 응답을 대기
- 사용자 공간 데몬이 응답하지 않을 경우 시스템 전체가 멈출 수 있는 문제 방지
- 안티바이러스, 보안 모니터링 소프트웨어의 안정성 향상

---

## 비GPL 모듈 제한 강화

### 변경사항
- **out-of-tree 파일시스템 모듈 제한** — 비GPL out-of-tree 파일시스템 모듈에 대한 추가 제한

### 배경 및 영향
- GPL 라이선스를 준수하지 않는 외부 모듈의 커널 인터페이스 접근을 점진적으로 제한
- 커널 보안 및 안정성 보장을 위한 장기적 방향

---

## Panic 처리 개선

### 변경사항
- **panic 상태 함수 패밀리** — 새로운 panic 상태 조회 함수 세트 추가
- **CONFIG_PANIC_ON_OOPS_VALUE 제거** — 레거시 설정 옵션 정리

### 배경 및 영향
- panic 상태 함수는 커널 패닉 전후의 상태를 더 정확하게 추적 가능
- 크래시 덤프 및 사후 분석 품질 향상

---

## 이전 버전과의 연속성

- **기밀 컴퓨팅 보안**: 6.15 iommufd 인프라 → 6.16 TDX KVM + SEV-SNP vTPM + TSM-MR → 6.18 CET 가상화 + CipherText Hiding + Secure TSC. 보호 범위가 메모리 → 제어 흐름 → 타이밍까지 확대
- **BPF 보안**: 6.16 BPF qdisc → 6.18 BPF 서명. BPF 프로그램의 보안 정책 적용 기반 마련
- **KASAN**: hw-tags write_only 모드 추가로 프로덕션 환경 KASAN 활용 가능성 확대
- **SELinux**: 6.16의 디렉터리 접근 캐시/wildcard genfscon 이후 6.18에서 주요 SELinux 변경 없음

---

## 참고 자료

- [kernelnewbies.org — Linux 6.18](https://kernelnewbies.org/Linux_6.18)
- [LWN.net — Code signing for BPF programs](https://lwn.net/Articles/1017549/)
- [Phoronix — KVM Virtualization Improvements In Linux 6.18](https://www.phoronix.com/news/Linux-6.18-KVM)
