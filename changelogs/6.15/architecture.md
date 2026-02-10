# Linux 6.15 아키텍처 변경사항

> 출처: kernelnewbies.org, Phoronix, LWN.net, LKML
> 릴리스: Linux 6.15
> 작성일: 2026-02-10

---

## 개요

Linux 6.15의 아키텍처 변경은 AMD INVLPGB 브로드캐스트 TLB 무효화 지원이 핵심이다. Zen 3 이상 프로세서에서 IPI 없이 원격 CPU의 TLB를 무효화하여 다중 코어 시스템의 성능을 개선한다. 또한 percpu 데이터 조직 개선과 RISC-V 확장 지원이 추가되었다.

---

## AMD INVLPGB (브로드캐스트 TLB 무효화)

### 변경사항
- **INVLPGB 명령어 지원** — AMD Zen 3+ 프로세서에서 브로드캐스트 TLB 무효화를 수행하는 명령어 활용
- IPI(Inter-Processor Interrupt) 없이 원격 CPU의 TLB 엔트리를 무효화
- Translation Cache Extensions(TCE) 활용 시 대상 PTE로 이어지는 상위 레벨 엔트리만 선택적으로 제거

### 배경 및 영향

**문제**: 기존 TLB 무효화 메커니즘은 원격 CPU의 TLB를 무효화하기 위해 IPI를 전송하고, 해당 CPU가 인터럽트를 처리할 때까지 대기해야 했다. 이는 다중 코어 시스템에서 상당한 오버헤드를 발생시킨다.

**해결**: INVLPGB는 IPI 없이 브로드캐스트 방식으로 원격 CPU의 TLB를 무효화하며, 원격 CPU가 인터럽트를 처리할 때까지 기다릴 필요가 없고, 실행 중인 코드에 대한 간섭을 줄인다.

**벤치마크** (will-it-scale `tlb_flush2_threads` 테스트, AMD Milan 36코어 시스템):

| 구성 | 결과 |
|------|------|
| Vanilla 커널 | 527,000 loops/sec |
| lru_add_drain 제거만 | 731,000 loops/sec |
| INVLPGB만 | 527,000 loops/sec |
| lru_add_drain + INVLPGB | **1,157,000 loops/sec** |

- INVLPGB만 적용 시 TLB 무효화가 전체 CPU 시간의 **40%에서 약 4%로 감소**했으나, 경합이 LRU 락으로 이동
- 두 최적화를 동시 적용 시 초당 반복 횟수가 약 **2.2배** 증가

**적용 범위**:
- x86 PCID 공간이 제한적이므로, 3개 이상의 CPU에서 활성화된 프로세스에 대해서만 브로드캐스트 TLB 무효화 사용. PCID 공간이 소진될수록 임계값이 점진적으로 증가
- "원격 프로세서"는 두 번째 소켓뿐 아니라 다른 CCD(Core Complex Die)도 포함하므로, 일반 Ryzen 데스크탑에서도 이점 제공
- TCE 활성화 시 관련 없는 상위 레벨 엔트리는 그대로 유지하여 불필요한 TLB 미스를 방지

**개발 이력**:
- 2024년 12월 Meta 엔지니어 Rik van Riel이 최초 패치 게시
- 14차례 리뷰를 거쳐 6.15 머지 윈도우(2025년 3월)에 준비 완료
- 후속으로 Intel의 RAR(Remote Action Request) TLB 플러싱 지원이 별도 시리즈로 계획됨

---

## Percpu 데이터 조직

### 변경사항
- **전용 percpu 서브섹션** — 자주 접근되는 프로세서 전용 데이터를 위한 새로운 percpu 서브섹션 도입
- 캐시 지역성 향상 및 멀티코어 환경에서 false sharing 감소

### 배경 및 영향
- Percpu 데이터는 각 CPU가 독점적으로 접근하는 데이터로, 동기화 오버헤드를 제거하는 핵심 기법
- 기존에는 percpu 데이터가 단일 섹션에 배치되어 자주/드물게 접근되는 데이터가 혼재
- 전용 서브섹션으로 hot data를 함께 배치하여 캐시 라인 활용 효율 향상
- 특히 스케줄러, 네트워킹, 메모리 관리 등 고빈도 percpu 접근 경로에서 성능 이점

---

## RISC-V 확장 지원

### 변경사항
- **BFloat16 확장** — 16비트 부동소수점 포맷 지원 (머신러닝 워크로드)
- **Zaamo / Zalrsc** — 원자적 메모리 연산 확장
- **ZBKB** — 비트 조작 및 암호화 관련 명령어 확장

### 배경 및 영향
- RISC-V 생태계의 성숙에 따라 커널이 다양한 ISA 확장을 지원하여 RISC-V 플랫폼의 성능과 기능성을 높이고 있다
- BFloat16은 AI/ML 워크로드에서 메모리 대역폭 절약과 연산 효율 향상에 기여
- Zaamo/Zalrsc는 RISC-V의 원자적 연산 모델을 강화하여 멀티스레드 프로그래밍의 정확성을 보장

---

## x86 32비트 대규모 시스템 지원 제거

### 변경사항
- **32비트 x86에서 8 CPU 초과 또는 4GB RAM 초과 시스템 지원 제거** — 대규모 32비트 시스템에 대한 커널 지원 중단

### 배경 및 영향
- 32비트 x86 시스템의 사용이 급격히 감소함에 따른 코드 정리
- 소규모 임베디드 시스템(8 CPU 이하, 4GB 이하)에서의 32비트 지원은 유지
- 코드 복잡성 감소 및 유지보수 부담 경감

---

## 이전 버전과의 연속성

- **TLB 무효화 최적화**: 6.15의 AMD INVLPGB는 멀티코어 확장성 개선의 핵심 작업이며, 후속으로 Intel RAR 지원이 계획됨
- **아키텍처 정리**: 32비트 대규모 시스템 지원 제거는 커널의 장기적 코드 단순화 트렌드의 일환
- **RISC-V**: 매 릴리스마다 ISA 확장 지원이 추가되며, 커널의 RISC-V 지원이 점진적으로 성숙 중

---

## 참고 자료

- [kernelnewbies.org — Linux 6.15](https://kernelnewbies.org/Linux_6.15)
- [Phoronix — AMD INVLPGB Ready For Linux Kernel](https://www.phoronix.com/news/AMD-INVLPGB-Ready-For-Linux) — 벤치마크 데이터
- [Phoronix — Benchmarking AMD INVLPGB Patches](https://www.phoronix.com/forums/forum/phoronix/latest-phoronix-articles/1515546-benchmarking-the-amd-invlpgb-linux-kernel-patches-for-better-performance)
- [LWN.net — AMD broadcast TLB invalidation](https://lwn.net/Articles/1008065/)
- [LKML — AMD INVLPGB v12 patch series](https://patchew.org/linux/20250221005345.2156760-1-riel@surriel.com/)
