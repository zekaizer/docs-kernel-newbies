# Linux 6.15 커널 변경사항 개요

> 출처: kernelnewbies.org, LWN.net
> 릴리스 날짜: 2025-05-15
> 작성일: 2026-02-10

---

## 주요 기능 (Headline Features)

- **VFS 마운트 네임스페이스 확장** — `open_tree_attr()` 시스콜 추가로 기존 idmapped 마운트에서 새로운 idmapped 마운트 생성 가능; 분리된 마운트를 먼저 attach하지 않고 조립할 수 있어 private rootfs 구성에 활용
- **fanotify 기반 마운트 알림** — `/proc/<pid>/mountinfo` 폴링 없이 마운트 토폴로지 변경(mount, umount, move)을 감지하는 API 제공
- **AMD INVLPGB TLB 무효화** — Zen 3+ 아키텍처에서 IPI 없이 원격 CPU의 TLB를 브로드캐스트 방식으로 무효화하여 성능 향상
- **io_uring zero-copy 수신** — 커널 메모리 복사 없이 애플리케이션 메모리로 직접 대량 수신 가능
- **Perf 지연시간 프로파일링** — CPU 시간이 아닌 wall-time 기반 지연시간 표시로 실제 실행 병목 식별 가능
- **fwctl 서브시스템** — 펌웨어 인터페이스를 유저스페이스에 표준화된 방식으로 노출 (디버깅, 설정, 프로비저닝)
- **Bcachefs 주요 기능 추가** — 스크러빙, 페이지 크기 초과 블록 크기 지원, case-folding 기능

---

## 서브시스템 요약

### 파일시스템
- **Bcachefs**: 스크러빙 기능, 페이지 크기 초과 블록 크기, case-folding 및 디스크 포맷 변경
- **Btrfs**: send 경로 계산 최적화(~30% 런타임 개선), zstd 음수 압축 레벨, extent_io.c 대형 folio 준비
- **XFS**: copy-on-write 기반 large atomic write, zone GC 임계값 조정, 30건 이상 zoned device 커밋
- **Ext4**: 선형 dentry 검색 최적화, remount-ro 에러 핸들링 개선, 슈퍼블록 업데이트 간격 튜닝
- **F2FS**: Mount API 전환 시작, nat_bits 마운트 옵션화
- 상세: [파일시스템 상세](filesystems.md)

### 메모리 관리
- Per-vma 락 refcounting 기반 재구현
- mseal 시스템 매핑 지원, 파일 기반/shmem guard region 권한 부여
- defrag_mode sysctl로 huge page 유지 관리 강화
- 대형 folio의 batched unmap lazyfree, per-CMA 락으로 동시 할당 성능 향상
- 상세: [메모리 관리 상세](memory-management.md)

### io_uring
- 네트워킹 zero-copy 수신: 커널 메모리 복사 없이 애플리케이션 메모리로 직접 대량 수신
- epoll 이벤트 읽기 지원으로 이벤트 루프의 부분적 completion 모델 전환 가능
- vectored registered buffer 및 커널 bvec 통합으로 ublk/stripe 성능 개선

### 스케줄러
- **sched_ext**: 코어 이벤트 카운터, per-NUMA idle cpumask 분할, `SCX_OPS_ALLOW_QUEUED_WAKEUP` 플래그
- pidfs 확장: `PIDFD_INTO_EXIT`로 프로세스 reaping 이후에도 exit 정보 조회 가능

### 보안
- RV(Runtime Verification) 스케줄러 스펙 모니터 추가
- UBSAN overflow 패턴 제외로 타겟팅된 undefined behavior 감지
- iommufd PASID attach/replace, vIOMMU vEVENTQ 인프라 확장

### Rust 지원
- pin-init 독립 크레이트로 리팩터링
- hrtimer Rust API 지원
- mm_struct, vm_area_struct, mmap 연산 Rust 바인딩 추가

### 아키텍처
- AMD INVLPGB 명령어: Zen 3+ 에서 IPI 없는 브로드캐스트 TLB 무효화

---

## 이전 버전 대비 주요 변화
- 6.14의 FUSE io_uring 통신에 이어 6.15에서는 io_uring zero-copy 수신으로 비동기 I/O 범위 확대
- 6.14의 XFS reflink 작업에 이어 6.15에서 large atomic write + COW 결합
- Per-vma 락이 refcounting 기반으로 재구현되어 확장성 기반 확보
- Bcachefs가 스크러빙, 유연한 블록 크기 등 프로덕션 수준 기능으로 빠르게 성숙

---

## 관련 문서
- [파일시스템 상세](filesystems.md)
- [메모리 관리 상세](memory-management.md)
