# Linux 6.17 파일시스템 변경사항

> 출처: kernelnewbies.org, LWN.net, LKML
> 릴리스: Linux 6.17
> 작성일: 2026-02-10

---

## 개요

Linux 6.17에서 파일시스템 영역은 Btrfs의 실험적 대형 데이터 folio 지원과 free space tree 20% 최적화, Ext4의 블록 할당 확장성 개선 및 uncached buffered I/O, NFS의 쓰기 위임 및 비동기 COPY 재활성화, FUSE의 iomap 기반 쓰기 경로 현대화, EROFS 메타데이터 압축이 핵심이다.

---

## Btrfs

### 변경사항
- **실험적 대형 데이터 folio 지원** — CONFIG_BTRFS_EXPERIMENTAL 조건으로 대형 데이터 folio 읽기/쓰기 활성화. 대용량 파일 처리 시 메모리 오버헤드 감소 (kernelnewbies.org)
- **Free space tree 최적화** — 빈 파일 생성 시 런타임 20% 개선 (kernelnewbies.org)
- **Xarray 인덱싱** — extent buffer에 xarray 기반 인덱싱 도입으로 더 조밀한 패킹 구현
- **압축 데이터 readahead 개선** — 압축된 데이터에 대한 미리읽기 성능 향상
- **Defrag ioctl 개선** — no-compression 플래그 추가로 defragmentation 시 압축 제어 강화
- **Accessor 속도 향상** — 다수의 accessor 함수 최적화 커밋

### 배경 및 영향
- 대형 데이터 folio는 6.16에서 defrag 전용이었으나, 6.17에서 읽기/쓰기 경로로 확대. 다만 실험적 설정이 필요하여 프로덕션 적용은 6.18 이후 전망
- Free space tree 20% 최적화는 컨테이너/빌드 서버처럼 다수의 소형 파일을 생성하는 워크로드에서 효과적
- Xarray 전환은 커널 전반의 radix tree → xarray 마이그레이션 트렌드의 일환

---

## Ext4

### 변경사항
- **블록 할당 확장성 개선** — I/O 집약적 작업에서 눈에 띄는 성능 향상 (kernelnewbies.org, Phoronix)
- **Uncached buffered I/O 지원** — 버퍼 캐시를 우회하는 buffered I/O 모드. 대용량 순차 쓰기 워크로드에서 캐시 오염 방지
- **FALLOC_FL_WRITE_ZEROES 지원** — NVMe SSD/SCSI 디바이스에서 실제 I/O 없이 효율적으로 제로 쓰기. 디바이스의 UNMAP/DEAC 명령 활용
- **i_version 옵션 보존** — 파일시스템 리마운트 시 i_version 옵션 유지

### 배경 및 영향
- 블록 할당 확장성 개선은 6.16의 대형 folio 37% 향상에 이어 Ext4 I/O 성능 최적화가 지속되는 흐름
- Uncached buffered I/O는 데이터베이스 로그, 백업, 대용량 파일 복사 시나리오에서 페이지 캐시 오염 없이 높은 처리량 달성
- FALLOC_FL_WRITE_ZEROES는 플래시 스토리지의 마모를 줄이면서 사전 할당 파일의 효율적 생성 가능

---

## NFS

### 변경사항
- **NFSD 쓰기 위임(write delegation)** — OPEN 시 SHARE_ACCESS에 대한 쓰기 위임 지원. 클라이언트가 파일에 대한 배타적 쓰기 권한 획득 시 서버 왕복 감소
- **max-ops-per-compound 제한 제거** — 복합 연산당 최대 연산 수 제한 해제로 대규모 배치 작업 효율 향상
- **비동기 NFSv4.2 COPY 재활성화** — 서버 간 비동기 파일 복사 기능 복원
- **클라이언트 btime 지원** — 파일 생성 시간(birth time) 지원
- **커널 키링 생성 및 TLS 키링** — NFS 보안 통신을 위한 키링 인프라 강화

### 배경 및 영향
- 쓰기 위임은 NFS 성능의 핵심 최적화로, 빈번한 서버 왕복을 줄여 네트워크 파일시스템 성능 향상
- max-ops-per-compound 제한 제거는 대규모 메타데이터 연산(디렉터리 열거, 속성 배치 조회)에서 효과적
- TLS 키링은 NFS-over-TLS 보안 구현의 일환

---

## F2FS

### 변경사항
- **새 mount API 전환** — 최신 커널 mount API로 마이그레이션
- **복구 통계 계정** — 복구 과정의 통계 추적 추가
- **GC 부스트 sysfs 튜닝 노드** — gc_boost_gc_greedy, gc_boost_gc_multiple, boost_zoned_gc_percent 파라미터
- **예약된 핀 섹션 sysfs 항목** — reserved_pin_section 노출

### 배경 및 영향
- 새 mount API 전환은 커널 전반의 VFS 현대화 트렌드에 따른 것으로, Android에서 F2FS를 기본 파일시스템으로 사용하는 점에서 중요
- GC 부스트 파라미터는 Android 기기의 스토리지 성능 유지에 활용 가능. 특히 zoned 스토리지 환경에서의 가비지 컬렉션 튜닝

---

## FUSE

### 변경사항
- **iomap 기반 버퍼 쓰기(buffered writes)** — 기존 address_space_operations에서 iomap 기반 쓰기로 전환, writeback 포함
- iomap 경로를 통한 현대적 I/O 처리로 성능 및 유지보수성 향상

### 배경 및 영향
- 6.16에서 FUSE에 대형 folio 지원(10 커밋)을 추가한 데 이어, 6.17에서 iomap 기반 쓰기로 I/O 경로 전체를 현대화
- iomap은 XFS, Ext4 등 주요 파일시스템이 사용하는 I/O 프레임워크로, FUSE의 채택은 유저스페이스 파일시스템의 성능 기반 강화
- Android의 FUSE 기반 /sdcard 스토리지에 장기적 성능 영향 가능

---

## EROFS

### 변경사항
- **메타데이터 압축 지원** — 파일시스템 메타데이터의 압축으로 이미지 크기 감소 (kernelnewbies.org)
- **디렉터리 readahead** — 디렉터리 읽기 성능 향상을 위한 미리읽기 지원

### 배경 및 영향
- EROFS는 읽기 전용 파일시스템으로 Android의 system 파티션(EROFS 채택 확대 중)에 직접 관련
- 메타데이터 압축은 OTA 업데이트 크기 및 스토리지 사용량 감소에 기여
- 디렉터리 readahead는 앱 설치 목록, 시스템 부팅 시 파일 탐색 속도 개선

---

## OverlayFS

### 변경사항
- **Case-folding 파일시스템 레이어 지원** — 대소문자 무관 파일명 매칭을 OverlayFS 레이어에서 지원

### 배경 및 영향
- Case-folding은 Windows 호환 파일명 처리에 중요. Ext4/F2FS의 case-folding 기능을 OverlayFS에서도 활용 가능
- Android의 앱 격리, APEX/APK 오버레이 시나리오에서 대소문자 구분 없는 파일 접근에 활용 가능성

---

## SMB/CIFS

### 변경사항
- **SMB1 reparse point 생성** — SMB1 프로토콜에서 reparse point 생성 지원

### 배경 및 영향
- 레거시 SMB1 환경과의 호환성 유지를 위한 변경. 보안상 SMB1 사용은 권장되지 않음

---

## 이전 버전과의 연속성

- **Btrfs folio 전환**: 6.15 extent_io.c folio 준비 → 6.16 defrag 대형 folio → 6.17 실험적 대형 데이터 folio (읽기/쓰기)
- **Ext4 성능 최적화**: 6.16 대형 folio (37% 향상) → 6.17 블록 할당 확장성 + uncached buffered I/O
- **FUSE 현대화**: 6.16 대형 folio (10 커밋) → 6.17 iomap 기반 쓰기
- **EROFS 확장**: 6.16 Intel QAT 하드웨어 가속 → 6.17 메타데이터 압축 + 디렉터리 readahead
- **F2FS**: 6.16 (주요 변경 없음) → 6.17 mount API 전환 + GC 튜닝

---

## 참고 자료
- [kernelnewbies.org Linux 6.17](https://kernelnewbies.org/Linux_6.17)
- [LWN.net 6.17 merge window part 1](https://lwn.net/Articles/1031713/)
- [LWN.net 6.17 merge window part 2](https://lwn.net/Articles/1032095/)
- [Phoronix Linux 6.17 Features](https://www.phoronix.com/news/Linux-6.17-Features-Reminder)
