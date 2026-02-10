# Linux 6.17 메모리 관리 변경사항

> 출처: kernelnewbies.org, LWN.net, LKML
> 릴리스: Linux 6.17
> 작성일: 2026-02-10

---

## 개요

Linux 6.17 메모리 관리는 대형 folio에 대한 mprotect() 3배 이상 속도 향상과 mremap() 37% 실행 시간 단축이 가장 두드러진다. NUMA 노드별 선제적 회수 인터페이스 도입, per-VMA lock의 /proc/pid/maps 읽기 확대, DAMON_STAT 간소화 모니터링, hugetlb folio 예약 방식 개선이 핵심이다.

---

## 대형 Folio mprotect()/mremap() 최적화

### 변경사항
- **mprotect() 대형 folio 처리** — 대형 folio에 대한 mprotect() 호출 시 3배 이상 속도 향상 (kernelnewbies.org)
- **mremap() 대형 folio 처리** — 대형 folio의 mremap() 실행 시간 37% 단축 (kernelnewbies.org)
- **Multi-VMA mremap() 이동** — 여러 VMA에 걸친 mremap() 이동 지원

### 배경 및 영향
- 대형 folio의 확산에 따라 메모리 보호 변경(mprotect)과 메모리 재매핑(mremap) 경로의 최적화가 필수적
- 3배 이상의 mprotect 속도 향상은 JIT 컴파일러(ART, V8), 보안 샌드박스(seccomp), 메모리 맵 파일 처리에서 실질적 성능 개선
- mremap 37% 단축은 컨테이너 메모리 관리, 대용량 메모리 매핑 워크로드에서 효과적

---

## NUMA 노드별 선제적 회수

### 변경사항
- **Per-node proactive reclaim** — memcg 인터페이스를 넘어 NUMA 노드별 유저스페이스 메모리 회수 제어 (kernelnewbies.org)
  - 사용법: `echo 512M swappiness=10 > /sys/devices/system/node/nodeX/reclaim`
- **NUMA 노드 notifier API** — 노드 상태 변경 시 콜백 알림 메커니즘

### 배경 및 영향
- 기존 memory.reclaim은 memcg 단위 제어만 가능했으나, 6.17에서 물리적 NUMA 노드 단위 제어로 확장
- CXL 메모리 확장, 이기종 메모리(HBM + DDR + CXL) 환경에서 노드별 메모리 압력 관리에 핵심
- swappiness 파라미터로 노드별 회수 정책 세분화 가능

---

## Per-VMA Lock 확대

### 변경사항
- **Per-VMA lock /proc/pid/maps 읽기** — /proc/pid/maps 파일 읽기 시 mmap_lock 대신 per-VMA lock 사용으로 경합 감소
- **MADV_DONTNEED per-VMA lock 지원** — madvise DONTNEED 호출에 per-VMA lock 적용
- **Madvise 정리 및 통합** — madvise 코드 구조 정리

### 배경 및 영향
- /proc/pid/maps 읽기는 디버거(gdb, lldb), 메모리 분석 도구, Android의 libmeminfo에서 빈번하게 사용
- 기존 mmap_lock 경합은 대규모 주소 공간을 가진 프로세스에서 심각한 병목. per-VMA lock 전환으로 경합 해소
- 6.15에서 시작된 per-VMA lock 확산이 6.17에서 /proc 인터페이스까지 확대

---

## DAMON 개선

### 변경사항
- **DAMON_STAT** — 간소화된 접근 모니터링 모듈 도입. 복잡한 DAMON 설정 없이 기본적인 메모리 접근 패턴 모니터링 (kernelnewbies.org, LWN.net)
- **DAMON 인터리빙** — migrate 액션에서 인터리빙 지원

### 배경 및 영향
- DAMON_STAT은 CONFIG_DMABUF_SYSFS_STATS와 유사한 간소화 인터페이스로, 메모리 관리 의사결정의 진입 장벽 낮춤
- 6.15 자동 튜닝 집계 → 6.16 NUMA 자동 튜닝 → 6.17 DAMON_STAT + 인터리빙으로 DAMON 기능 지속 확대

---

## Hugetlb 및 대형 페이지

### 변경사항
- **Hugetlb folio 예약 후 할당** — 할당 전 folio 예약으로 메모리 부족 시 안전한 실패 처리
- **Hugetlb faulting 경로 재작업** — misc hugetlb 페이지 폴트 처리 경로 전면 개선
- **Frozen pages for large kmalloc** — 대형 kmalloc 할당에서 frozen pages 활용

### 배경 및 영향
- Hugetlb folio 예약은 대형 페이지 할당의 예측 가능성 향상. 데이터베이스, VM 호스트 환경에서 OOM 위험 감소
- Faulting 경로 재작업은 hugetlb 코드의 구조적 정리로 유지보수성 향상

---

## 페이지 테이블 및 GUP

### 변경사항
- **pXX_devmap 비트 제거** — 디바이스 매핑용 페이지 테이블 비트 제거
- **pfn_t 타입 제거** — 레거시 pfn_t 타입 완전 삭제
- **GUP longterm pin_user_pages() 최적화** — 장기 핀 사용자 페이지 획득 최적화

### 배경 및 영향
- pXX_devmap과 pfn_t 제거는 page → folio 전환의 부수적 정리 작업
- GUP 최적화는 RDMA, GPU DMA, VFIO 등 사용자 페이지를 장기 핀하는 워크로드의 성능 개선

---

## 스왑 및 페이지 회수

### 변경사항
- **shmem/swap mTHP swap-in 버그 수정** — 다중 크기 투명 대형 페이지(mTHP) 스왑 인 관련 수정
- **KSM VMA 병합 방지** — Kernel Samepage Merging 시 VMA 병합 방지로 안정성 향상
- **Tmpfs folio fault 완료** — tmpfs의 folio 기반 페이지 폴트 처리 완성
- **Shmem spinlock 감소** — shmem에서 spinlock 사용 줄임
- **Vmscan 비례 회수** — memcg에서 MGLRU 기반 비례 회수
- **NR_WRITEBACK_TEMP 카운터 제거** — 미사용 카운터 정리

### 배경 및 영향
- mTHP 스왑 인 수정은 메모리 압력 하에서의 안정성 향상. Android의 zram 스왑 환경에 관련
- Vmscan 비례 회수는 컨테이너/cgroup 환경에서 공정한 메모리 회수를 보장

---

## Readahead 및 기타

### 변경사항
- **Readahead 대형 folio 조정** — 더 큰 folio를 위한 readahead 파라미터 튜닝
- **migrate_isolate 독립 비트** — 마이그레이션 격리를 독립 비트로 관리

### 배경 및 영향
- Readahead 대형 folio 조정은 순차 읽기 워크로드의 I/O 효율 향상에 기여. 파일시스템 레벨의 대형 folio 지원(Ext4, Btrfs)과 연계

---

## 이전 버전과의 연속성

- **대형 folio 최적화**: 6.16 Ext4 대형 folio 37% 향상 → 6.17 mprotect 3x/mremap 37% 속도 향상으로 메모리 관리 경로도 folio 최적화
- **Per-VMA lock**: 6.15 도입 → 6.16 확대 → 6.17 /proc/pid/maps + MADV_DONTNEED까지 확대
- **DAMON 진화**: 6.15 자동 튜닝 → 6.16 NUMA 자동 튜닝 → 6.17 DAMON_STAT 간소화 모듈
- **NUMA 메모리 관리**: 6.16 가중 인터리브 자동 튜닝 → 6.17 노드별 선제적 회수
- **Page → Folio 전환**: pXX_devmap/pfn_t 제거, tmpfs folio fault 완료 등 구조적 정리 계속

---

## 참고 자료
- [kernelnewbies.org Linux 6.17](https://kernelnewbies.org/Linux_6.17)
- [LWN.net 6.17 merge window part 1](https://lwn.net/Articles/1031713/)
- [LWN.net 6.17 merge window part 2](https://lwn.net/Articles/1032095/)
