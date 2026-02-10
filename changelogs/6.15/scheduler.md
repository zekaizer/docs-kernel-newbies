# Linux 6.15 스케줄러 변경사항

> 출처: kernelnewbies.org, Phoronix, LWN.net, kernel.org
> 릴리스: Linux 6.15
> 작성일: 2026-02-10

---

## 개요

Linux 6.15 스케줄러는 sched_ext 프레임워크의 관측성 및 NUMA 인식 강화, pidfs의 프로세스 생명주기 관리 확장, perf의 스케줄러 기반 지연시간 프로파일링 도입이 핵심이다. EEVDF 코어 스케줄러 자체에는 큰 변화가 없으나, 위에 구축되는 확장 인프라가 크게 성숙했다.

---

## sched_ext (확장 가능 스케줄러)

### 변경사항
- **내부 이벤트 카운터 메커니즘** — sched_ext 내부에서 발생하는 이벤트를 카운팅하고 리포팅하는 메커니즘 추가. Phoronix에 따르면 "subtle corner condition에 대한 가시성을 크게 향상"
- **Per-NUMA idle cpumask 분할** — 기존 글로벌 idle cpumask를 NUMA 노드별로 분할하여 대규모 NUMA 시스템에서의 확장성 개선
- **`SCX_OPS_ALLOW_QUEUED_WAKEUP` 플래그** — BPF 스케줄러에서 큐잉된 wakeup을 허용하는 새로운 연산 플래그
- **`scx_bpf_nr_node_ids()` 추가** — BPF 스케줄러에서 시스템의 NUMA 노드 수를 조회하는 함수
- **ttwu_queue 선택적 토글** — 하드웨어 설계에 따라 try-to-wake-up 큐 동작을 선택적으로 활성화/비활성화
- **기본 idle CPU 선택 로직 개선** — idle CPU 선택 정책 최적화

### 배경 및 영향

**내부 이벤트 카운터**: sched_ext는 6.12에서 메인라인에 합류한 이후 BPF 기반 커스텀 스케줄링 정책의 프레임워크로 빠르게 성장 중이다. 그러나 내부 동작의 관측이 어려워 디버깅과 성능 튜닝이 제한적이었다. 이벤트 카운터는 이 문제를 해결하여, 예를 들어 dispatch 실패, 태스크 마이그레이션, watchdog 트리거 등의 빈도를 추적할 수 있게 한다.

**Per-NUMA idle cpumask**: 대규모 NUMA 시스템(수백 코어)에서 글로벌 idle cpumask를 스캔하는 것은 비효율적이다. NUMA 노드별 분할로 idle CPU 선택 시 같은 NUMA 노드 내의 CPU를 우선하며, 전체 스캔 범위를 줄인다. 내장 idle CPU 선택 정책의 우선순위: fully idle SMT 코어 → 동일 CPU → 동일 LLC 도메인 → 동일 NUMA 노드. LLC 및 NUMA 인식은 시스템에 다중 NUMA 노드 또는 다중 LLC 도메인이 있을 때만 활성화된다.

**watchdog 안전망**: BPF 스케줄러가 일정 시간 내 태스크를 스케줄하지 못하면, watchdog가 자동으로 BPF 스케줄러를 언로드하고 기본 EEVDF 스케줄러로 안전하게 복귀한다. 이는 sched_ext의 프로덕션 환경 안정성을 보장하는 핵심 메커니즘이다.

---

## pidfs (Process ID Filesystem)

### 변경사항
- **`PIDFD_INFO_EXIT` 플래그** — pidfd를 통해 프로세스 reaping 이후에도 exit 정보(exit_code, cgroupid, coredump_mask)를 조회 가능
- **`pidfd_self*` 센티넬** — 자기 자신의 pidfd를 간편하게 참조하는 메커니즘
- **스레드 그룹 핸들링 개선** — 엣지 케이스 처리 강화

### 배경 및 영향
- 기존에는 프로세스가 reap된 후 exit 정보에 접근할 수 없었다. `PIDFD_INFO_EXIT`는 `PIDFD_GET_INFO` ioctl을 통해 이 정보를 stash하여, 프로세스 모니터링 도구와 컨테이너 런타임이 post-mortem 분석을 수행할 수 있게 한다
- `pidfs_exit_info` 구조체에 cgroupid, exit_code, coredump_mask가 저장됨

---

## Perf 스케줄러 기반 지연시간 프로파일링

### 변경사항
- **Wall-time 기반 지연시간 표시** — perf가 스케줄러 컨텍스트 스위치를 추적하여, CPU 시간이 아닌 실제 벽시계 시간(wall-time)으로 샘플을 가중
- 어떤 코드가 실행 지연시간에 불균형적으로 기여하는지 식별 가능

### 배경 및 영향
- 전통적 CPU 프로파일링은 on-CPU 시간만 측정하여, 락 대기, I/O 대기, 스케줄링 지연 등으로 인한 실제 레이턴시를 파악하기 어려웠다
- Wall-time 프로파일링으로 off-CPU 시간까지 포함한 전체 실행 경로의 레이턴시 병목을 정확히 식별
- 실시간 시스템 및 지연시간에 민감한 워크로드(데이터베이스, 네트워크 서비스) 디버깅에 특히 유용

---

## 이전 버전과의 연속성

| 버전 | 스케줄러 주요 변화 |
|------|-------------------|
| 6.12 | sched_ext 메인라인 합류, EEVDF 기본 스케줄러 확립 |
| 6.13 | Lazy preemption 모드 도입, voluntary/full preemption 간 균형 |
| 6.14 | (안정화 릴리스) |
| **6.15** | **sched_ext 이벤트 카운터, per-NUMA cpumask, pidfs exit 정보** |
| 6.17 | Proxy execution (우선순위 상속), unconditional SMP 스케줄러 |

- sched_ext는 6.12 합류 이후 매 릴리스마다 기능이 확장되는 중이며, 6.15에서 관측성(이벤트 카운터)과 NUMA 인식(per-NUMA cpumask)이라는 프로덕션 환경의 핵심 요구사항을 충족
- pidfs는 6.14에서 파일 핸들 지원과 bind-mount가 추가된 데 이어, 6.15에서 exit 정보 조회로 프로세스 생명주기 관리를 완성

---

## 참고 자료

- [kernelnewbies.org — Linux 6.15](https://kernelnewbies.org/Linux_6.15)
- [Phoronix — Sched_Ext Changes Submitted For Linux 6.15](https://www.phoronix.com/news/Linux-6.15-Sched-Ext)
- [Phoronix — sched_ext NUMA Awareness](https://www.phoronix.com/news/sched_ext-NUMA-Awareness)
- [LKML — sched_ext per-NUMA idle cpumask patches](https://lore.kernel.org/lkml/20241126101259.52077-1-arighi@nvidia.com/T/)
- [kernel.org — Extensible Scheduler Class Documentation](https://docs.kernel.org/scheduler/sched-ext.html)
- [kernel.org — EEVDF Scheduler Documentation](https://docs.kernel.org/scheduler/sched-eevdf.html)
