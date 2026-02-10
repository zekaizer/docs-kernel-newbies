# Linux 6.16 커널 변경사항 개요

> 출처: kernelnewbies.org, LWN.net, Phoronix
> 릴리스 날짜: 2025-07-27
> 작성일: 2026-02-10

---

## 주요 기능 (Headline Features)

- **Ext4 대형 folio 및 atomic write** — 일반 파일에 대한 대형 folio 지원으로 대용량 순차 I/O 워크로드에서 37% 성능 향상. bigalloc 파일시스템에서 multi-fsblock atomic write 지원 추가
- **XFS large atomic write** — 다중 블록 원자적 쓰기(all-or-nothing) 연산 구현으로 torn write 방지 기능 본격화
- **DMABUF 기반 zero-copy TCP 전송** — GPU/가속기 버퍼에서 TCP 패이로드를 zero-copy로 전송하는 경로 완성. 6.12의 수신 경로에 이어 송신 경로 추가
- **Intel TDX 초기 지원** — KVM에서 Intel Trusted Domain Extensions를 통한 기밀 가상머신 생성/실행 지원. AMD SEV-SNP에 대응하는 Intel 기밀 컴퓨팅 기술
- **USB 오디오 오프로드** — 2년 이상, 30개 이상 패치 시리즈를 거쳐 머지. 시스템 슬립 중에도 USB 오디오 스트림 유지로 배터리 절약
- **Futex2 확장** — 프로세스 로컬 해시 테이블, FUTEX2_NUMA 토폴로지 인식, FUTEX2_MPOL 메모리 정책 통합으로 동기화 프리미티브 현대화
- **Intel APX 레지스터 확장** — 범용 레지스터를 16개에서 32개로 확장하여 load/store 감소 및 성능·전력 효율 향상

---

## 서브시스템 요약

### 파일시스템
- **Ext4**: 대형 folio로 순차 I/O 37% 향상, fast commit 최적화, bigalloc atomic write
- **Btrfs**: extent buffer writeback 처리량 50% 향상/런타임 33% 감소, extent unpinning 3-5% 개선, 대형 데이터 folio defrag
- **Bcachefs**: 단일 디바이스 모드, 스냅샷 삭제 개선, 비상 읽기 전용 모드, AC 전원 시에만 리밸런스
- **FUSE**: 대형 folio 지원(10 커밋), 단일 연산으로 전체 dentry 캐시 무효화
- **NFS**: 읽기/쓰기 크기 제한 4MB로 확대, FALLOC_FL_ZERO_RANGE 지원, localio sysfs 노출
- **OverlayFS**: 비특권 사용자 네임스페이스에서 data-only 레이어 + dm-verity 지원
- **EROFS**: Intel QAT deflate 하드웨어 가속 압축 해제
- 상세: [파일시스템 상세](filesystems.md)

### 메모리 관리
- **가중 인터리브 자동 튜닝** — NUMA 대역폭 데이터 기반 노드 가중치 자동 재계산
- **DAMON NUMA 자동 튜닝** — 계층형 메모리 티어 최적화
- **Memcg 개선** — IRQ-safe 통계, NMI-safe kmem charging, 비차단 제한 설정
- **Folio API 확장** — folio_mk_pte(), mincore 대형 folio 배치 처리
- **MGLRU** — SWAPPINESS_ANON_ONLY 모드, MADV_DONTNEED/MADV_FREE 배치 TLB 플러시
- 상세: [메모리 관리 상세](memory-management.md)

### io_uring
- io_uring 컨텍스트당 다중 네트워크 큐 지원
- DMABUF 통합 zero-copy RX (zcrx) 확장
- 파이프 생성 기능 추가
- 상세: [io_uring 상세](io-uring.md)

### 스케줄러
- **동적 비대칭 우선순위** — CPU 능력 인식 스케줄링
- **NUMA 밸런스 통계** — 노드 간 태스크 마이그레이션 추적
- **sched_ext**: 계층적 스케줄러 준비, 토폴로지 기반 idle CPU 선택 확대
- **RT 그룹 스케줄링** — 부트 타임 커맨드라인으로 제어 전환
- 상세: [스케줄러 상세](scheduler.md)

### 네트워킹
- **DMABUF TCP TX** — GPU/가속기 메모리에서 TCP zero-copy 전송 완성
- **OpenVPN DCO** — OpenVPN 커널 오프로드 드라이버 도입으로 VPN 성능 향상
- **DCCP 제거** — 2005년 도입된 Datagram Congestion Control Protocol 지원 완전 제거
- **BPF qdisc** — BPF struct_ops 기반 트래픽 제어 큐잉 디시플린 구현
- 상세: [네트워킹 상세](networking.md)

### 보안
- **SELinux 디렉터리 접근 캐시** — per-task 캐싱으로 Android 부팅 시간 ~15% 단축
- **SELinux wildcard genfscon** — sysfs 경로 매칭의 와일드카드 지원
- **Coredump 소켓** — AF_UNIX 소켓을 통한 코어 덤프 전달로 특권 헬퍼 불필요
- **TPM kexec 측정** — load/execute 간 이벤트 측정 인프라
- 상세: [보안 상세](security.md)

### 트레이싱 & BPF
- **BPF qdisc** — 10 커밋으로 BPF 기반 큐 디시플린 지원
- **BPF rbtree 순회 및 list peek** — 데이터 구조 탐색 기능 강화
- **BPF DMABUF 통계** — CONFIG_DMABUF_SYSFS_STATS 대체
- **perf off-cpu 덤프** — 후처리 없이 직접 샘플 덤프
- **perf Rust 디맹글링** — rustc-demangle을 통한 Rust 심볼 처리
- 상세: [트레이싱 & BPF 상세](tracing-bpf.md)

### Rust 지원
- **cpufreq, OPP, clk, cpumask** — 전력 관리 하드웨어 추상화 레이어
- **DRM 추상화** — 그래픽 드라이버 인프라 바인딩
- **nova GPU 드라이버** — NVIDIA 오픈소스 커널 드라이버 초기 통합 진행
- **XArray** — 최소 XArray 바인딩 추가
- **Auxiliary bus** — 디바이스 바인딩 프레임워크
- 상세: [Rust 지원 상세](rust.md)

### 가상화
- **KVM TDX** — Intel TDX 기밀 게스트 VM/VCPU 생성 지원
- **Hyper-V VTL Boot** — Virtual Trust Level 부팅 지원 (11 커밋)
- **virtio_rtc** — 새 실시간 시계 디바이스 모듈
- **ARM64 KVM** — 비보호 게스트에서 THP 지원, 중첩 가상화 (기본 비활성)
- **RISC-V KVM** — 실험적 상태에서 정식 전환
- 상세: [가상화 상세](virtualization.md)

### 블록 레이어
- **바운스 버퍼링 완전 제거** — 블록 레이어 바운스 버퍼 인프라 전면 삭제
- **zram** — 알고리즘별 파라미터 지원, writeback 인터페이스 현대화
- **ublk** — 서버 스레드 디커플링, 자동 bvec 등록, quiesce 기능
- **Zoned 루프 디바이스** — 새 zoned 블록 디바이스 드라이버
- 상세: [블록 레이어 상세](block-layer.md)

### 아키텍처
- **x86**: CONFIG_X86_NATIVE_CPU로 `-march=native` 빌드 최적화, Intel APX 32개 범용 레지스터, 5단계 페이지 테이블 무조건 활성화
- **ARM64**: lazy preemption 지원, Scalable Matrix Extension 지원
- **RISC-V**: getrandom() vDSO 성능 향상, SiFive 벤더 확장, Zicbop/Zabha/Svinval 확장
- **LoongArch**: 최대 CPU 2048개 지원, 멀티코어 스케줄링
- **PowerPC**: 동적 preemption 지원
- 상세: [아키텍처 상세](architecture.md)

### 드라이버
- **USB 오디오 오프로드** — 30개 이상 패치 시리즈, 2년 이상 개발. 슬립 중 오디오 유지
- **NVIDIA Hopper/Blackwell GPU** — nouveau 드라이버에 PCI ID 추가
- **AMDGPU 사용자 모드 큐** — 실험적 지원 제출
- **Intel TDX, QAT GEN6** — 기밀 컴퓨팅 및 암호화 가속
- **Realtek RTL8127A 10GbE** — 새 이더넷 컨트롤러 지원
- **Apple M2 PCIe** — Apple Silicon PCIe 컨트롤러 지원
- 상세: [드라이버 상세](drivers.md)

### 시스콜
- **PTRACE_SET_SYSCALL_INFO** — 차단된 tracee의 시스콜 세부정보 수정 API
- **uselib() 제거** — 더 이상 사용되지 않는 시스콜 완전 삭제
- **Coredump 소켓** — AF_UNIX 기반 코어 덤프 전달

---

## 이전 버전 대비 주요 변화

- 6.15에서 시작된 XFS zoned device 지원에 이어 6.16에서 XFS large atomic write 본격 구현
- 6.15의 Ext4 안정성 중심 업데이트에서 6.16에서는 대형 folio 도입으로 순차 I/O 37% 성능 향상이라는 대폭 개선
- 6.12에서 도입된 DMABUF TCP RX zero-copy에 이어 6.16에서 TX 경로 완성으로 양방향 zero-copy 네트워킹 실현
- 기밀 컴퓨팅: 6.15의 iommufd PASID/vIOMMU 인프라에 이어 6.16에서 Intel TDX KVM 초기화로 실질적 기밀 VM 생성 가능
- Rust: 6.15의 pin-init/hrtimer/mm 바인딩에 이어 6.16에서 cpufreq, OPP, DRM 추상화로 드라이버 작성 기반 대폭 확대
- Futex: 6.15의 기본 futex 인프라에서 6.16에서 프로세스 로컬 해시, NUMA 인식, mempolicy 통합으로 현대화

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
