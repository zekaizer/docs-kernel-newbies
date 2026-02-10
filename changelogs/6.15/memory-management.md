# Linux 6.15 메모리 관리 변경사항

> 출처: kernelnewbies.org, LWN.net, LKML
> 릴리스: Linux 6.15
> 작성일: 2026-02-10

---

## 개요

Linux 6.15 메모리 관리는 per-vma 락의 refcounting 기반 재구현, mseal 시스템 매핑 지원, 대형 folio lazyfree 최적화가 핵심이다. DAMON은 자동 튜닝 기능과 필터 개선으로 접근 패턴 기반 메모리 관리를 강화했다.

---

## Per-VMA 락

### 변경사항
- **Refcounting 기반 재구현** — 기존 per-vma 락 메커니즘이 참조 카운팅 방식으로 전면 재작성
- **mseal 시스템 매핑 지원** — `mseal()`로 보호된 매핑에 대한 per-vma 락 통합
- **Guard region 확장** — 파일 기반 및 shmem 매핑에 대한 guard region 권한 부여

### 배경 및 영향
- Per-vma 락은 mmap_lock의 병목을 해소하기 위한 장기 프로젝트로, 6.15에서 refcounting 기반으로 안정적 기반 확보
- 이후 6.17, 6.18에서 `/proc/pid/maps` 읽기에 per-vma 락이 적용되는 등 점진적 확산
- mseal 통합으로 보안 강화된 매핑에서도 성능 이점 유지

---

## 메모리 할당 및 단편화

### 변경사항
- **defrag_mode sysctl** — 페이지 할당자가 "단편화를 회피하고 huge page 생성 능력을 유지하도록 더 노력하는" 새로운 sysctl 추가 (kernelnewbies.org)
- **Buddy 할당자 유사 folio 분할** — folio 분할 로직이 buddy 시스템과 유사한 방식으로 개선
- **rmqueue_bulk() 폴백 가속** — 벌크 페이지 할당 시 대체 경로 성능 향상

### 배경 및 영향
- defrag_mode는 THP(Transparent Huge Pages) 워크로드에서 장기 실행 시 발생하는 단편화 문제를 사전에 완화
- 서버 워크로드에서 huge page 가용성 유지에 기여

---

## 페이지 회수 및 Lazyfree

### 변경사항
- **대형 folio의 batched unmap lazyfree** — MADV_FREE로 마킹된 대형 folio를 배치 방식으로 unmap하여 회수 효율 향상
- **Proactive 메모리 회수 통계 추가** — 사전 회수 동작의 모니터링을 위한 통계 인터페이스 제공
- **Swap slot cache 제거** — 기존 swap slot 캐시가 제거되어 코드 단순화
- **Per-CMA 락** — CMA(Contiguous Memory Allocator) 영역별 개별 락으로 동시 할당 성능 향상

### 배경 및 영향
- Batched lazyfree는 대형 folio가 확산됨에 따라 회수 경로의 성능 병목을 해소
- Per-CMA 락은 다수의 CMA 영역을 사용하는 시스템(예: GPU 메모리 할당)에서 경합 감소

---

## DAMON (Data Access Monitoring)

### 변경사항
- **자동 튜닝 집계 간격** — 모니터링 집계 인터벌이 워크로드에 맞게 자동 조정
- **필터 개선** — reject-then-allow 방식 필터, unmapped/active 페이지 필터 타입 추가
- **hugepage_size 필터 지원** — 특정 크기의 huge page를 대상으로 한 필터링 가능

### 배경 및 영향
- DAMON은 6.13부터 매 릴리스마다 개선되는 주요 메모리 관리 프레임워크
- 자동 튜닝으로 관리자 개입 없이도 최적의 모니터링 주기 유지 가능

---

## 대형 Folio 및 메모리 소유권

### 변경사항
- **대형 비-hugetlb folio에 대한 mm owner 추적 확장** — 기존 hugetlb에만 적용되던 소유권 추적이 일반 대형 folio로 확장
- **CONFIG_NO_PAGE_MAPCOUNT 지원** — 특수 배포 환경을 위한 page mapcount 비활성화 옵션

### 배경 및 영향
- 커널 전반의 folio 전환 트렌드(6.13~)에서 메모리 추적 인프라도 folio 기반으로 전환되는 과정

---

## madvise 및 매핑 보호

### 변경사항
- **madvise mmap 락 분리** — madvise 연산의 mmap_lock 의존성을 분리하여 동시성 향상
- **Guard region 확장** — 파일 기반 및 shmem 매핑에 대한 guard region 권한 부여

### 배경 및 영향
- 6.13에서 도입된 MADV_GUARD_INSTALL 경량 guard page의 연장선으로, 보호 범위를 확대

---

## 이전 버전과의 연속성

- **Per-VMA 락**: 6.15에서 refcounting 재구현 → 6.17에서 `/proc/pid/maps` 적용 → 6.18에서 더 넓은 범위로 확산
- **대형 folio**: 6.13 tmpfs 지원 → 6.14 folio 확대 → 6.15 lazyfree 최적화 및 소유권 추적 확장
- **DAMON**: 매 릴리스마다 필터 및 자동화 기능 추가 (6.14 debugfs 제거, 6.15 자동 튜닝)
- **Guard page**: 6.13 MADV_GUARD_INSTALL 도입 → 6.15 파일 기반/shmem 확장

---

## 참고 자료

- [kernelnewbies.org — Linux 6.15](https://kernelnewbies.org/Linux_6.15)
- [LWN.net — Per-VMA locking](https://lwn.net) (확인 필요)
- [LWN.net — DAMON auto-tuning](https://lwn.net) (확인 필요)
