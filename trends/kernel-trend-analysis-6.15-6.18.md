# 커널 트렌드 분석: Linux 6.15 — 6.18

> 작성일: 2026-02-10
> 대상 버전: Linux 6.15 (2025-05-15), Linux 6.16 (2025-07-27), Linux 6.18 (2025-11-30)

---

## 개요

Linux 6.15~6.18 세 릴리스에서 관찰되는 주요 기술 트렌드를 분석한다. Rust의 핵심 언어 승격과 프로덕션 드라이버 병합, struct page 분리 프로젝트 시작(memdesc_flags_t), 메모리 할당/스왑 인프라의 근본적 개선(Slub Sheaves, Swap Table), 기밀 컴퓨팅의 제어 흐름 보호까지 확대, BPF 서명 프로그램 인프라가 6.18의 신규 핵심 트렌드이다.

---

## 1. Rust 커널 드라이버 생태계 — 실험에서 프로덕션으로

| 버전 | 주요 진전 |
|------|----------|
| 6.15 | pin-init 독립 크레이트, hrtimer API, mm_struct/vm_area_struct/mmap 바인딩 |
| 6.16 | cpufreq/OPP/clk/cpumask 전력 관리 추상화, DRM 그래픽 인프라, nova GPU 진행, XArray, auxiliary bus, configfs |
| 6.18 | **Rust Binder 드라이버 병합**, **Tyr Mali GPU Rust DRM 드라이버**, USB 프레임워크, LKMM atomics, debugfs/sg_table/iov_iter/maple tree/request_irq 추상화, **Rust 핵심 언어 승격** |

**트렌드 분석**: 6.15~6.16의 인프라 구축 단계를 지나 6.18에서 결정적 전환이 발생. Android 핵심 IPC인 Binder와 Arm Mali GPU 드라이버라는 실제 프로덕션 드라이버가 병합되었고, 2025년 12월 커널 개발자 서밋에서 Rust가 C/어셈블리와 동등한 핵심 언어로 공식 승격. 양적 성장(추상화 수)에서 질적 전환(프로덕션 드라이버)으로의 도약.

**Android SoC 영향**: Rust Binder는 Android 보안 모델의 핵심에 영향. 현 시점에서는 C 드라이버 유지 권장이나, 장기적으로 Rust Binder가 기본이 될 것. 벤더 드라이버 팀의 Rust 역량 확보 계획 필요.

---

## 2. struct page/slab/folio 분리 — 대형 folio에서 근본적 재구성으로

| 버전 | 주요 진전 |
|------|----------|
| 6.15 | Btrfs extent_io.c folio 준비, F2FS folio 전환 계속, batched unmap lazyfree |
| 6.16 | Ext4 일반 파일 대형 folio (37% I/O 향상), FUSE 대형 folio (10 커밋), Btrfs defrag 대형 folio, folio_mk_pte() API |
| 6.18 | **memdesc_flags_t 도입** (page/slab/folio 분리 기반), kernel file mapped folios, persistent huge zero folio, filemap_map_pages refcount 최적화, MM 플래그 64비트 통일 |

**트렌드 분석**: 6.15~6.16의 "대형 folio 확산"이 6.18에서 한 단계 더 진화하여 struct page 자체의 근본적 재구성으로 발전. memdesc_flags_t는 page/slab/folio를 타입 안전하게 분리하기 위한 첫 걸음으로, 커널 메모리 관리의 가장 큰 구조적 변화 중 하나가 시작됨. 동시에 folio API도 계속 확장(kernel file mapped folios, persistent huge zero folio).

**Android SoC 영향**: memdesc_flags_t는 struct page 내부 플래그에 직접 접근하는 벤더 드라이버의 호환성에 영향. GKI 모듈 ABI 검증 필요.

---

## 3. 메모리 할당 및 스왑 인프라 혁신

| 버전 | 주요 진전 |
|------|----------|
| 6.15 | defrag_mode sysctl, per-CMA 락, batched unmap lazyfree, DAMON 자동 튜닝 |
| 6.16 | MGLRU SWAPPINESS_ANON_ONLY, 배치 TLB 플러시, DAMON NUMA 자동 튜닝, memcg NMI-safe, zram 알고리즘별 파라미터 |
| 6.18 | **Slub Sheaves** (per-CPU 할당 캐시), **Swap Table Phase I** (5-20% 처리량 향상), 스왑 클러스터 스캐닝 개선, DAMON ARM32 LPAE, kmalloc_nolock 재진입, Rust 대형 정렬/NUMA 할당자 |

**트렌드 분석**: 6.15~6.16의 세밀한 최적화(MGLRU, zram 파라미터, memcg)를 넘어 6.18에서 인프라 수준의 근본적 변화 발생. Slub Sheaves는 SLUB 할당자의 가장 큰 구조적 변화이며, Swap Table Phase I은 스왑 캐시 백엔드의 새 설계. 이 두 변화는 후속 릴리스에서 더 큰 개선의 기반이 될 것.

**Android SoC 영향**: Swap Table의 zram 환경 5-20% 처리량 향상은 메모리 제약 모바일 디바이스에서 가장 체감 효과가 큰 변화. 앱 스위칭, 저메모리 시나리오 성능 검증 권장.

---

## 4. 기밀 컴퓨팅 — 메모리에서 제어 흐름까지

| 버전 | 주요 진전 |
|------|----------|
| 6.15 | iommufd PASID attach/replace, vIOMMU vEVENTQ, ARM SMMUv3 중첩 MSI 매핑 |
| 6.16 | Intel TDX KVM 초기화, Hyper-V VTL Boot, SEV/SNP SVSM vTPM, TSM-MR 통합 ABI |
| 6.18 | **KVM CET 가상화** (Intel Shadow Stack + IBT, AMD Shadow Stack), **AMD Secure AVIC**, **SEV-SNP CipherText Hiding/Secure TSC**, guest_memfd 매핑, ARM64 FF-A 1.2, **PSP 암호화 프로토콜** |

**트렌드 분석**: 보호 범위가 매 릴리스마다 확대. 6.15 IOMMU 인프라 → 6.16 메모리 암호화(TDX, SEV-SNP) → 6.18 제어 흐름 보호(CET) + 타이밍 보호(Secure TSC) + 네트워크 암호화(PSP). 기밀 컴퓨팅이 CPU/메모리에서 제어 흐름, 인터럽트, 네트워크 경로까지 전 계층을 포괄하는 방향으로 발전.

**향후 전망**: 6.19에서 PCIe 링크 암호화가 추가되면 I/O 경로까지 보호 범위 확대. 완전한 end-to-end 기밀 컴퓨팅 달성에 근접.

---

## 5. BPF 보안 및 프로그래밍 확대

| 버전 | 주요 진전 |
|------|----------|
| 6.15 | io_uring 보안 훅, 기존 XDP/TC 강화 |
| 6.16 | BPF qdisc (10 커밋), rbtree 순회/list peek, DMABUF 통계 BPF 전환, atomic htab of maps |
| 6.18 | **BPF 서명 프로그램**, bpf_task_work (태스크 컨텍스트 지연 실행), 검증기 경로 비민감 분석, bpf_strcasecmp, skb 메타데이터 dynptr |

**트렌드 분석**: 6.15~6.16의 "프로그래밍 범위 확대"(qdisc, 데이터 구조 조작)에 이어 6.18에서 "보안 인프라" 차원이 추가됨. BPF 서명은 커널 모듈 서명과 유사한 보안 수준을 BPF에 적용하여, 비특권 사용자의 검증된 BPF 로딩이라는 새 패러다임을 열 가능성. bpf_task_work은 BPF의 실행 컨텍스트 제약을 극복하는 중요한 런타임 확장.

---

## 6. io_uring 기능 안정화

| 버전 | 주요 진전 |
|------|----------|
| 6.15 | zero-copy 수신, epoll 이벤트 읽기, vectored registered buffer |
| 6.16 | 다중 네트워크 큐, DMABUF zcrx 확장, 파이프 생성, DMABUF TCP TX |
| 6.18 | mixed-size CQE, uring_cmd multishot + provided buffers, zcrx 업데이트, 기능 조회 인터페이스 |

**트렌드 분석**: 6.15~6.16의 급속한 기능 확장(zero-copy 양방향, DMABUF, 다중 큐) 이후 6.18에서는 기존 기능의 유연성 향상(mixed-size CQE, multishot)과 인프라 정비(기능 조회)에 초점. 성숙 단계로의 전환 징후.

---

## 7. 스케줄러 프로그래밍 가능성

| 버전 | 주요 진전 |
|------|----------|
| 6.15 | sched_ext 이벤트 카운터, per-NUMA idle cpumask, SCX_OPS_ALLOW_QUEUED_WAKEUP |
| 6.16 | sched_ext 계층적 구조, 토폴로지 기반 idle CPU 선택 확대, 동적 비대칭 우선순위 |
| 6.18 | sched_ext 에러 상태 개선, migrate 인라인화, CFS 스로틀 지연, softirq PREEMPT_RT 개선, cgroup per-threadgroup resem |

**트렌드 분석**: sched_ext는 6.15~6.16의 기능 확대(계층 구조, 토폴로지 인식) 후 6.18에서 안정성/디버깅(에러 상태) 단계로 진입. CFS 스로틀 지연과 PREEMPT_RT softirq 개선은 실시간 성능 향상 방향. cgroup per-threadgroup resem은 컨테이너 환경의 확장성 인프라 변경.

---

## 8. Bcachefs — 성숙에서 제거로

| 버전 | 주요 진전 |
|------|----------|
| 6.15 | 스크러빙, 페이지 초과 블록 크기, case-folding, 디스크 포맷 변경 |
| 6.16 | 단일 디바이스 모드, 비상 읽기 전용, 스냅샷 삭제 개선, 복구 인프라, AC 전원 리밸런스 |
| 6.18 | **메인라인에서 완전 제거**. 외부 DKMS 모듈로 전환 |

**트렌드 분석**: 6.15~6.16에서 활발하게 프로덕션 기능을 추가하고 안정화하던 Bcachefs가 6.18에서 예상치 못하게 메인라인에서 제거됨. 기술적 성숙도보다 커뮤니티 프로세스 준수가 메인라인 유지에 더 중요하다는 교훈. 이전 트렌드 분석의 "프로덕션 환경에 점점 가까워지는 추세" 예측이 커뮤니티 동학에 의해 뒤집힌 사례.

---

## 9. SELinux 성능 및 유연성

| 버전 | 주요 진전 |
|------|----------|
| 6.15 | (주요 SELinux 변경 없음) |
| 6.16 | 디렉터리 접근 캐시 (Android 부팅 ~15% 단축), wildcard genfscon |
| 6.18 | (주요 SELinux 변경 없음) |

**트렌드 분석**: 6.16의 디렉터리 접근 캐시/wildcard genfscon이 최신 상태. 6.18에서 추가 SELinux 변경은 없으나, 6.16의 개선이 Android 생태계에 충분히 큰 영향을 미침.

---

## 10. Atomic Write 확대

| 버전 | 주요 진전 |
|------|----------|
| 6.15 | XFS COW 기반 large atomic write, zoned device 확장 |
| 6.16 | XFS large atomic write 본격화, Ext4 bigalloc multi-fsblock atomic write |
| 6.18 | (주요 atomic write 변경 없음) |

**트렌드 분석**: 6.15~6.16에서 XFS/Ext4에 확산된 atomic write는 6.18에서 추가 확장 없이 안정화 단계. 기본 인프라가 갖춰졌으므로 향후 릴리스에서 다른 파일시스템이나 스토리지 스택으로 확산 가능.

---

## 11. (신규) 네트워크 고성능 암호화

| 버전 | 주요 진전 |
|------|----------|
| 6.15 | (기존 네트워크 최적화) |
| 6.16 | DMABUF TCP TX zero-copy, 다중 네트워크 큐 |
| 6.18 | **PSP 프로토콜** (Google TCP 암호화), **TCP Accurate ECN**, **UDP 수신 50% 향상** |

**트렌드 분석**: 6.18에서 데이터센터 네트워크 암호화의 새 패러다임인 PSP 프로토콜이 도입되고, TCP 혼잡 제어(Accurate ECN)와 UDP 성능이 대폭 개선. 고성능 네트워킹의 암호화 + 혼잡 제어 + 프로토콜 최적화가 동시에 진행되는 새 트렌드.

---

## 종합 요약

6.15~6.18에서 가장 두드러진 트렌드:

1. **Rust의 프로덕션 전환** — 인프라(6.15~6.16) → 프로덕션 드라이버(Binder, Tyr) + 핵심 언어 승격(6.18). 가장 극적인 변화
2. **메모리 서브시스템 근본적 혁신** — folio 확산(6.15~6.16) → memdesc_flags_t + Slub Sheaves + Swap Table(6.18). 할당자와 스왑 인프라 모두 재설계
3. **기밀 컴퓨팅 전 계층 확대** — IOMMU(6.15) → 메모리 암호화(6.16) → 제어 흐름 + 타이밍 + 네트워크(6.18). defense-in-depth 완성 방향
4. **BPF 보안 차원 추가** — 프로그래밍 범위(6.15~6.16) → 서명 인프라(6.18). 보안과 유연성의 동시 추구
5. **Bcachefs 교훈** — 기술적 성숙도 ≠ 메인라인 유지. 커뮤니티 프로세스의 중요성

**Android SoC 관점 핵심 영향**:
- 6.18 LTS 가능성으로 차기 GKI 베이스라인 전략 수립 필요
- Rust Binder 도입으로 빌드/검증 프로세스 업데이트 필요
- Swap Table + Slub Sheaves로 메모리 성능 검증 필수
- memdesc_flags_t로 벤더 드라이버 struct page 접근 코드 감사 필요
- KASAN hw-tags write_only로 프로덕션 MTE 활용 전략 수립 가능
