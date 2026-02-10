# Linux 6.16 트레이싱 & BPF 변경사항

> 출처: kernelnewbies.org, LWN.net, LKML
> 릴리스: Linux 6.16
> 작성일: 2026-02-10

---

## 개요

Linux 6.16 트레이싱 & BPF는 BPF qdisc 지원(네트워크 트래픽 제어), rbtree 순회 및 list peek, DMABUF 통계의 BPF 전환, perf의 off-cpu 덤프 및 Rust 디맹글링이 핵심이다. 트레이싱 링 버퍼의 유저스페이스 메모리 매핑도 중요한 변경이다.

---

## BPF

### BPF qdisc
- **BPF struct_ops 기반 큐잉 디시플린** — 10개 커밋으로 BPF 프로그램이 커널 트래픽 제어 qdisc를 구현할 수 있는 프레임워크 추가
- 커널 재빌드 없이 커스텀 네트워크 스케줄링 알고리즘 배포 가능

### 데이터 구조 접근
- **rbtree 순회 (traversal)** — BPF에서 red-black tree 순회 기능 추가
- **list peek** — BPF 리스트에서 비파괴적 peek 연산 지원

### DMABUF 통계
- **BPF를 통한 DMABUF 통계** — 기존 CONFIG_DMABUF_SYSFS_STATS를 BPF 기반 통계로 대체
- sysfs 기반의 성능 오버헤드 없이 필요 시에만 DMABUF 통계 수집 가능

### 기타 BPF 변경
- **prog 인수 suffix** — prog->aux 컨텍스트 전달 지원
- **XDP_REDIRECT for device-bound programs** — 디바이스에 바인딩된 BPF 프로그램에서 XDP 리다이렉트 가능
- **dynptr 메모리 읽기 kfuncs** — 향상된 메모리 접근 kfunc 추가
- **Atomic htab of maps** — maps of maps에 대한 원자적 업데이트 지원
- **bpf_doc.py JSON 출력** — BPF 헬퍼 문서의 JSON 생성 스크립트

### 배경 및 영향
- BPF qdisc는 sched_ext(CPU 스케줄러)에 이은 네트워크 스케줄러의 BPF 프로그래밍 가능화. 클라우드/CDN 환경에서 동적 QoS 정책 적용에 핵심
- DMABUF 통계의 BPF 전환은 sysfs 기반의 상시 오버헤드를 제거하고 온디맨드 모니터링으로 전환. GPU 메모리 추적에 유용
- rbtree/list 접근 확장으로 BPF 프로그램의 커널 데이터 구조 조작 능력 강화

---

## perf

### 변경사항
- **off-cpu 덤프** — 후처리 없이 직접 off-cpu 샘플 덤프 기능. `perf record --off-cpu` 수행 시 별도 분석 단계 불필요
- **Rust 디맹글링** — rustc-demangle을 통한 Rust 심볼 올바른 디코딩
- **cpu event term** — CPU별 이벤트 설정 지원
- **hierarchy 모드 커스텀 필드** — 계층 표시에서 사용자 정의 출력 필드 지원
- **BPF 기반 시스콜 요약** — BPF 백엔드를 통한 시스콜 통계 구현
- **tgid 정렬 키** — 스레드 그룹 ID 기준 정렬
- **cgroup 요약 모드** — cgroup별 시스콜 요약
- **lock contention 플래그** — 락 경합 분석 플래그 추가
- **메트릭 성능 개선** — 최적화 패스로 perf stat 메트릭 처리 속도 향상

### 배경 및 영향
- off-cpu 덤프는 성능 디버깅 워크플로우를 크게 단순화. 기존에는 `perf script`로 후처리가 필요했으나 직접 덤프로 즉시 분석 가능
- Rust 디맹글링은 커널 내 Rust 코드 증가에 따른 필수 도구. perf에서 Rust 함수 이름이 올바르게 표시
- BPF 시스콜 요약은 strace 대비 낮은 오버헤드로 시스콜 패턴 분석

---

## ftrace 및 트레이싱 인프라

### 변경사항
- **트레이싱 링 버퍼 유저스페이스 매핑** — 커널 트레이싱 링 버퍼를 유저스페이스 메모리에 매핑 가능 (LWN.net)
- **콜 그래프 깊이 unsigned 정수 노출** — 콜 그래프 깊이를 unsigned 정수로 노출
- 다수의 트레이싱 인프라 개선

### 배경 및 영향
- 링 버퍼의 유저스페이스 매핑은 트레이싱 데이터를 커널-유저 간 복사 없이 직접 접근할 수 있게 하여, 고빈도 트레이싱 시나리오의 오버헤드 감소
- Android의 perfetto/atrace와 같은 트레이싱 도구가 이 기능을 활용하면 트레이싱 오버헤드 대폭 감소 가능

---

## 이전 버전과의 연속성

- **BPF 프로그래밍 가능 범위**: sched_ext(CPU 스케줄러) → 6.16 qdisc(네트워크 스케줄러)로 커널 서브시스템의 BPF 프로그래밍 범위 계속 확대
- **BPF 데이터 구조**: 6.15 이전 기본 자료구조 → 6.16 rbtree 순회/list peek으로 커널 내부 데이터 구조 접근 확장
- **perf Rust 지원**: 커널 내 Rust 코드 증가에 맞춰 perf 도구 체인도 Rust 심볼 처리 능력 확보

---

## 참고 자료

- [kernelnewbies.org — Linux 6.16](https://kernelnewbies.org/Linux_6.16)
- [LWN.net — The first half of the 6.16 merge window](https://lwn.net/Articles/1022512/)
- [LWN.net — The second half of the 6.16 merge window](https://lwn.net/Articles/1023075/)
