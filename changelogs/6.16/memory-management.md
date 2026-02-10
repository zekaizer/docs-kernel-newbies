# Linux 6.16 메모리 관리 변경사항

> 출처: kernelnewbies.org, LWN.net, LKML
> 릴리스: Linux 6.16
> 작성일: 2026-02-10

---

## 개요

Linux 6.16 메모리 관리는 NUMA 가중 인터리브 자동 튜닝, memcg의 IRQ/NMI-safe 통계, MGLRU의 SWAPPINESS_ANON_ONLY 모드, PFN 매핑 리라이트가 핵심이다. DAMON은 NUMA 계층형 메모리 자동 튜닝을 추가했고, folio API가 계속 확장되었다.

---

## 가중 인터리브 자동 튜닝 (Weighted Interleave Auto-tuning)

### 변경사항
- **자동 대역폭 기반 NUMA 할당 정책** — 부팅 또는 핫플러그 시 새로운 대역폭 데이터가 이용 가능해지면 노드 가중치를 자동으로 재계산 (kernelnewbies.org)
- 기존의 수동 가중치 설정을 자동화하여 관리 부담 감소

### 배경 및 영향
- 현대 NUMA 시스템은 노드 간 메모리 대역폭이 비대칭적. CXL 메모리 확장 시나리오에서 특히 중요
- 자동 튜닝으로 관리자가 각 노드의 대역폭을 수동으로 측정/설정할 필요 없이 최적의 인터리브 비율 유지
- HBM, CXL 메모리, 다층 NUMA 토폴로지에서 메모리 대역폭 활용 효율 극대화

---

## 메모리 제어 그룹 (Memcg) 개선

### 변경사항
- **IRQ-safe 통계** — 인터럽트 컨텍스트에서 안전하게 memcg 통계 접근 가능
- **비차단 제한 설정** — memcg 제한 설정 시 차단 없는 옵션 추가
- **Memcg/objcg stock 분리** — memcg 스톡과 objcg 스톡을 분리하여 효율 향상
- **멀티-memcg percpu 충전 캐시** — percpu 단위로 다중 memcg 충전 캐시 도입
- **NMI-safe kmem 충전** — NMI 컨텍스트에서도 커널 메모리 충전 가능 (BPF 프로그램 메모리 할당에 핵심)

### 배경 및 영향
- NMI-safe kmem 충전은 BPF 프로그램이 NMI 컨텍스트에서 메모리를 할당할 때 정확한 memcg 회계를 가능하게 함
- 멀티-memcg percpu 캐시는 컨테이너화된 환경에서 다수의 cgroup이 동시에 메모리를 할당하는 시나리오의 경합 감소
- IRQ-safe 통계는 실시간 모니터링 도구의 정확도 향상

---

## MGLRU (Multi-Gen LRU) 및 페이지 회수

### 변경사항
- **SWAPPINESS_ANON_ONLY 모드** — 파일 페이지를 회수 대상에서 제외하고 anonymous 페이지만 스왑하는 모드
- **MADV_DONTNEED/MADV_FREE 배치 TLB 플러시** — 메모리 해제 시 TLB 무효화를 배치로 처리하여 성능 개선
- **memory.reclaim max swappiness 인수** — memory.reclaim 인터페이스에 최대 swappiness 값 지정 가능
- **Swap 해제 코드 다수 개선** — 스왑 해제 경로의 효율성 향상

### 배경 및 영향
- SWAPPINESS_ANON_ONLY는 파일 서버나 데이터베이스처럼 파일 캐시가 중요한 워크로드에서 유용. 파일 캐시 보존을 우선시하면서 메모리 압력을 anonymous 페이지 스왑으로 해소
- 배치 TLB 플러시는 대량의 메모리를 해제하는 시나리오(앱 전환, 컨테이너 종료)에서 성능 향상
- Android의 LMKD와 연동 시 메모리 회수 효율 개선 가능

---

## DAMON (Data Access Monitoring)

### 변경사항
- **NUMA 자동 튜닝** — 계층형 메모리 티어(예: DRAM + CXL) 환경에서 자동 최적화
- DAMON 기반 NUMA 밸런싱으로 데이터 접근 패턴에 따른 메모리 티어 간 페이지 이동 자동화

### 배경 및 영향
- DAMON은 6.13부터 매 릴리스마다 개선되어 온 핵심 메모리 관리 프레임워크
- NUMA 자동 튜닝은 이기종 메모리(HBM, CXL, DDR) 환경에서 핫 데이터를 빠른 메모리로 자동 이동하는 기반
- 6.15의 자동 튜닝 집계 간격에 이어 6.16에서 NUMA 티어 최적화로 확장

---

## Folio API 확장

### 변경사항
- **folio_mk_pte()** — folio 참조로부터 PTE(Page Table Entry) 생성 함수 추가
- **대형 folio 배치 mincore 처리** — mincore 시스콜에서 대형 folio를 배치로 처리하여 효율 향상

### 배경 및 영향
- folio_mk_pte()는 folio 기반 메모리 매핑의 핵심 빌딩 블록. 페이지 테이블 조작 시 folio를 직접 활용하여 코드 경로 단순화
- 커널 전반의 page → folio 전환이 6.16에서도 계속 진행. Ext4, FUSE, Btrfs 등 파일시스템 레벨의 folio 도입과 연계

---

## PFN 매핑 리라이트

### 변경사항
- **vm_pat 필드 제거** — PAT(Page Attribute Table) 기반 가상 매핑 추적에서 vm_pat 필드를 제거하여 구조 단순화
- PFN 매핑 코드 전면 재작성

### 배경 및 영향
- vm_pat 필드 제거는 VMA 구조체의 크기 감소 및 코드 복잡성 감소에 기여
- PAT 관련 처리의 정리는 향후 메모리 매핑 최적화의 기반

---

## 페이지 테이블 및 메모리 압축

### 변경사항
- **커널 페이지 테이블 생성자 필수화** — 모든 아키텍처에서 커널 페이지 테이블에 대해 반드시 생성자를 호출
- **선제적 메모리 압축 강화** — 더 공격적인 proactive compaction 설정 가능

### 배경 및 영향
- 페이지 테이블 생성자 필수화는 아키텍처 간 일관성을 보장하고 특정 메모리 초기화 버그 방지
- 공격적 proactive compaction은 THP 가용성 유지에 기여. 6.15의 defrag_mode에 이어 단편화 방지 수단 강화

---

## Hugetlb 및 VMA (Rust)

### 변경사항
- **Hugetlb 별도 nodemask** — 부트 메모리 할당을 위한 별도의 nodemask 도입
- **Rust VMA 추상화** — mm_struct, vm_area_struct, mmap 연산에 대한 Rust 바인딩 확장 (6.15에서 시작, 6.16에서 계속)

### 배경 및 영향
- Rust VMA 추상화는 Rust 기반 커널 드라이버가 메모리 매핑을 안전하게 조작하기 위한 필수 인프라

---

## 이전 버전과의 연속성

- **Folio 전환**: 6.13 tmpfs → 6.14 확대 → 6.15 lazyfree 최적화 → 6.16 folio_mk_pte(), mincore 배치 처리, Ext4/FUSE/Btrfs 파일시스템 도입
- **DAMON**: 6.14 debugfs 제거 → 6.15 자동 튜닝 집계 간격 → 6.16 NUMA 티어 자동 튜닝
- **메모리 압축**: 6.15 defrag_mode sysctl → 6.16 공격적 proactive compaction 설정
- **MGLRU**: 6.16에서 SWAPPINESS_ANON_ONLY 및 memory.reclaim 확장으로 회수 정책 세분화
- **Memcg**: NMI/IRQ-safe 통계와 멀티-percpu 캐시로 컨테이너 환경의 정확도 및 성능 향상

---

## 참고 자료

- [kernelnewbies.org — Linux 6.16](https://kernelnewbies.org/Linux_6.16)
- [LWN.net — The first half of the 6.16 merge window](https://lwn.net/Articles/1022512/)
- [LWN.net — The second half of the 6.16 merge window](https://lwn.net/Articles/1023075/)
