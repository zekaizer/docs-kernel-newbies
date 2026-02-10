# Linux 6.16 스케줄러 변경사항

> 출처: kernelnewbies.org, LWN.net, LKML
> 릴리스: Linux 6.16
> 작성일: 2026-02-10

---

## 개요

Linux 6.16 스케줄러는 동적 비대칭 우선순위 스케줄링, NUMA 밸런스 통계 추적, sched_ext 계층적 스케줄러 준비, 그리고 RT 그룹 스케줄링의 부트 타임 제어 전환이 핵심이다. VMA 스캐닝 최적화로 고정된 NUMA 노드에서의 불필요한 스캔을 제거했다.

---

## 동적 비대칭 우선순위

### 변경사항
- **CPU 능력 인식 스케줄링** — CPU별 처리 능력 차이를 인지하여 동적으로 비대칭 우선순위를 적용하는 스케줄링 지원

### 배경 및 영향
- big.LITTLE/DynamIQ 아키텍처(ARM)나 Intel 하이브리드(P-core/E-core)에서 CPU 코어 간 성능 차이를 스케줄러가 동적으로 반영
- 기존의 정적 CPU capacity 설정에 비해 런타임 상태(서멀 스로틀링, 주파수 변경)에 따른 적응적 스케줄링 가능
- Energy-Aware Scheduling(EAS)과 연계하여 모바일/노트북 환경의 성능-전력 효율 최적화

---

## NUMA 밸런스 통계

### 변경사항
- **노드 간 태스크 마이그레이션 추적** — NUMA 밸런싱에 의한 태스크 마이그레이션 통계를 추적하여 성능 분석 지원
- **VMA 스캐닝 최적화** — 고정된(pinned) NUMA 노드의 VMA 스캔을 건너뛰어 불필요한 오버헤드 제거

### 배경 및 영향
- NUMA 밸런싱 통계는 다중 소켓 시스템에서 워크로드의 메모리 접근 패턴과 마이그레이션 빈도를 가시화
- VMA 스캔 최적화는 대규모 메모리 워크로드에서 NUMA 밸런싱의 CPU 오버헤드 감소
- numactl이나 cgroup의 cpuset으로 노드가 고정된 경우 불필요한 스캔을 제거하여 성능 향상

---

## sched_ext

### 변경사항
- **계층적 스케줄러 준비** — per-instance 상태를 갖는 계층적 스케줄러 구조 기반 작업
- **토폴로지 기반 idle CPU 선택 확대** — CPU affinity가 있는 태스크에도 토폴로지 기반 idle CPU 선택 로직 적용 (LWN.net)

### 배경 및 영향
- sched_ext는 BPF를 통해 커널 재빌드 없이 커스텀 CPU 스케줄러를 배포할 수 있는 프레임워크
- 6.15의 코어 이벤트 카운터, per-NUMA idle cpumask에 이어 6.16에서 계층적 구조와 토폴로지 최적화 확대
- 토폴로지 기반 idle CPU 선택 확대로 affinity가 설정된 태스크에서도 캐시 친화적인 CPU 배치 가능

---

## RT 그룹 스케줄링

### 변경사항
- **커맨드라인 부트 옵션** — 실시간 그룹 스케줄링(rt_group_sched)의 활성화/비활성화를 커널 부트 파라미터로 제어 가능하도록 변경

### 배경 및 영향
- 기존에는 CONFIG 옵션으로 빌드 타임에 결정되었으나, 부트 타임으로 이전하여 동일 커널 이미지로 다양한 배포 시나리오 대응
- 실시간 워크로드(오디오, 산업 제어)와 일반 워크로드 간 유연한 전환 가능

---

## Futex2 확장

### 변경사항
- **프로세스 로컬 해시 테이블** — PROCESS_PRIVATE futex에 대한 프로세스별 해시 테이블로 경합 감소
- **FUTEX2_NUMA** — NUMA 토폴로지를 인식하여 futex 데이터 구조를 소비 프로세스 근처에 배치
- **FUTEX2_MPOL** — 메모리 정책(mempolicy)을 인식한 futex 연산

### 배경 및 영향
- 프로세스 로컬 해시 테이블은 글로벌 해시 테이블의 경합을 제거. 대규모 멀티스레드 애플리케이션(데이터베이스, 게임 엔진)에서 동기화 오버헤드 대폭 감소
- FUTEX2_NUMA는 다중 소켓 시스템에서 futex 접근의 원격 메모리 레이턴시를 최소화
- 이 변경들은 futex를 현대 하드웨어 토폴로지에 맞게 현대화하는 장기 프로젝트의 일환

---

## 이전 버전과의 연속성

- **sched_ext**: 6.12 초기 도입 → 6.15 이벤트 카운터/per-NUMA cpumask → 6.16 계층적 구조/토폴로지 확대. 매 릴리스마다 기능 확장
- **EEVDF**: 6.13~6.15에서 지속 개선. 6.16에서는 EEVDF 자체의 변경보다 sched_ext와 비대칭 우선순위에 집중
- **Futex**: 기본 futex2 인프라에서 6.16에서 프로세스 로컬 해시, NUMA 인식, mempolicy 통합으로 현대화
- **NUMA 밸런싱**: 6.16에서 통계 추적과 VMA 스캔 최적화로 가시성 및 효율 개선

---

## 참고 자료

- [kernelnewbies.org — Linux 6.16](https://kernelnewbies.org/Linux_6.16)
- [LWN.net — The first half of the 6.16 merge window](https://lwn.net/Articles/1022512/)
- [LWN.net — The second half of the 6.16 merge window](https://lwn.net/Articles/1023075/)
