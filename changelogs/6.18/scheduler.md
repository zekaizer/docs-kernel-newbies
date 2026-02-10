# Linux 6.18 스케줄러 변경사항

> 출처: kernelnewbies.org, Phoronix
> 릴리스: Linux 6.18
> 작성일: 2026-02-10

---

## 개요

Linux 6.18 스케줄러는 대규모 구조적 변경보다는 효율화와 안정성에 초점을 맞추었다. migrate_enable/disable 인라인화, 사용자 공간 복귀 시 스로틀 지연, rseq 최적화, sched_ext 에러 상태 개선 등이 포함된다. CFS 스케줄러의 스로틀 지연은 지연 시간 최적화에 기여한다.

---

## migrate_enable/disable 인라인화

### 변경사항
- **인라인 함수 전환** — migrate_enable()/migrate_disable() 함수를 인라인으로 변경하여 함수 호출 오버헤드 제거

### 배경 및 영향
- migrate_enable/disable은 선점 가능 커널에서 태스크의 CPU 마이그레이션을 제어하는 핵심 함수
- 빈번하게 호출되는 경로에서 인라인화를 통해 미세한 성능 향상 확보
- PREEMPT_RT 환경에서 특히 유의미한 개선

---

## 사용자 공간 복귀 시 스로틀 지연

### 변경사항
- **커널→유저 전환 시 스로틀 실행 지연** — 태스크가 사용자 공간으로 복귀할 때까지 CFS 대역폭 스로틀 실행을 지연

### 배경 및 영향
- 기존에는 커널 내부 실행 중에도 스로틀이 적용되어 불필요한 스케줄링 지연 발생
- 스로틀을 사용자 공간 복귀 시점까지 지연함으로써 커널 내 크리티컬 섹션 중단 방지
- 컨테이너 환경에서 CFS 대역폭 제한 사용 시 레이턴시 감소에 기여

---

## rseq 최적화

### 변경사항
- **사용자 공간 복귀 경로 최적화** — 다수 커밋으로 rseq(restartable sequences)의 사용자 공간 복귀 경로 효율화

### 배경 및 영향
- rseq는 사용자 공간에서 per-CPU 연산을 효율적으로 수행하기 위한 메커니즘
- 사용자 공간 복귀 경로의 오버헤드 감소는 rseq 기반 고성능 라이브러리(glibc 등)에 직접 영향
- 빈번한 시스콜 패턴에서 전반적 성능 향상 기여

---

## sched_ext 개선

### 변경사항
- **에러 상태 개선** — sched_ext 에러 상태 덤프에 마이그레이션 비활성 카운터 추가
- **도구 업데이트** — 업스트림 리포지토리에서 sched_ext 도구 동기화

### 배경 및 영향
- sched_ext는 BPF를 통한 커스텀 CPU 스케줄러 구현 프레임워크
- 마이그레이션 비활성 카운터는 sched_ext 스케줄러의 디버깅을 용이하게 함
- 6.16의 계층적 구조/토폴로지 기반 idle CPU 선택에 이어 안정성 및 디버깅 중심 개선

---

## Softirq PREEMPT_RT 개선

### 변경사항
- **softirq-BKL 락 드롭 가능** — PREEMPT_RT에서 softirq의 Big Kernel Lock을 드롭할 수 있는 기능 추가

### 배경 및 영향
- PREEMPT_RT에서 softirq 처리의 선점 가능성 향상으로 실시간 지연 시간 개선
- 산업용/자동차 리눅스 등 실시간 요구사항이 있는 환경에 유용

---

## Cgroup 개선

### 변경사항
- **per-threadgroup resem** — cgroup.procs 쓰기에 사용되는 전역 percpu_rwsem을 per-threadgroup resem으로 교체
- **로컬 시간 통계** — cgroup.stat에 로컬 시간 통계 추가

### 배경 및 영향
- 전역 percpu_rwsem은 다수의 컨테이너가 동시에 cgroup.procs를 변경할 때 경합의 원인
- per-threadgroup 전환으로 컨테이너 밀도가 높은 환경에서의 확장성 향상
- 로컬 시간 통계는 cgroup 내 태스크의 CPU 시간 소비를 더 정확히 추적

---

## 이전 버전과의 연속성

- **sched_ext**: 6.15 이벤트 카운터/per-NUMA idle cpumask → 6.16 계층적 구조/토폴로지 idle CPU → 6.18 에러 상태 개선. 기능 확대 후 안정성/디버깅 강화 단계
- **EEVDF**: 6.18에서 주요 EEVDF 변경은 없으나, CFS 스로틀 지연을 통한 효율화 지속
- **실시간**: PREEMPT_RT softirq-BKL 드롭, migrate 인라인화 등 실시간 커널의 점진적 개선
- **Cgroup**: per-threadgroup resem은 cgroup v2 환경의 확장성을 위한 중요한 인프라 변경

---

## 참고 자료

- [kernelnewbies.org — Linux 6.18](https://kernelnewbies.org/Linux_6.18)
- [Phoronix — Linux 6.18 Features Expected](https://www.phoronix.com/news/Linux-6.18-Features-Expected)
