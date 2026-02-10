# Linux 6.17 트레이싱 & BPF 변경사항

> 출처: kernelnewbies.org, LWN.net, LKML
> 릴리스: Linux 6.17
> 작성일: 2026-02-10

---

## 개요

Linux 6.17의 트레이싱 & BPF는 BPF 정밀도 마크 전파(11 커밋), tracing link 쿠키/토큰 인프라, 다수의 새 kfunc(dynptr_memset, cgroup_read_xattr), mprog API cgroup 확장, DYNAMIC_FTRACE 항상 활성화, ftrace graph tracer의 args/retval 옵션이 핵심이다.

---

## BPF 검증기 (Verifier) 개선

### 변경사항
- **정밀도 마크 전파** — 상태 그래프 백에지(backedge)를 통한 정밀도 마크 전파 (11 커밋) (kernelnewbies.org)
- 루프가 있는 BPF 프로그램에서의 검증 정확도 향상

### 배경 및 영향
- BPF 검증기는 프로그램의 안전성을 정적으로 분석하는 핵심 컴포넌트. 정밀도 마크는 어떤 레지스터 값이 정확히 추적되어야 하는지 결정
- 백에지 전파는 루프를 포함하는 프로그램의 검증 정확도를 향상시켜, 이전에 거부되던 정당한 프로그램의 로딩 허용
- BPF 프로그램의 복잡성 한계를 확장하는 중요한 인프라 변경

---

## BPF 쿠키 및 토큰

### 변경사항
- **Tracing link 쿠키** — BPF tracing link에 쿠키 값 연결 지원 (kernelnewbies.org)
- **struct bpf_token_info** — BPF 토큰 정보 구조체 도입
- **BPF map 쿠키 객체** — BPF 맵에 쿠키 객체 연결
- **BPF 메모리 회계** — 프로그램 메모리 사용량 추적

### 배경 및 영향
- 쿠키는 BPF 프로그램과 유저스페이스 간의 상태 전달 메커니즘. tracing link에 쿠키 추가로 다수의 BPF 프로그램을 구분하여 관리 가능
- bpf_token_info는 BPF 프로그램의 권한 및 인증 정보 관리를 위한 인프라
- 메모리 회계는 BPF 프로그램의 리소스 사용 모니터링 강화

---

## 새 kfunc

### 변경사항
- **bpf_dynptr_memset()** — 동적 포인터에 대한 메모리 설정 kfunc
- **bpf_cgroup_read_xattr** — cgroup의 확장 속성(xattr) 읽기 kfunc
- **읽기 전용 문자열 연산 kfunc** — 표준 문자열 연산(strlen, strcmp 등)을 BPF에서 안전하게 사용

### 배경 및 영향
- bpf_dynptr_memset은 BPF 프로그램에서 동적 크기 버퍼를 효율적으로 초기화하는 기능
- cgroup xattr 읽기는 BPF 프로그램이 cgroup 메타데이터에 접근하여 정책 결정에 활용
- 문자열 kfunc는 BPF 프로그램의 데이터 처리 능력 강화. 로그 파싱, 필터링 등에 활용
- 6.16의 BPF qdisc/rbtree 순회에 이어 BPF의 프로그래밍 범위 지속 확대

---

## mprog API 확장

### 변경사항
- **cgroup 프로그램 mprog API** — cgroup BPF 프로그램에 mprog(multi-program) API 적용 (kernelnewbies.org)
- **UID 필터링** — 필터로 이전된 UID 기반 BPF 프로그램 필터링
- **표준 스트림(stdin/stdout/stderr)** — BPF 프로그램에서 표준 스트림 지원

### 배경 및 영향
- mprog API는 여러 BPF 프로그램을 체계적으로 관리하는 인터페이스. XDP/TC에서 사용되던 mprog이 cgroup으로 확대
- Android의 cgroup v2 기반 네트워크 정책 관리에서 다수의 BPF 프로그램을 체계적으로 적용 가능
- 표준 스트림 지원은 BPF 프로그램 디버깅 및 유저스페이스 통신 강화

---

## Ftrace 및 트레이싱 인프라

### 변경사항
- **DYNAMIC_FTRACE 항상 활성화** — 지원 아키텍처에서 DYNAMIC_FTRACE를 기본 활성화 (kernelnewbies.org)
- **fprobe-events lazy 등록** — fprobe 이벤트가 실제 활성화 시에만 등록되어 오버헤드 감소
- **다중 tprobe 같은 tracepoint** — 동일 tracepoint에 여러 tprobe 연결 지원
- **eprobe 배열 처리** — eprobe에서 배열 데이터 처리 지원
- **execmem_rox_cache** — ftrace/kprobes를 위한 읽기-실행 전용 메모리 캐시
- **tracefs 자동 마운트 deprecated** — /sys/kernel/debug/tracing 자동 마운트 2030년 제거 계획

### 배경 및 영향
- DYNAMIC_FTRACE 기본 활성화로 tracing 사용 시 별도 커널 설정 불필요. 프로덕션 커널에서도 동적 트레이싱 즉시 사용 가능
- fprobe lazy 등록은 트레이싱 비활성 시 오버헤드 제로를 보장
- tracefs 자동 마운트 deprecation은 debugfs/tracefs 분리를 위한 장기 계획의 일환

---

## Ftrace Graph Tracer 옵션

### 변경사항
- **args 옵션** — 함수 인자 표시
- **retval 옵션** — 함수 반환값 표시
- **retval-hex 옵션** — 반환값 16진수 표시
- **retaddr 옵션** — 반환 주소 표시

### 배경 및 영향
- ftrace graph tracer의 정보량 대폭 확대. 함수 호출 시 인자와 반환값을 동시에 확인 가능
- 커널 디버깅, 성능 분석에서 함수 호출 흐름과 데이터 흐름을 한 번에 파악

---

## Perf 도구 개선

### 변경사항
- **DRM PMU 지원** — GPU 성능 카운터를 perf에서 수집
- **ftrace 지연 측정 -e 옵션** — 이벤트 간 지연 측정
- **새 ilist 도구** — 명령어 리스트 표시 애플리케이션
- **flamegraph 스크립트 -e/-i 옵션** — 플레임그래프 이벤트 포함/제외 필터
- **perf archive --exclude-buildids** — 빌드 ID 제외 아카이브
- **BPF 메타데이터 수집** — 기존/새 BPF 프로그램의 메타데이터 수집
- **PERF_RECORD_BPF_METADATA 이벤트** — BPF 메타데이터 기록 이벤트
- **show_fdinfo** — perf_event의 fdinfo 표시
- **libcrypto 의존성 제거** — 빌드 의존성 감소

### 배경 및 영향
- DRM PMU 지원은 GPU 워크로드 프로파일링의 커널 수준 통합. 모바일 GPU(Adreno, Mali) 성능 분석에 장기적 관련성
- flamegraph 필터 옵션은 대규모 프로파일링 데이터에서 관심 영역 추출 효율 향상

---

## 이전 버전과의 연속성

- **BPF 프로그래밍 범위**: 6.16 qdisc/rbtree/list → 6.17 cgroup mprog/쿠키/토큰/문자열 kfunc
- **BPF 검증기**: 매 릴리스 검증 정밀도 향상. 6.17의 백에지 정밀도 전파(11 커밋)는 규모가 큰 개선
- **ftrace 진화**: 6.17에서 graph tracer args/retval 옵션 추가 및 DYNAMIC_FTRACE 기본 활성화
- **perf GPU 통합**: 6.16 Rust 디맹글링 → 6.17 DRM PMU 지원

---

## 참고 자료
- [kernelnewbies.org Linux 6.17](https://kernelnewbies.org/Linux_6.17)
- [LWN.net 6.17 merge window](https://lwn.net/Articles/1031713/)
- [LWN.net The rest of 6.17 merge window](https://lwn.net/Articles/1032095/)
