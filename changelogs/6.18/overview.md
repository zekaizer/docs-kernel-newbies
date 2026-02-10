# Linux 6.18 커널 변경사항 개요

> 출처: kernelnewbies.org, LWN.net, Phoronix
> 릴리스 날짜: 2025-11-30
> 작성일: 2026-02-10

---

## 주요 기능 (Headline Features)

- **Slub Sheaves 메모리 할당 캐시** — per-CPU 캐시 메커니즘을 SLUB 할당자에 도입하여 메모리 할당/해제 시 CPU 간 동기화 오버헤드를 크게 감소. kfree_rcu() 배칭 지원으로 RCU 기반 메모리 해제 효율 향상
- **Rust Binder 드라이버** — Android의 핵심 IPC 메커니즘인 Binder 드라이버를 Rust로 재작성하여 병합. 메모리 안전성 강화로 보안 취약점 감소. 기존 C 드라이버와 공존하며 빌드 시 선택 가능
- **Tyr — Rust 기반 Arm Mali GPU 드라이버** — Collabora, Arm, Google 공동 개발. 기존 C 기반 Panthor 드라이버의 Rust 포팅으로, RK3588 등 CSF 기반 Mali GPU 지원. 실험적 상태
- **Bcachefs 제거** — 6.7에서 도입된 실험적 파일시스템 Bcachefs가 메인라인에서 제거. 외부 DKMS 모듈로 전환
- **PSP 암호화 프로토콜** — Google이 개발한 PSP(Protocol Security Protocol) TCP 암호화 지원. IPsec/TLS 대비 우수한 하드웨어 오프로드 능력으로 대규모 데이터센터 네트워크에 최적화
- **Swap Table 인프라 (Phase I)** — 스왑 테이블을 캐시 백엔드로 활용하는 새 인프라 도입. 처리량 5-20% 향상, 커널 빌드 시간 개선. 96 작업 / 10GB zRAM / 64KB mTHP 환경에서 시스템 시간 50% 감소
- **KVM CET 가상화** — Intel Shadow Stack + Indirect Branch Tracking, AMD Shadow Stack을 KVM에서 가상화. Control-flow Enforcement Technology로 VM 내 제어 흐름 공격 방어

---

## 서브시스템 요약

### 파일시스템
- **Btrfs**: block size > page size 준비, 체크섬 검색 시 락 경합 감소로 동기화 시간 대폭 단축, per-filesystem 압축 워크스페이스 관리
- **XFS**: 온라인 fsck 기본 활성화, 새 시스콜 기반 파일시스템 속성 설정, V4 포맷 제거
- **Ext4**: 마운트된 상태에서 슈퍼블록 업데이트 ioctl (tune2fs 호환)
- **FUSE**: COPY_FILE_RANGE_64 대용량 복사, prune 알림, 동기식 FUSE_INIT
- **NFS**: RWF_DONTCACHE 플래그, Direct I/O 정렬 개선, pnfs 대형 extent 배열
- **Bcachefs**: 메인라인에서 완전 제거 (외부 유지보수로 전환)
- 상세: [파일시스템 상세](filesystems.md)

### 메모리 관리
- **Slub Sheaves** — per-CPU 할당 캐시로 할당/해제 동기화 오버헤드 대폭 감소
- **Swap Table (Phase I)** — 스왑 캐시 백엔드로 5-20% 처리량 향상
- **memdesc_flags_t** — struct page/slab/folio 분리를 위한 새 플래그 타입 도입
- **DAMON**: ARM32 LPAE 지원, addr_unit 지원, stat-purpose DAMOS 필터
- **VMA 락**: /proc/pid/maps 읽기에 per-vma lock 적용
- 상세: [메모리 관리 상세](memory-management.md)

### io_uring
- Mixed-size CQE (가변 크기 완료 큐 엔트리) 지원
- uring_cmd multishot + provided buffers
- zcrx (zero-copy receive) 업데이트
- io_uring 기능 조회 인터페이스
- 상세: [io_uring 상세](io-uring.md)

### 스케줄러
- **migrate_enable/disable 인라인화** — 마이그레이션 비활성/활성 함수 인라인으로 오버헤드 감소
- **사용자 공간 복귀 시 스로틀 지연** — 불필요한 커널 내 스로틀 감소
- **sched_ext**: 에러 상태 개선, 마이그레이션 비활성 카운터 추가
- **rseq 최적화**: 사용자 공간 복귀 경로 다중 커밋 효율화
- 상세: [스케줄러 상세](scheduler.md)

### 네트워킹
- **PSP 프로토콜** — Google의 TCP 암호화 스키마, 다중 운영 모드 및 터널링 지원
- **TCP Accurate ECN** — RFC 9768 기반 라운드트립당 다중 혼잡 피드백 신호 지원
- **UDP 수신 50% 성능 향상** — 경합 감소, NUMA 인식 락킹, 데이터 구조 레이아웃 최적화
- **NFS 서버 확장성** — I/O 캐싱 비활성화 프로토타입으로 저메모리 시스템에서도 운용 가능
- 상세: [네트워킹 상세](networking.md)

### 보안
- **BPF 서명 프로그램** — BPF 프로그램 암호화 서명으로 보안 정책 기반 마련
- **KASAN hw-tags write_only** — 하드웨어 태그 기반 KASAN에 쓰기 전용 모드 추가
- **KVM CET 가상화** — VM 내 제어 흐름 강화 기술 지원
- **AMD SEV-SNP CipherText Hiding** — 게스트 메모리 암호문에 대한 비인가 접근 차단
- 상세: [보안 상세](security.md)

### 트레이싱 & BPF
- **BPF 서명 프로그램** — 암호화 서명으로 비특권 사용자의 검증된 BPF 로딩 기반 마련
- **bpf_task_work** — 태스크 컨텍스트에서 지연 실행 스케줄링, NMI 등 비슬리퍼블 컨텍스트에서도 호출 가능
- **검증기 개선** — 경로 비민감 라이브 스택 분석으로 전환
- **bpf_strcasecmp kfunc** — 대소문자 무시 문자열 비교 함수
- 상세: [트레이싱 & BPF 상세](tracing-bpf.md)

### Rust 지원
- **Rust Binder 드라이버** — Android IPC 핵심 드라이버의 Rust 재작성 병합
- **Tyr Mali GPU 드라이버** — CSF 기반 Arm Mali GPU의 Rust DRM 드라이버 (실험적)
- **LKMM atomics** — Linux Kernel Memory Model 원자적 연산 Rust 바인딩
- **USB 드라이버 프레임워크** — USB 드라이버 Rust 바인딩 초기 프레임워크
- **debugfs, sg_table, scatterlist, hrtimer, iov_iter** — 다수 커널 인프라 추상화 추가
- **Rust가 실험적 → 핵심 언어로 승격** — 2025년 12월 커널 개발자 서밋에서 결정
- 상세: [Rust 지원 상세](rust.md)

### 가상화
- **KVM CET** — Intel/AMD 제어 흐름 강화 기술 가상화
- **AMD AVIC 기본 활성화** — Zen 4+ x2AVIC 지원 시스템에서 기본 활성
- **AMD Secure AVIC** — SEV-SNP VM에서의 보안 가상 인터럽트 컨트롤러
- **SEV-SNP CipherText Hiding / Secure TSC** — 게스트 메모리 보호 및 TSC 변조 방지
- **guest_memfd 매핑** — 호스트 사용자 공간에서 guest_memfd 메모리 mmap() 지원
- **ARM64 KVM**: FF-A 1.2 보안 메모리 도관, 마이그레이션 개선
- **RISC-V KVM**: SBI FWFT 확장, Zicbop/bfloat16 게스트 지원
- **LoongArch KVM**: 페이지 테이블 워크 감지, IPI/PCH-PIC 에뮬레이션 개선
- 상세: [가상화 상세](virtualization.md)

### 블록 레이어
- **dm-pcache** — CXL 영속 메모리/DAX 디바이스를 캐시로 활용하는 device mapper 타겟
- **md/llbitmap** — 락프리 비트맵 메커니즘으로 MD RAID 성능 개선
- **블록 무결성** — P2P 소스/목적지 활성화
- 상세: [블록 레이어 상세](block-layer.md)

### 아키텍처
- **ARM64**: Apple M2 Pro/Max/Ultra Device Tree 업스트림, Qualcomm Glymur/SM8750, MediaTek MT8196/MT6991, Samsung Exynos990, Google GS101 CPU Idle, Raspberry Pi 5 확장
- **x86**: microcode= 커맨드라인 옵션, AMD Zen 6 준비, Intel Wildcat Lake 디스플레이, AMD EDAC Venice 지원
- **RISC-V**: RPMI 플랫폼 관리 인터페이스, ESWIN EIC7700 SoC, ioremap_wc(), ACPI 부트 로고
- **LoongArch**: KVM PTW 감지, IPI 에뮬레이션 개선
- 상세: [아키텍처 상세](architecture.md)

### 드라이버
- **Tyr (Arm Mali Rust DRM)** — CSF GPU의 Rust 드라이버 (Collabora/Arm/Google)
- **Nouveau GSP 기본화** — Turing/Ampere에서 NVIDIA GSP 펌웨어 기본 사용
- **Rockchip NPU** — RK3588 NPU 가속기 드라이버 업스트림
- **Google 햅틱 터치패드** — 초기 햅틱 터치패드 지원
- **Sony DualSense** — 오디오 잭 핸들링
- **Intel USBIO** — USB IO Expander 드라이버
- 상세: [드라이버 상세](drivers.md)

### 시스콜
- **RWF_NOSIGNAL** — pwritev2()에서 SIGPIPE 생성 방지 플래그
- **네임스페이스 파일 핸들** — name_to_handle_at()/open_by_handle_at() 네임스페이스 확장
- **procfs "pidns" 마운트 옵션** — 대상 PID 네임스페이스 지정

---

## 이전 버전 대비 주요 변화

- 6.16의 folio 확산(Ext4 37%, FUSE 10 커밋)에 이어 6.18에서는 memdesc_flags_t 도입으로 struct page/slab/folio 근본적 분리 시작
- 6.16의 Intel TDX KVM 초기화에 이어 6.18에서 KVM CET 가상화 + AMD Secure AVIC + SEV-SNP CipherText Hiding으로 기밀 컴퓨팅 대폭 확대
- Rust: 6.16의 cpufreq/OPP/DRM 추상화에 이어 6.18에서 실제 프로덕션 드라이버(Binder, Tyr) 병합. Rust가 실험적 → 핵심 언어로 공식 승격
- Bcachefs: 6.16까지 활발하게 개발되던 파일시스템이 6.18에서 메인라인 제거. 유지보수자와의 커뮤니티 갈등이 원인
- 네트워킹: 6.16의 DMABUF TCP TX zero-copy에 이어 6.18에서 PSP 프로토콜 + Accurate ECN + UDP 50% 최적화로 고성능 네트워킹 강화
- 스왑/메모리: 6.16의 MGLRU/zram 개선에 이어 6.18에서 Swap Table Phase I (5-20% 향상) + Slub Sheaves로 메모리 서브시스템 근본적 개선
- **LTS 커널 가능성**: 6.18은 2025년 마지막 커널 릴리스로, LTS 지정 가능성 높음 (2027년 12월까지 유지보수 예상)

---

## 관련 문서
- [파일시스템 상세](filesystems.md)
- [메모리 관리 상세](memory-management.md)
- [io_uring 상세](io-uring.md)
- [스케줄러 상세](scheduler.md)
- [네트워킹 상세](networking.md)
- [보안 상세](security.md)
- [트레이싱 & BPF 상세](tracing-bpf.md)
- [Rust 지원 상세](rust.md)
- [가상화 상세](virtualization.md)
- [블록 레이어 상세](block-layer.md)
- [아키텍처 상세](architecture.md)
- [드라이버 상세](drivers.md)
- [Android SoC 개발 인사이트](android-soc-insights.md)
