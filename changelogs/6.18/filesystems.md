# Linux 6.18 파일시스템 변경사항

> 출처: kernelnewbies.org, LWN.net, Phoronix
> 릴리스: Linux 6.18
> 작성일: 2026-02-10

---

## 개요

Linux 6.18 파일시스템 영역에서 가장 큰 사건은 Bcachefs의 메인라인 제거이다. Btrfs는 block size > page size 준비와 체크섬 검색 최적화, XFS는 온라인 fsck 기본 활성화, FUSE는 대용량 복사 및 동기식 초기화를 도입했다. NFS는 서버 확장성 개선과 RWF_DONTCACHE 지원을 추가했다.

---

## Bcachefs (제거)

### 변경사항
- **메인라인에서 완전 제거** — Linux 6.7에서 도입된 Bcachefs 파일시스템 코드가 커널 소스에서 제거됨. 외부 DKMS 모듈로 유지보수 전환

### 배경 및 영향
- Bcachefs 개발자(Kent Overstreet)와 커널 커뮤니티 간의 코드 품질 및 유지보수 프로세스 관련 갈등이 제거의 주요 원인
- 6.15~6.16에서 활발하게 개선되던(스크러빙, 비상 읽기 전용, 단일 디바이스 모드 등) 파일시스템이 불과 2 릴리스 만에 제거
- 기존 Bcachefs 사용자는 외부 모듈을 통해 계속 사용 가능하나, 메인라인 커널 빌드에서는 더 이상 포함되지 않음
- 향후 커뮤니티 프로세스 준수 시 재도입 가능성은 열려 있음

---

## Btrfs

### 변경사항
- **체크섬 검색 락 경합 감소** — commit root에서 데이터 체크섬을 검색하여 락 경합을 크게 줄임. 일부 워크로드에서 동기화 시간이 분 단위에서 초 단위로 단축 (Phoronix)
- **block size > page size 준비** — 압축 코드에서 블록 크기가 페이지 크기를 초과하는 환경을 위한 준비 작업
- **ref_tracker for delayed_nodes** — 지연 노드에 참조 추적기 적용으로 디버깅 용이성 향상
- **per-filesystem 압축 워크스페이스 관리** — 파일시스템별 독립적인 압축 워크스페이스 관리자 도입으로 다중 Btrfs 인스턴스 환경에서의 리소스 격리

### 배경 및 영향
- 체크섬 검색 최적화는 읽기 집약적 워크로드에서 가장 큰 효과. 스냅샷이 많은 환경에서 특히 유의미한 성능 개선
- block size > page size 지원은 ARM64(4KB 페이지) 환경에서 더 큰 블록 크기를 사용하기 위한 장기 프로젝트의 일환
- 6.16의 extent buffer writeback +50% 향상에 이어 6.18에서도 성능 최적화 지속

---

## XFS

### 변경사항
- **온라인 fsck 기본 활성화** — CONFIG_XFS_ONLINE_SCRUB이 기본으로 활성화. 마운트된 상태에서 파일시스템 일관성 검사 가능
- **새 시스콜 기반 속성 설정** — 파일시스템 속성을 새 시스콜을 통해 설정하는 메커니즘 추가
- **V4 포맷 지원 제거** — 레거시 XFS V4 온디스크 포맷 지원 Kconfig에서 제거
- **Zoned 모드 개선** — Zoned 블록 디바이스 지원 소규모 개선
- **XFS_IOC_DIOINFO** — vfs_getattr을 통한 Direct I/O 정보 ioctl 구현
- **온라인 복구 reap 계산 개선** — 온라인 수리 시 공간 회수 계산 정확도 향상

### 배경 및 영향
- 온라인 fsck 기본 활성화는 XFS의 자가 복구 능력을 프로덕션 환경에서 기본으로 제공하겠다는 의지
- V4 포맷 제거는 2018년 이후 deprecated된 레거시 포맷의 최종 정리. 현대 XFS는 V5 포맷만 사용
- 6.16의 large atomic write에 이어 안정성 및 관리 기능 중심의 개선

---

## Ext4

### 변경사항
- **마운트된 슈퍼블록 업데이트 ioctl** — tune2fs 등의 도구가 블록 디바이스 직접 쓰기 없이 마운트된 상태에서 슈퍼블록 파라미터를 변경할 수 있는 ioctl 인터페이스

### 배경 및 영향
- 기존에는 tune2fs가 블록 디바이스에 직접 쓰기로 슈퍼블록을 수정하여 마운트된 파일시스템과의 불일치 위험이 있었음
- 새 ioctl은 커널을 통한 안전한 슈퍼블록 업데이트를 보장
- 6.16의 대형 folio/atomic write 대비 규모는 작지만, 운영 안정성에 기여하는 실용적 개선

---

## FUSE

### 변경사항
- **COPY_FILE_RANGE_64** — 대용량 파일 복사를 위한 64비트 범위 지원. 기존 32비트 제한 해소
- **Prune 알림** — FUSE 서버에 캐시 정리(prune) 알림 전달
- **동기식 FUSE_INIT** — FUSE 초기화 과정을 동기적으로 수행하는 옵션 추가
- **fuseblk FUSE_SYNCFS** — 블록 기반 FUSE 서버에서 syncfs 연산 활성화

### 배경 및 영향
- COPY_FILE_RANGE_64는 Android의 FUSE 기반 /sdcard에서 대용량 미디어 파일 복사 시 효율 향상 가능
- 동기식 FUSE_INIT은 초기화 순서가 중요한 환경(부팅 시 FUSE 마운트 등)에서의 안정성 개선
- 6.16의 대형 folio(10 커밋) 이후 기능 확장 지속

---

## NFS

### 변경사항
- **RWF_DONTCACHE 플래그 지원** — preadv2()/pwritev2()에서 캐시 회피 플래그 지원
- **Direct I/O 정렬 개선** — LOCALIO의 정렬 불일치 DIO 연산 처리
- **STATX_DIOALIGN 지원** — NFSD filecache에서 Direct I/O 정렬 요구사항 노출
- **pnfs 대형 extent 배열** — pNFS에서 대형 extent 배열 지원으로 대규모 레이아웃 처리 가능
- **flexfiles 스트라이프 레이아웃** — flexfiles에서 스트라이프 레이아웃 구성 지원
- **I/O 캐싱 비활성화 프로토타입** — NFS 서버에서 I/O 캐싱을 비활성화하여 저메모리 시스템에서 동작 가능하게 하고 대규모 워크로드의 쓰래싱 감소

### 배경 및 영향
- RWF_DONTCACHE는 대용량 스트리밍 I/O에서 페이지 캐시 오염을 방지
- I/O 캐싱 비활성화 프로토타입은 NFS 서버의 확장성 문제를 근본적으로 해결하는 새로운 접근
- 6.16의 4MB I/O 크기 확대에 이어 성능 및 확장성 개선 지속

---

## SMB/CIFS

### 변경사항
- **smbdirect 소켓 통합** — 인커널 클라이언트/서버 간 smbdirect 공통 구조체 및 소켓 코드 통합. 광범위한 리팩토링
- **ksmbd max IP connections 파라미터** — 커널 SMB 서버에서 IP별 최대 연결 수 제한 설정
- **arc4 라이브러리 사용** — RC4 암호화에 arc4 커널 라이브러리 활용
- **클라이언트 drop_dir_cache 모듈 파라미터** — 디렉터리 캐시 해제를 위한 새 모듈 파라미터

### 배경 및 영향
- smbdirect 통합은 RDMA 네트워크 지원의 코드 중복을 제거하고 유지보수성 향상
- ksmbd 연결 제한은 DoS 방어 및 리소스 관리에 유용

---

## SquashFS

### 변경사항
- **SEEK_DATA/SEEK_HOLE 지원** — lseek()에서 데이터/홀 탐색 지원으로 효율적인 파일 복사 가능
- **추가 inode 건전성 검사** — inode 메타데이터에 대한 추가 유효성 검증

### 배경 및 영향
- SEEK_DATA/SEEK_HOLE은 cp --sparse=always 등 도구의 효율성 향상
- SquashFS는 Android system 파티션의 읽기 전용 이미지에 사용 가능하므로 관련성 있음

---

## exFAT

### 변경사항
- **FS_IOC_GETFSLABEL/SETFSLABEL 지원** — 파일시스템 레이블 조회/설정 ioctl
- **할당 비트맵 로딩 최적화** — 비트맵 로딩 성능 개선
- **리마운트 옵션 수정** — 리마운트 시 옵션 변경 지원

### 배경 및 영향
- exFAT는 SD 카드 등 이동식 미디어의 표준 파일시스템으로, Android 디바이스에서 광범위하게 사용
- 비트맵 로딩 최적화는 대용량 SD 카드 마운트 시간 단축에 기여

---

## F2FS

### 변경사항
- **lookup_mode 마운트 옵션** — casefold 디렉터리에서의 조회 모드 제어
- **특권 사용자용 예약 노드** — 디스크 공간 부족 시에도 특권 사용자 연산을 위한 노드 예약
- **F2FS_GET_BLOCK_PRECACHE 모드 노드 블록 readahead** — 프리캐시 모드에서 노드 블록 선읽기 지원

### 배경 및 영향
- F2FS는 Android 데이터 파티션의 주요 파일시스템
- 특권 사용자 예약 노드는 디스크 풀 상황에서의 시스템 안정성 개선
- casefold lookup_mode는 Android의 대소문자 무시 파일 시스템 지원과 관련

---

## 기타 파일시스템

### OverlayFS
- **Casefold 레이어 지원** — 대소문자 무시 레이어 구성 가능

### AFS
- **RENAME_NOREPLACE/RENAME_EXCHANGE 지원** — 원자적 rename 연산 지원 추가

### NTFS3
- **FS_IOC_SETFSLABEL ioctl** — 파일시스템 레이블 설정 지원

### DLM
- **configfs release_recover 엔트리** — lockspace 멤버에 대한 새 configfs 복구 해제 인터페이스

---

## 이전 버전과의 연속성

- **Bcachefs**: 6.15 스크러빙/case-folding → 6.16 비상 읽기 전용/단일 디바이스 → 6.18 메인라인 제거. 급격한 방향 전환
- **Btrfs**: 6.16 extent buffer writeback +50% → 6.18 체크섬 검색 최적화 + block size > page size 준비. 성능/확장성 개선 지속
- **XFS**: 6.16 large atomic write → 6.18 온라인 fsck 기본 활성화 + V4 제거. 레거시 정리와 자가 복구 강화
- **Ext4**: 6.16 대형 folio 37% + atomic write → 6.18 슈퍼블록 ioctl. 대형 성능 개선 후 운영 안정성 보완
- **FUSE**: 6.16 대형 folio 10 커밋 → 6.18 COPY_FILE_RANGE_64 + 동기식 INIT. 기능 확장 지속
- **Folio 전환**: 6.16의 Ext4/FUSE/Btrfs folio 도입에 이어 6.18에서 kernel file mapped folios 개념 도입으로 folio 활용 범위 더욱 확대

---

## 참고 자료

- [kernelnewbies.org — Linux 6.18](https://kernelnewbies.org/Linux_6.18)
- [Phoronix — Linux 6.18 Released](https://www.phoronix.com/news/Linux-6.18-Released)
- [Phoronix — Linux 6.18 Features](https://www.phoronix.com/news/LInux-6.18-Features-Reminder)
- [The Register — Version 6.18 of the Linux kernel is here](https://www.theregister.com/2025/12/03/kernel_version_618/)
- [CNX Software — Linux 6.18 release](https://www.cnx-software.com/2025/12/01/linux-6-18-release-main-changes-arm-risc-v-and-mips-architectures/)
