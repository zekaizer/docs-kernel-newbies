# Linux 6.17 스케줄러 변경사항

> 출처: kernelnewbies.org, LWN.net, LKML
> 릴리스: Linux 6.17
> 작성일: 2026-02-10

---

## 개요

Linux 6.17 스케줄러는 유니프로세서 전용 코드의 완전 제거(SMP 무조건 컴파일), 프록시 실행을 통한 초기 우선순위 역전 방지 메커니즘, sched_ext cgroup 대역폭 제어 인터페이스, workqueue WQ_PERCPU 플래그가 핵심이다. ~50개 이상의 #ifdef 블록이 삭제되어 스케줄러 코드가 대폭 단순화되었다.

---

## SMP 무조건 컴파일

### 변경사항
- **유니프로세서 전용 코드 완전 제거** — 단일 프로세서 시스템 전용 코드 경로(#ifdef CONFIG_SMP 조건부 코드)를 모두 삭제하고, SMP 코드만 유지 (kernelnewbies.org, LWN.net)
- ~50개 이상의 #ifdef 블록 제거
- 단일 코어 시스템에서도 멀티프로세서 코드로 동작하되, 런타임에 단일 CPU 감지

### 배경 및 영향
- 현대 시스템에서 단일 프로세서 구성은 극히 드물며, 유니프로세서 코드 유지는 복잡성만 증가
- 코드 단순화로 스케줄러 버그 발생 가능성 감소 및 유지보수 용이성 향상
- 실질적 성능 영향은 미미하나, 코드 경로 통일로 테스트 커버리지 향상
- 임베디드/IoT의 단일 코어 SoC에서도 SMP 코드로 동작하므로, 미미한 메모리 오버헤드 가능

---

## 프록시 실행 (Proxy Execution)

### 변경사항
- **초기 우선순위 상속(Priority Inheritance) 구현** — 뮤텍스를 보유한 저우선순위 태스크가 대기 중인 고우선순위 태스크의 스케줄링 컨텍스트를 상속하는 메커니즘 (kernelnewbies.org, LWN.net)
- **현재 제한 사항**: 단일 런큐에서만 동작, CONFIG_EXPERT 조건부 활성화

### 배경 및 영향
- 우선순위 역전(Priority Inversion)은 실시간 시스템의 고전적 문제. Mars Pathfinder 사건 이후 priority inheritance가 표준 해결책
- 커널 내부의 뮤텍스 경합에서 발생하는 우선순위 역전을 해결하기 위한 첫 단계
- 단일 런큐 제한으로 현재는 실용성이 제한적이나, 향후 다중 런큐/NUMA 환경으로 확장 예정
- Android의 실시간 오디오 스레드, UI 렌더링 스레드 등 우선순위 민감 워크로드에 장기적 관련성

---

## sched_ext 개선

### 변경사항
- **cgroup 대역폭 제어 인터페이스** — sched_ext에서 cgroup 기반 CPU 대역폭 제한 지원 (kernelnewbies.org)
- BPF 기반 커스텀 스케줄러에서 cgroup의 cpu.max/cpu.weight 인터페이스를 존중

### 배경 및 영향
- 6.16에서 sched_ext의 계층적 구조와 토폴로지 기반 idle CPU 선택이 추가된 데 이어, 6.17에서 cgroup 대역폭 통합
- 컨테이너/cgroup 환경에서 BPF 스케줄러를 사용할 때 기존 리소스 제한 정책과의 호환성 보장
- Android의 cgroup v2 기반 앱 그룹 관리와의 연동 가능성 확대

---

## Workqueue 개선

### 변경사항
- **WQ_PERCPU 플래그** — per-CPU workqueue 생성을 위한 새 플래그 도입
- **system_percpu_wq** — 시스템 수준 per-CPU workqueue 추가

### 배경 및 영향
- Per-CPU workqueue는 CPU 간 캐시라인 바운싱을 제거하여 높은 빈도의 작업 처리에서 성능 향상
- 네트워크 패킷 처리, 타이머 콜백 등 CPU 로컬 작업에 적합

---

## Fair 스케줄링 개선

### 변경사항
- **다른 슬라이스 간 lag 관리** — 서로 다른 타임 슬라이스를 가진 태스크 간의 공정성(lag) 관리 개선
- EEVDF(Earliest Eligible Virtual Deadline First) 스케줄러의 공정성 보장 강화

### 배경 및 영향
- EEVDF는 6.6에서 CFS를 대체한 새 공정 스케줄러. 매 릴리스마다 edge case 수정과 공정성 개선 계속
- 혼합 워크로드(인터랙티브 + 배치) 환경에서의 응답 시간 일관성 향상

---

## 보조 타임키퍼 (Auxiliary Timekeeper)

### 변경사항
- **보조 타임키핑 소스 인프라** — 주 클럭 외의 보조 타임키핑 소스를 위한 인프라 추가

### 배경 및 영향
- PTP(Precision Time Protocol), GPU 타임스탬프, 네트워크 하드웨어 클럭 등 보조 시간 소스의 커널 통합 기반
- 실시간 시스템, 고정밀 타임스탬프 요구 워크로드에 활용

---

## 이전 버전과의 연속성

- **SMP/유니프로세서**: 장기간 유지되어 온 유니프로세서 코드 경로가 6.17에서 최종 제거
- **sched_ext 진화**: 6.15 이벤트 카운터/idle cpumask → 6.16 계층적 구조/토폴로지 → 6.17 cgroup 대역폭
- **EEVDF 성숙**: 6.6 도입 → 매 릴리스 공정성/lag 관리 개선 지속
- **프록시 실행**: 6.17에서 초기 구현 시작. 향후 릴리스에서 확대 예정

---

## 참고 자료
- [kernelnewbies.org Linux 6.17](https://kernelnewbies.org/Linux_6.17)
- [LWN.net 6.17 merge window](https://lwn.net/Articles/1031713/)
- [LWN.net The rest of 6.17 merge window](https://lwn.net/Articles/1032095/)
