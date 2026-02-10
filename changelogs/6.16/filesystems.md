# Linux 6.16 파일시스템 변경사항

> 출처: kernelnewbies.org, LWN.net, LKML
> 릴리스: Linux 6.16
> 작성일: 2026-02-10

---

## 개요

Linux 6.16에서 파일시스템 영역은 Ext4의 대형 folio/atomic write/fast commit 대폭 개선, Btrfs의 extent buffer writeback 성능 향상(+50% 처리량), Bcachefs의 안정성 강화, FUSE 대형 folio 지원, OverlayFS 비특권 컨테이너 지원이 핵심이다. NFS는 최대 I/O 크기를 4MB로 확대했다.

---

## Ext4

### 변경사항
- **대형 folio 지원 (일반 파일)** — 일반 파일에 대한 대형 folio 읽기/쓰기 지원으로 대용량 순차 I/O 워크로드에서 37% 성능 향상 (kernelnewbies.org)
- **Multi-fsblock atomic write (bigalloc)** — bigalloc 파일시스템에서 다중 파일시스템 블록에 걸친 원자적 쓰기 지원. 쓰기가 완전히 완료되거나 전혀 반영되지 않음을 보장
- **Fast commit 경로 최적화** — 저널링 fast commit 경로에 대한 다수의 성능 개선 패치
- **DAX 지원 없는 ext2 deprecation 참조** — ext2의 DAX 지원이 deprecated됨 (2025년 말 완전 제거 예정). 영속 메모리(persistent memory)의 시장 확산이 예상보다 부진

### 배경 및 영향
- Ext4의 대형 folio 도입은 커널 전반의 folio 전환 트렌드에서 가장 실질적인 성능 효과를 보여주는 사례. 37% 향상은 대용량 순차 쓰기 벤치마크에서의 수치
- Atomic write는 6.13 XFS/Ext4 direct I/O → 6.15 XFS COW → 6.16 Ext4 bigalloc 확장으로 지속 확대
- Fast commit은 ext4의 저널링 오버헤드를 줄이는 핵심 기술로, 6.16에서 추가 최적화

---

## Btrfs

### 변경사항
- **Extent buffer writeback 개선** — 메타데이터 집약적 연산에서 처리량 50% 향상, 런타임 33% 감소 (kernelnewbies.org)
- **Extent unpinning 효율화** — 트랜잭션 커밋 시 extent unpinning에서 3-5% 런타임 개선 (kernelnewbies.org)
- **대형 데이터 folio defrag 지원** — defragmentation 연산에서 대형 데이터 folio 활용
- **블록 수준 완벽 압축** — 실험적 기능에서 정식 기능으로 전환 (block perfect compression)
- **scrub_sector_verification 메모리 감소** — 스크럽 검증 시 메모리 사용량 절감

### 배경 및 영향
- Extent buffer writeback 개선은 Btrfs의 메타데이터 I/O 병목을 크게 완화. 스냅샷, 서브볼륨, 밸런스 연산 등 메타데이터 집약적 워크로드에서 실질적 성능 향상
- Block perfect compression의 정식 전환은 압축 비율 최적화에 기여. 기존에는 압축 단위가 고정 크기였으나, 블록 경계에 맞춘 압축으로 공간 활용 효율 증가

---

## Bcachefs

### 변경사항
- **단일 디바이스 모드** — 다중 디바이스 없이 단일 디바이스에서의 운용 지원
- **스냅샷 삭제 개선** — 더 빠른 스냅샷 삭제 처리
- **Recovery passes 슈퍼블록 섹션** — 복구 패스를 슈퍼블록에 기록하여 복구 상태 추적 강화
- **비상 읽기 전용 트리거** — 심각한 오류 발생 시 자동으로 읽기 전용 모드 전환
- **비동기 객체 디버깅 인프라** — 런타임 디버깅을 위한 인프라 추가
- **AC 전원 시에만 리밸런스** — 배터리 전원에서는 리밸런스 연산 억제 (LWN.net)

### 배경 및 영향
- Bcachefs는 6.15의 스크러빙/case-folding에 이어 6.16에서 안정성과 복구 메커니즘을 강화
- 단일 디바이스 모드는 데스크톱/노트북 사용 사례를 위한 접근성 향상
- 비상 읽기 전용 모드는 데이터 손실 방지를 위한 안전장치로, 프로덕션 환경 적용 가능성 높임
- AC 전원 리밸런스 조건은 임베디드/노트북 환경에서 배터리 소모 방지에 유용

---

## FUSE

### 변경사항
- **대형 folio 지원** — 10개 커밋으로 FUSE에 대형 folio 읽기/쓰기 지원 추가
- **캐시 무효화 동작 제어** — 단일 연산으로 모든 dentry 캐시 무효화 가능 (LWN.net)
- **/dev/fuse fdinfo 디바이스 ID** — fdinfo에서 FUSE 디바이스 식별자 노출
- **readdir 버퍼 크기 증가** — 디렉터리 읽기 시 더 큰 버퍼 사용으로 성능 개선

### 배경 및 영향
- FUSE의 대형 folio 지원은 유저스페이스 파일시스템(Android의 FUSE 기반 스토리지 포함)의 대용량 I/O 성능을 잠재적으로 개선
- 단일 dentry 무효화 연산은 마운트 지점 변경이나 파일시스템 갱신 시 캐시 일관성 관리를 크게 단순화

---

## NFS

### 변경사항
- **읽기/쓰기 크기 제한 4MB로 확대** — 기본값은 1MB 유지, 최대 4MB까지 설정 가능 (LWN.net)
- **FALLOC_FL_ZERO_RANGE 지원** — NFS에서 파일 영역 제로화 연산 지원
- **localio sysfs 노출** — 로컬 I/O 경로를 sysfs를 통해 관리
- **비동기 LOCALIO 프로빙** — 로컬 I/O 감지를 비동기로 수행
- **/sys/kernel/debug/nfsd 엔드포인트** — NFS 데몬 디버깅 인터페이스 추가
- **FATTR4_CLONE_BLKSIZE 속성** — 클론 블록 크기 속성 구현
- **CB_OFFLOAD referring call lists** — 오프로드 콜백 참조 목록 지원

### 배경 및 영향
- 4MB I/O 크기 지원은 고속 네트워크(25GbE+) 환경에서 NFS 처리량을 크게 개선할 수 있는 변경
- ZERO_RANGE 지원은 데이터센터 환경에서의 파일 사전 할당 및 초기화 워크로드에 유용

---

## OverlayFS

### 변경사항
- **비특권 사용자 네임스페이스에서 data-only 레이어 지원** — 메타데이터/데이터 전용 레이어를 비특권 네임스페이스에서 사용 가능
- **dm-verity data-only 레이어** — 신뢰할 수 있는 메타데이터 레이어와 비신뢰 데이터 레이어를 비특권 네임스페이스에서 결합 (LWN.net)
- **composefs 호환성** — 비특권 컨테이너를 위한 composefs 지원

### 배경 및 영향
- 이 변경은 컨테이너 런타임(특히 루트리스 컨테이너)의 보안 모델을 크게 강화
- dm-verity와의 통합으로 OCI 컨테이너 이미지의 무결성을 비특권 환경에서도 검증 가능
- Android의 APEX/APK 오버레이 사용 사례에도 잠재적 영향

---

## EROFS

### 변경사항
- **Intel QAT deflate 압축 해제** — Intel QuickAssist Technology를 활용한 하드웨어 가속 deflate 압축 해제
- **'fsoffset' 마운트 옵션** — 파일시스템 오프셋 지정을 위한 새 마운트 옵션

### 배경 및 영향
- QAT 하드웨어 가속은 서버 환경에서 EROFS 읽기 성능을 크게 개선 가능. 데이터센터의 읽기 전용 이미지 배포에 유용
- Android에서 EROFS는 system 파티션 이미지에 사용 가능하므로, 하드웨어 가속 압축 해제는 부팅 속도 개선에 기여 가능

---

## SMB

### 변경사항
- **smbdirect 통합** — 인커널 클라이언트/서버 간 smbdirect 헤더 및 구조체 통합
- **ParentLeaseKey 지원** — 클라이언트에서 상위 lease 키 지원
- **디렉터리 캐시 재사용 개선** — 디렉터리 캐시 활용 효율 향상

### 배경 및 영향
- smbdirect 통합은 커널 내 SMB 구현의 코드 중복을 제거하고 RDMA 지원의 일관성 향상

---

## 기타 파일시스템

### F2FS
- **encoding flags export** — 인코딩 플래그 내보내기
- **injection stats proc 엔트리** — 오류 주입 통계 노출
- **FAULT_TIMEOUT 지원** — 타임아웃 기반 오류 주입
- **선형 검색 기능 export** — 선형 조회 기능 내보내기

### Squashfs
- **전체 압축 블록 캐싱 (선택적)** — 압축된 전체 블록을 캐시하는 옵션 추가

### EXT2
- **DAX 지원 deprecated** — 2025년 말 완전 제거 예정

### 기타
- **UFS**: 새 mount API 전환
- **OMFS, OrangeFS, BFS**: 새 mount API 전환
- **NTFS3**: 압축 변경 기능 제거

---

## 이전 버전과의 연속성

- **Atomic write**: 6.13 XFS/Ext4 Direct I/O 도입 → 6.15 XFS COW 결합 → 6.16 Ext4 bigalloc 다중 블록 원자적 쓰기 + XFS large atomic write
- **Folio 전환**: 커널 전반의 대형 folio 확산이 6.16에서 Ext4(37% 성능 향상), FUSE(10 커밋), Btrfs(defrag)에서 실질적 성과
- **Bcachefs 성숙**: 6.15 스크러빙/case-folding → 6.16 비상 읽기 전용, 단일 디바이스 모드, 복구 인프라 강화
- **OverlayFS 컨테이너 지원**: 비특권 네임스페이스에서의 dm-verity + data-only 레이어 지원은 루트리스 컨테이너 보안의 중요한 진전

---

## 참고 자료

- [kernelnewbies.org — Linux 6.16](https://kernelnewbies.org/Linux_6.16)
- [LWN.net — The first half of the 6.16 merge window](https://lwn.net/Articles/1022512/)
- [LWN.net — The second half of the 6.16 merge window](https://lwn.net/Articles/1023075/)
- [Phoronix — Linux 6.16 Features](https://www.phoronix.com/news/Linux-6.16-Features-Early-Look)
- [linuxiac — Linux Kernel 6.16 Released](https://linuxiac.com/linux-kernel-6-16-released-this-is-whats-new/)
