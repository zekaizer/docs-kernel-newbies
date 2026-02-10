# Linux 6.15 파일시스템 변경사항

> 출처: kernelnewbies.org, LWN.net, LKML
> 릴리스: Linux 6.15
> 작성일: 2026-02-10

---

## 개요

Linux 6.15에서 파일시스템 영역은 Bcachefs의 프로덕션 수준 기능 추가, Btrfs send 최적화, XFS의 zoned device 대규모 지원이 핵심이다. Ext4와 F2FS는 안정성 및 API 현대화에 집중했다.

---

## Bcachefs

### 변경사항
- **스크러빙 기능 추가** — 디스크 상의 데이터 무결성을 주기적으로 검증하는 scrub 연산 지원
- **페이지 크기 초과 블록 크기** — 4KB 페이지 시스템에서도 4KB 이상의 블록 크기 사용 가능
- **Case-folding** — 대소문자 구분 없는 파일 이름 지원 및 관련 디스크 포맷 수정
- 체크섬 오류 핸들링 강화

### 배경 및 영향
- Bcachefs는 6.15에서 스크러빙과 유연한 블록 크기를 추가하며 프로덕션 환경에서의 신뢰성을 크게 높였다
- Case-folding은 Ext4, F2FS에 이미 존재하던 기능으로, Bcachefs의 기능 패리티를 위한 추가
- 디스크 포맷 변경이 수반되어 이전 버전과의 호환성에 주의 필요

---

## Btrfs

### 변경사항
- **send 경로 계산 최적화** — send 연산 시 경로 계산 로직 개선으로 ~30% 런타임 감소 (kernelnewbies.org)
- **zstd 음수 압축 레벨 지원** — 빠른 압축을 위한 낮은 압축 비율 옵션 제공
- **extent_io.c 대형 folio 지원 준비** — 향후 대형 folio 도입을 위한 코드 기반 정비

### 배경 및 영향
- send 최적화는 대규모 스냅샷 간 차이 전송 시 실질적인 시간 절약 효과
- zstd 음수 레벨은 압축 속도가 중요한 워크로드(예: 실시간 백업)에 유용
- 대형 folio 준비 작업은 커널 전반의 folio 전환 트렌드(6.13~)의 일환

---

## XFS

### 변경사항
- **Copy-on-write 기반 large atomic write** — `RWF_ATOMIC` 플래그와 COW를 결합하여 다중 블록 원자적 쓰기 지원
- **rwf_dontcache 플래그 구현** — 캐시를 우회하는 I/O 경로 제공
- **Zone GC 임계값 조정** — zoned storage의 가비지 컬렉션 트리거 조건 튜닝
- **Zoned device 지원 대규모 확장** — 30건 이상의 커밋으로 zoned block device 지원 강화

### 배경 및 영향
- 6.13에서 시작된 atomic write 지원이 6.15에서 COW와 결합되어 데이터베이스 워크로드에서의 안정성 향상
- Zoned device 지원은 SMR HDD 및 ZNS SSD 환경을 위한 장기 프로젝트로, 6.15에서 대폭 확장

---

## Ext4

### 변경사항
- **선형 dentry 검색 최적화** — 디렉터리 엔트리 검색 성능 향상
- **Remount-ro 에러 핸들링 개선** — 읽기 전용 재마운트 시 오류 처리 강화
- **데이터 write-back 실패 처리 개선** — 쓰기 실패 시 데이터 손실 방지 로직 보강
- **슈퍼블록 업데이트 간격 튜닝** — 슈퍼블록 갱신 빈도를 조절 가능

### 배경 및 영향
- 안정성 및 에러 복구에 집중한 릴리스로, 새로운 기능보다는 기존 기능의 견고성 강화
- 슈퍼블록 업데이트 간격 조정은 고성능 스토리지에서 불필요한 메타데이터 쓰기 감소에 기여

---

## F2FS

### 변경사항
- **Mount API 전환 시작** — 레거시 마운트 인터페이스에서 새로운 커널 mount API로 전환 작업 착수
- **nat_bits 마운트 옵션화** — 기존 빌드 타임 설정이 런타임 마운트 옵션으로 변경
- **Folio 전환 계속** — 기존 page 기반 코드를 folio 기반으로 점진적 전환
- **posix_fadv_noreuse 회수 지원** — POSIX fadvise NOREUSE 힌트에 따른 페이지 회수 구현

### 배경 및 영향
- Mount API 전환은 커널 전체의 VFS 현대화 작업에 맞춘 변경
- nat_bits 옵션화로 운영 중 NAT 비트맵 동작을 유연하게 제어 가능

---

## OverlayFS

### 변경사항
- **override_creds 마운트 옵션 추가** — 자격증명 오버라이드를 마운트 시 제어
- **O_PATH 파일 디스크립터 지원** — 레이어 지정 시 O_PATH fd 사용 가능

### 배경 및 영향
- 컨테이너 환경에서의 보안 및 유연성 개선에 기여

---

## 이전 버전과의 연속성

- **Atomic write**: 6.13에서 XFS/Ext4 Direct I/O로 시작 → 6.15에서 XFS COW 결합으로 확장
- **Folio 전환**: 6.13부터 시작된 커널 전반의 folio 도입이 Btrfs(extent_io.c 준비)와 F2FS에서 계속 진행
- **Bcachefs 성숙**: 6.15에서 스크러빙, 블록 크기 유연성, case-folding 추가로 프로덕션 레벨 접근
- **Zoned device**: XFS의 zoned storage 지원이 6.15에서 30건 이상 커밋으로 대폭 강화

---

## 참고 자료

- [kernelnewbies.org — Linux 6.15](https://kernelnewbies.org/Linux_6.15)
- [LWN.net — Bcachefs scrubbing](https://lwn.net) (확인 필요)
- [LWN.net — XFS atomic writes](https://lwn.net) (확인 필요)
