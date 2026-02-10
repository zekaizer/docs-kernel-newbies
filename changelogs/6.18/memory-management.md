# Linux 6.18 메모리 관리 변경사항

> 출처: kernelnewbies.org, LWN.net, Phoronix
> 릴리스: Linux 6.18
> 작성일: 2026-02-10

---

## 개요

Linux 6.18 메모리 관리는 Slub Sheaves(per-CPU 할당 캐시), Swap Table Phase I(5-20% 처리량 향상), memdesc_flags_t(struct page/slab/folio 분리 기반)가 핵심이다. DAMON은 ARM32 LPAE와 stat-purpose 필터를 추가했고, VMA 락킹이 /proc/pid/maps까지 확장되었다. 스왑 클러스터 스캐닝 개선으로 대형 할당 성능이 크게 향상되었다.

---

## Slub Sheaves

### 변경사항
- **per-CPU 캐시 메커니즘** — SLUB 할당자에 sheaves라 불리는 per-CPU 캐시 레이어 도입. 각 CPU마다 두 개의 sheave(해제된 객체 배열)를 유지
- **kfree_rcu() 배칭** — RCU 기반 메모리 해제 시 sheaves를 활용한 배치 처리로 효율 향상
- **재진입 가능 kmalloc_nolock** — 제한된 컨텍스트에서도 안전한 메모리 할당 가능

### 배경 및 영향
- 기존 SLUB 할당자는 해제된 객체를 folio 내에서 추적하는 복잡한 메커니즘 사용. Sheaves는 이를 단순한 per-CPU 배열로 대체하여 해제 비용 대폭 감소
- CPU 간 동기화 오버헤드가 줄어들어 다중 CPU 시스템에서 할당/해제 경합 감소
- NUMA 지역성 관리가 단순화되지만, 크로스 노드 할당이 증가할 수 있는 트레이드오프 존재
- 현재 maple trees에 대해 opt-in으로 적용. 향후 확대 적용 예상
- LWN.net에 의하면 "초기 결과가 유망하나 slab 디버깅 통합 개선 필요"

---

## Swap Table 인프라 (Phase I)

### 변경사항
- **스왑 테이블 캐시 백엔드** — 스왑 테이블을 스왑 캐시의 백엔드로 활용하는 예비 코드 도입
- **처리량 5-20% 향상** — 스왑 관련 워크로드에서 처리량, RPS(Requests Per Second), 빌드 시간 5-20% 개선 (kernelnewbies.org)
- **Tencent 벤치마크** — 96 작업 커널 빌드, 10GB zRAM, 64KB mTHP 환경에서 시스템 시간 약 50% 감소, 스왑 실패율 감소 (Phoronix)

### 배경 및 영향
- 기존 스왑 캐시는 radix tree/XArray 기반으로 스왑 집약적 워크로드에서 병목 발생
- 스왑 테이블은 더 효율적인 데이터 구조로 캐시 조회/삽입 비용 감소
- Phase I은 인프라 기반 작업으로, 후속 릴리스에서 추가 최적화 예상
- Android의 zram 기반 스왑에서 특히 큰 효과 기대

---

## 스왑 클러스터 스캐닝

### 변경사항
- **개선된 클러스터 스캔 전략** — 스왑 클러스터 스캐닝 전략 개선으로 단편화 감소 및 대형 할당 성능 향상

### 배경 및 영향
- mTHP(multi-size THP) 기반 대형 페이지 스왑 할당의 실패율 감소
- 6.16의 MGLRU/스왑 해제 개선에 이어 스왑 서브시스템 전반의 최적화 지속

---

## memdesc_flags_t

### 변경사항
- **새 플래그 타입 도입** — struct page, struct slab, struct folio를 위한 통합 플래그 타입 memdesc_flags_t 도입
- **전체 구조체 통합** — 관련 구조체 전반에 걸쳐 memdesc_flags_t 적용

### 배경 및 영향
- struct page의 크기와 복잡성은 오래된 커널 과제. page/slab/folio를 분리하기 위한 장기 프로젝트의 핵심 진전
- memdesc_flags_t는 향후 struct page를 완전히 분해하기 위한 타입 안전성 기반 제공
- 이 작업은 메모리 사용량 감소 및 코드 유지보수성 향상으로 이어질 것으로 기대

---

## DAMON (Data Access Monitoring)

### 변경사항
- **ARM32 LPAE 지원** — ARM32 Large Physical Address Extension 환경에서 DAMON 사용 가능
- **addr_unit 지원 (DAMON_RECLAIM)** — DAMON 기반 회수에서 주소 단위 설정 지원
- **Stat-purpose DAMOS 필터** — 통계 목적의 DAMOS(DAMON-based Operation Schemes) 필터 추가
- **Auto-tuned interval 노출** — 자동 튜닝된 모니터링 간격을 사용자 공간에 노출
- **기타 수정 및 개선** — 다수의 버그 수정 및 소규모 개선

### 배경 및 영향
- ARM32 LPAE 지원은 구형 32비트 ARM SoC에서도 DAMON 활용 가능. 임베디드/IoT 디바이스에 유용
- 6.16의 NUMA 자동 튜닝에 이어 6.18에서 stat 필터 + addr_unit으로 모니터링 세분화 확대

---

## VMA 락킹

### 변경사항
- **/proc/pid/maps 읽기에 per-vma lock 적용** — 프로세스 메모리 맵 조회 시 per-VMA 락 사용으로 mmap_lock 경합 감소

### 배경 및 영향
- /proc/pid/maps 읽기는 프로파일링, 디버깅, 보안 모니터링에서 빈번히 사용
- 기존에는 mmap_lock 읽기 락이 필요하여 다른 VMA 연산과 경합 발생
- per-vma lock은 6.15~6.16에서 페이지 폴트 경로에 적용된 후 6.18에서 /proc 경로까지 확장

---

## THP (Transparent Huge Pages) 구성

### 변경사항
- **PR_SET_THP_DISABLE 확장** — 개별 프로세스가 시스템 전체 THP 설정에 영향 없이 "madvise" 모드로 전환 가능
- **Khugepaged anonymous collapse 범위 확대** — khugepaged의 anonymous 페이지 통합 범위 확장

### 배경 및 영향
- 프로세스별 THP 제어는 혼합 워크로드 환경에서 유용. 특정 앱만 THP를 비활성화하고 싶을 때 시스템 전체 설정 변경 불필요
- khugepaged 확장은 더 많은 anonymous 페이지를 huge page로 통합하여 TLB 미스 감소

---

## OOM 개선

### 변경사항
- **피해자 프로세스 해동(thawing)** — OOM 킬러가 선택한 프로세스의 frozen 상태 해제
- **reaper 순회 순서 개선** — OOM reaper의 메모리 회수 순회 효율 향상

### 배경 및 영향
- frozen 프로세스의 해동은 suspend/resume 중 OOM 상황에서의 회복력 향상
- Android에서 프로세스 동결(freeze)은 일반적이므로, 이 개선은 메모리 부족 상황의 안정성 향상에 기여

---

## Folio 관련 개선

### 변경사항
- **커널 파일 매핑 folio** — Btrfs 등에서 사용하는 커널 파일 매핑 folio 개념 도입
- **연속 페이지 클리어링 최적화** — contiguous page clearing 성능 향상
- **filemap_map_pages folio refcount 최적화** — 페이지 매핑 시 folio 참조 카운트 업데이트 효율화
- **Persistent huge zero folio** — 지속적인 huge zero folio 지원
- **Hugetlb folio 할당 정리** — hugetlb folio 할당 코드 정리

### 배경 및 영향
- 커널 파일 매핑 folio는 기존 page cache folio와 구분되는 커널 내부용 folio 개념
- 6.16의 folio_mk_pte()/mincore 배치 처리에 이어 folio API 확장 지속

---

## 기타 메모리 관리 변경

### Mlock
- **대형 folio 추적 개선** — mlock된 대형 folio의 추적 정확도 향상

### Mincore
- **최적화** — mincore 시스콜 처리 속도 향상

### Readahead
- **mmap_miss 휴리스틱 개선** — 동시 페이지 폴트에서의 mmap_miss 판단 정확도 향상

### Userfaultfd
- **기회적 TLB 플러시 배칭** — present 페이지에 대한 TLB 플러시를 기회적으로 배치 처리

### MGLRU
- **proactive reclaim 통계 최적화** — memcg용 능동적 회수 통계 효율 향상

### Memcg
- **사용자 공간 복귀 최적화** — memcg exit-to-userspace 경로 효율화

### MM 플래그
- **64비트 비트맵 전환** — 모든 아키텍처에서 MM 플래그를 64비트 비트맵으로 통일

### DMA 매핑
- **물리 주소 기반 API 이전** — DMA 매핑 인터페이스를 물리 주소 기반으로 전환

### 할당 추적
- **/proc/allocinfo 부정확 카운터 표시** — 부정확한 할당 카운터를 명시적으로 표시

### KASAN
- **hw-tags write_only 옵션** — 하드웨어 태그 모드에서 쓰기만 감시하는 옵션 추가

### KHO (Kexec Handover)
- **vmalloc 할당 보존** — kexec 핸드오버 시 vmalloc 할당 보존 지원

### Hugepage 예약
- **커맨드라인 설정 확대** — 부트 커맨드라인으로 hugepage 예약량 확대 가능

### HMM
- **PMD 스왑 엔트리에서 PFN 채움** — PMD 수준 스왑 엔트리에서 PFN 세트 채우기 지원

### Rust 할당자
- **대형 정렬 및 NUMA 노드 지원** — Rust 커널 코드에서 정렬 요구사항과 NUMA 노드 지정이 가능한 할당자

---

## 이전 버전과의 연속성

- **Folio 전환**: 6.15 lazyfree → 6.16 folio_mk_pte()/Ext4/FUSE/Btrfs 대형 folio → 6.18 kernel file mapped folios, persistent huge zero folio, filemap 최적화
- **DAMON**: 6.16 NUMA 자동 튜닝 → 6.18 ARM32 LPAE, stat-purpose 필터, addr_unit. 모니터링 범위 및 세밀도 확대
- **VMA 락킹**: 6.15~6.16 페이지 폴트 per-vma lock → 6.18 /proc/pid/maps per-vma lock. 적용 범위 지속 확대
- **스왑 최적화**: 6.16 MGLRU SWAPPINESS_ANON_ONLY, zram 파라미터 → 6.18 Swap Table Phase I (5-20% 향상), 클러스터 스캐닝 개선. 근본적 인프라 개선
- **struct page 분리**: memdesc_flags_t 도입으로 page/slab/folio 분리 프로젝트의 구체적 진전 시작
- **메모리 할당**: Slub sheaves 도입은 SLUB 할당자의 가장 큰 구조적 변화. 향후 전체 slab 할당에 확대 적용 가능

---

## 참고 자료

- [kernelnewbies.org — Linux 6.18](https://kernelnewbies.org/Linux_6.18)
- [LWN.net — Slab allocator: sheaves and any-context allocations](https://lwn.net/Articles/1016001/)
- [Phoronix — The Many Memory Management Improvements In Linux 6.18](https://www.phoronix.com/news/Linux-6.18-MM)
- [Phoronix — Linux 6.18 Features](https://www.phoronix.com/news/LInux-6.18-Features-Reminder)
