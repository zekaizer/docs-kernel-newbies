# Linux 6.18 블록 레이어 변경사항

> 출처: kernelnewbies.org, Phoronix
> 릴리스: Linux 6.18
> 작성일: 2026-02-10

---

## 개요

Linux 6.18 블록 레이어는 dm-pcache(영속 메모리 캐시) 도입, md/llbitmap(락프리 비트맵), 블록 무결성 P2P 지원이 핵심이다. dm-pcache는 CXL 영속 메모리 활용의 실질적 사용 사례를 제공한다.

---

## dm-pcache (Persistent Memory Cache)

### 변경사항
- **Device Mapper 영속 메모리 캐시 타겟** — CXL 영속 메모리(persistent memory) 및 DAX 디바이스를 SSD/HDD 위의 고성능 캐시 계층으로 활용하는 새 device mapper 타겟
- **고처리량/저지연 직접 접근** — DAX 모드를 통한 직접 접근으로 기존 블록 I/O 경로 우회
- **읽기/쓰기 캐시** — 읽기와 쓰기 모두 캐시 가능

### 배경 및 영향
- CXL(Compute Express Link) 기반 영속 메모리가 상용화됨에 따라, 이를 캐시로 활용하는 실질적 인프라 필요성 증가
- 기존 dm-cache/dm-writecache는 DRAM 또는 SSD를 캐시로 사용했으나, dm-pcache는 영속 메모리의 특성(바이트 단위 접근, 비휘발성)을 직접 활용
- 서버/데이터센터 환경에서 DRAM과 SSD 사이의 성능 계층으로 CXL 메모리 활용 가능
- DAX 모드의 직접 접근은 기존 블록 I/O 스택의 소프트웨어 오버헤드 제거

---

## md/llbitmap (Lockless Bitmap)

### 변경사항
- **락프리 비트맵 메커니즘** — MD RAID에서 기존 비트맵 대신 락프리 비트맵 구현 도입
- **CONFIG_MD_BITMAP** — 새 Kconfig 옵션으로 비트맵 기능 선택 가능

### 배경 및 영향
- MD RAID의 비트맵은 쓰기 의도 추적에 사용되며, 동기화 중 어떤 영역을 재동기화해야 하는지 결정
- 기존 비트맵은 락 기반으로 고성능 NVMe RAID 환경에서 병목 발생 가능
- 락프리 구현은 다수의 디스크에서 동시 쓰기 시 경합 감소
- NVMe RAID 구성에서의 성능 향상 기대

---

## rnull 개선

### 변경사항
- **configfs 지원** — rnull(Rust null 블록 드라이버)에 configfs 인터페이스 추가
- **원격 완료** — 원격 완료(remote completion) 기능 추가

### 배경 및 영향
- rnull은 Rust로 작성된 null 블록 드라이버로, Rust 블록 레이어 드라이버의 레퍼런스 구현
- configfs 지원으로 런타임 구성 가능
- Rust 블록 레이어 추상화의 발전을 보여주는 사례

---

## dm-integrity

### 변경사항
- **동기식 해시 인터페이스 우선 사용** — 비동기 해시 대신 동기식 해시 인터페이스를 우선 선택

### 배경 및 영향
- dm-integrity는 블록 디바이스 데이터 무결성 보호를 위한 device mapper 타겟
- 동기식 해시는 코드 경로를 단순화하고 대부분의 환경에서 충분한 성능 제공
- 비동기 해시의 복잡성(콜백, 에러 처리)을 피하여 안정성 향상

---

## NBD (Network Block Device)

### 변경사항
- **소켓 제한** — NBD 소켓을 TCP와 UDP로 제한. 기타 소켓 타입 사용 불가

### 배경 및 영향
- 보안 강화를 위한 입력 검증. 예기치 않은 소켓 타입 사용에 의한 문제 방지

---

## 블록 무결성

### 변경사항
- **P2P 소스/목적지 활성화** — 블록 무결성 검사에서 P2P(Peer-to-Peer) 소스 및 목적지 지원

### 배경 및 영향
- NVMe P2P 통신 환경에서 데이터 무결성 검증을 지원
- GPU-direct storage 등 P2P 데이터 경로에서의 무결성 보장

---

## Zpool 레이어 제거

### 변경사항
- **Zpool 간접 레이어 제거** — zswap에서 사용하던 zpool 추상화 계층 제거

### 배경 및 영향
- zpool은 zsmalloc/zbud/z3fold 등 압축 풀을 통합하는 간접 계층이었으나, 실제로는 불필요한 복잡성 추가
- 직접 구현 호출로 전환하여 코드 단순화 및 미세한 성능 향상

---

## 이전 버전과의 연속성

- **Device Mapper**: 6.16 dm-crypt 래핑 인라인 암호화 키 → 6.18 dm-pcache. DM 생태계에 새로운 캐시 타겟 추가
- **RAID**: 6.16의 바운스 버퍼 완전 제거에 이어 6.18 md/llbitmap으로 RAID 성능 인프라 현대화
- **Rust 블록 드라이버**: rnull의 configfs 지원은 Rust 블록 레이어 추상화의 점진적 발전
- **CXL 메모리**: dm-pcache는 CXL 영속 메모리 생태계의 핵심 사용 사례 구현

---

## 참고 자료

- [kernelnewbies.org — Linux 6.18](https://kernelnewbies.org/Linux_6.18)
- [Phoronix — Linux 6.18 Features](https://www.phoronix.com/news/LInux-6.18-Features-Reminder)
