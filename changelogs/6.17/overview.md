# Linux 6.17 커널 변경사항 개요

> 출처: kernelnewbies.org, LWN.net, Phoronix, CNX Software
> 릴리스 날짜: 2025-09-28
> 작성일: 2026-02-10

---

## 주요 기능 (Headline Features)

- **CPU 취약점 완화 공격 벡터 제어(AVC)** — x86 아키텍처에서 CPU 취약점 완화 옵션을 공격 벡터 단위로 제어하는 새 프레임워크 도입. 새로운 취약점 발견 시 관리자가 개별 파라미터를 재설정할 필요 없이 공격 벡터 기반으로 자동 적용
- **file_getattr/file_setattr 시스콜** — openat(2) 시맨틱으로 확장 파일 속성을 조작하는 두 개의 새 시스템 콜 추가. 파일을 열지 않고도 inode 속성 조작 가능
- **SMP 스케줄러 무조건 컴파일** — 단일 프로세서 전용 코드 경로 완전 제거, ~50개 이상의 #ifdef 블록 삭제로 스케줄러 코드 단순화. 단일 코어 시스템에서도 SMP 코드로 동작
- **프록시 실행(Priority Inheritance)** — 뮤텍스를 보유한 저우선순위 태스크가 대기 중인 고우선순위 태스크의 스케줄링 컨텍스트를 상속하는 초기 메커니즘. 현재 단일 런큐, CONFIG_EXPERT 조건부
- **NUMA 노드별 선제적 메모리 회수** — memcg 인터페이스를 넘어 NUMA 노드별 유저스페이스 메모리 회수 제어 가능: `echo 512M swappiness=10 > /sys/devices/system/node/nodeX/reclaim`
- **DualPI2 혼잡 제어 프로토콜** — L4S(Low Latency, Low Loss, Scalable Throughput) 기반의 이중 PI 큐 혼잡 제어로 게이밍/실시간 애플리케이션의 네트워크 지연 감소
- **ARM64 커널 라이브 패칭** — 64비트 ARM 시스템에서 커널 라이브 패칭 지원 추가. 재부팅 없이 커널 보안 패치 적용 가능

---

## 서브시스템 요약

### 파일시스템
- **Btrfs**: 실험적 대형 데이터 folio 지원(CONFIG_BTRFS_EXPERIMENTAL), free space tree 최적화(빈 파일 생성 런타임 20% 개선), xarray 인덱싱, 압축 데이터 readahead 개선
- **Ext4**: 블록 할당 확장성 개선, uncached buffered I/O 지원, FALLOC_FL_WRITE_ZEROES 지원
- **NFS**: 쓰기 위임(write delegation), 비동기 NFSv4.2 COPY 재활성화, max-ops-per-compound 제한 제거, btime/TLS keyring 지원
- **F2FS**: 새 mount API 전환, GC 부스트 sysfs 튜닝 노드, 복구 통계 계정
- **FUSE**: iomap 기반 버퍼 쓰기 및 writeback
- **EROFS**: 메타데이터 압축, 디렉터리 readahead 성능 향상
- **OverlayFS**: case-folding 파일시스템 레이어 지원
- 상세: [파일시스템 상세](filesystems.md)

### 메모리 관리
- **대형 folio mprotect() 3배 이상 속도 향상**, mremap() 37% 실행 시간 단축
- **NUMA 노드별 선제적 회수** — memcg 외 per-node 제어 인터페이스
- **Per-VMA lock /proc/pid/maps 읽기**, multi-VMA mremap() 이동, MADV_DONTNEED per-VMA lock
- **DAMON_STAT** — 간소화된 접근 모니터링 모듈
- **Hugetlb folio 예약 후 할당**, frozen pages for large kmalloc
- 상세: [메모리 관리 상세](memory-management.md)

### io_uring
- 전송 타임스탬프 명령, 벡터화된 전송 지원
- 테스트용 mock 파일 인프라
- multishot 수신 길이 제한, NOP_TW 완료 플래그
- 상세: [io_uring 상세](io-uring.md)

### 스케줄러
- **SMP 무조건 컴파일** — 유니프로세서 코드 완전 제거
- **프록시 실행** — 우선순위 역전 방지를 위한 초기 구현
- **sched_ext**: cgroup 대역폭 제어 인터페이스
- **Workqueue**: WQ_PERCPU 플래그 및 system_percpu_wq 추가
- 상세: [스케줄러 상세](scheduler.md)

### 네트워킹
- **DualPI2** — L4S 기반 이중 PI 큐 혼잡 제어 프로토콜 메인라인 통합
- **MCTP 게이트웨이 라우팅** — Management Component Transport Protocol 게이트웨이 지원
- **TCP_MAXSEG 소켓 옵션** — 멀티패스 TCP 지원
- **IPv6 force_forwarding sysctl** — 인터페이스별 포워딩 활성화
- **RFC 6675 손실 감지 제거** — 2018년 이후 기본값으로 사용되지 않았던 구식 프로토콜 제거
- 상세: [네트워킹 상세](networking.md)

### 보안
- **공격 벡터 제어(AVC)** — Spectre, Meltdown 등 CPU 취약점 완화를 위협 클래스 단위로 관리하는 통합 프레임워크
- **코어 덤프 소켓 확장** — 개별 코어 덤프에 대한 서버 지시 처리로 보안 강화
- **CRC 코드 대규모 재작업**, SHA-1/SHA-2 생성 새 API
- 상세: [보안 상세](security.md)

### 트레이싱 & BPF
- **BPF 정밀도 마크 전파** — 상태 그래프 백에지를 통한 정밀도 마크 전파 (11 커밋)
- **BPF 토큰/쿠키** — tracing link 쿠키, bpf_token_info 구조체, map 쿠키 객체
- **bpf_dynptr_memset()** kfunc, **bpf_cgroup_read_xattr** kfunc
- **mprog API** — cgroup 프로그램 확장
- **DYNAMIC_FTRACE 항상 활성화**, fprobe-events lazy 등록
- **ftrace graph tracer**: args/retval/retval-hex/retaddr 옵션
- **perf**: DRM PMU 지원, ftrace 지연 측정 -e 옵션, ilist 도구, flamegraph -e/-i 옵션
- 상세: [트레이싱 & BPF 상세](tracing-bpf.md)

### Rust 지원
- **regulator 서브시스템** — 전압 레귤레이터 추상화
- **firmware properties** — 디바이스 프로퍼티 읽기 바인딩
- **I/O 리소스 및 I/O 메모리** — 플랫폼 I/O 지원
- **ACPI 매치 테이블**, bits/genmask 매크로
- **hrtimer Instant/Delta 변환**, strncpy_from_user
- 상세: [Rust 지원 상세](rust.md)

### 가상화
- **KVM 디바이스 Posted IRQ 대규모 개편** — IOMMU 통합 포함
- **vhost-net VIRTIO_F_IN_ORDER** 지원
- **virtio GSO over UDP 터널** 지원
- **vsock SIOCINQ ioctl** 도입
- **KVM irqfd 전역 고유 등록**
- 상세: [가상화 상세](virtualization.md)

### 블록 레이어
- **FALLOC_FL_WRITE_ZEROES** — NVMe/SCSI 디바이스에서 대역폭 소비 없이 효율적 제로 쓰기
- **WBT 최적화** 및 문서 업데이트
- **ublk**: off-daemon zero-copy 버퍼 등록, 서버 종료 처리 속도 향상
- **UFS HID 지원**, MediaTek UFS 하드웨어 버전/FDE/Vcore 스케일링
- **pktcdvd 드라이버 제거** — 2016년 deprecated, 최종 삭제
- 상세: [블록 레이어 상세](block-layer.md)

### 아키텍처
- **ARM64**: 커널 라이브 패칭, GICv5 인터럽트 컨트롤러, Branch Record Buffer Extension (BRBE), kcfi + BPF, feat_mte_store_only
- **x86**: 공격 벡터 제어(AVC), Intel Wildcat Lake/Bartlett Lake-S, AMD Hardware Feedback Interface (HFI)
- **RISC-V**: 변경사항 거부됨 — Linus Torvalds가 "garbage" pull request로 지적, 6.18으로 이월
- **LoongArch**: BPF 지원 강화 (동적 코드 수정, trampolines, struct ops)
- **S390**: 투명 대형 페이지 스왑/마이그레이션 지원
- 상세: [아키텍처 상세](architecture.md)

### 드라이버
- **Intel Xe3 그래픽** — Panther Lake용 기본 활성화
- **AMD SmartMux** — 하이브리드 GPU 노트북 자동 전환
- **Apple M1/M2 SMC** — 메인라인 커널로 재부팅 가능
- **Raspberry Pi 5 RP1** — I/O 칩 드라이버 + pinctrl/clk 통합
- **Qualcomm Iris 디코더** — HEVC (H.265) 및 VP9 코덱 초기 지원
- **Intel IPU7** — Lunar Lake/Panther Lake 웹캠 지원
- **Samsung Exynos 2200** — Galaxy S22 칩셋 지원
- **NVIDIA Tegra264** — Thor 칩 Jetson T5000 모듈
- 상세: [드라이버 상세](drivers.md)

### 시스콜
- **file_getattr() / file_setattr()** — openat(2) 시맨틱의 확장 파일 속성 시스콜
- **FALLOC_FL_WRITE_ZEROES** — fallocate(2) 새 플래그
- **scm_pidfd** — 수확된(reaped) pidfd 수신
- **open_by_handle_at()** — 파일 핸들 기반 순수 열기
- **syscall_user_dispatch**: PR_SYS_DISPATCH_INCLUSIVE_ON 지원

---

## 이전 버전 대비 주요 변화

- 6.16의 Ext4 대형 folio/atomic write에 이어 6.17에서 Ext4 블록 할당 확장성 및 uncached buffered I/O로 I/O 성능 최적화 지속
- 6.16의 FUSE 대형 folio 지원에 이어 6.17에서 FUSE iomap 기반 버퍼 쓰기로 I/O 경로 현대화
- 6.16의 SELinux 캐싱/와일드카드에 이어 6.17에서 코어 덤프 소켓 확장 및 공격 벡터 제어(AVC)로 보안 체계 강화
- 6.16의 sched_ext 계층적 구조에 이어 6.17에서 SMP 무조건 컴파일 및 프록시 실행으로 스케줄러 기반 혁신
- 6.16의 DMABUF TCP TX zero-copy에 이어 6.17에서 DualPI2 혼잡 제어로 네트워킹 최적화 방향 전환 (대역폭 → 지연)
- Rust: 6.16의 cpufreq/OPP/DRM에 이어 6.17에서 regulator/firmware properties/I/O 리소스로 하드웨어 추상화 계속 확대
- 6.16의 KVM TDX 초기화에 이어 6.17에서 KVM Posted IRQ 대규모 개편으로 가상화 I/O 성능 강화
- ARM64: 6.16의 lazy preemption에 이어 6.17에서 라이브 패칭 및 GICv5로 엔터프라이즈급 기능 확대

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
