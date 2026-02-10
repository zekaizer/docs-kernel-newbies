# Linux 6.16 블록 레이어 변경사항

> 출처: kernelnewbies.org, LWN.net, LKML
> 릴리스: Linux 6.16
> 작성일: 2026-02-10

---

## 개요

Linux 6.16 블록 레이어는 바운스 버퍼링 인프라의 완전 제거, zram 알고리즘별 파라미터 지원, ublk 서버 아키텍처 개선, 새 zoned 루프 디바이스 드라이버가 핵심이다. device mapper와 md에서도 의미 있는 개선이 이루어졌다.

---

## 바운스 버퍼링 완전 제거

### 변경사항
- **블록 레이어 바운스 버퍼 인프라 전면 삭제** — 오래된 바운스 버퍼링(bounce buffering) 코드가 완전히 제거됨

### 배경 및 영향
- 바운스 버퍼링은 DMA 제약(예: 32비트 DMA 한계)이 있는 오래된 하드웨어를 위해 I/O 버퍼를 낮은 메모리 영역에 복사하는 메커니즘이었음
- 현대 하드웨어에서는 IOMMU가 이 역할을 대신하므로 바운스 버퍼가 불필요
- 코드 제거로 블록 I/O 경로의 복잡성 감소 및 코드 유지보수 부담 경감
- 매우 오래된 하드웨어(ISA DMA, 초기 PCI)에서만 영향 가능

---

## zram

### 변경사항
- **알고리즘별 파라미터 지원** — 압축 알고리즘(lz4, zstd, lzo 등)에 대한 개별 설정 파라미터 지정 가능
- **writeback 인터페이스 현대화** — zram writeback 인터페이스의 API 개선

### 배경 및 영향
- Android에서 zram은 기본 스왑 백엔드로 사용. 알고리즘별 파라미터로 압축률과 속도 간 세밀한 트레이드오프 설정 가능
- 예: zstd의 압축 레벨, lz4의 가속 모드 등을 런타임에 조정
- writeback 인터페이스 개선은 zram의 디스크 기반 writeback(cold page를 실제 스왑 디바이스로 이동) 기능의 사용성 향상

---

## ublk (User Block Device)

### 변경사항
- **UBLK_U_CMD_UPDATE_SIZE** — 런타임에 블록 디바이스 크기 업데이트 명령
- **UBLK_F_QUIESCE** — 블록 디바이스를 일시 정지(quiesce)하는 기능
- **자동 bvec 버퍼 등록** — bio_vec 버퍼의 자동 등록으로 설정 간소화
- **서버 스레드 디커플링** — ublk 서버 스레드를 queue/hctx 바인딩에서 분리하여 유연한 스레드 관리

### 배경 및 영향
- ublk는 유저스페이스에서 블록 디바이스를 구현하는 프레임워크로, io_uring 기반의 고성능 I/O 경로 제공
- 서버 스레드 디커플링은 ublk 데몬의 스레드 모델을 유연하게 만들어 복잡한 스토리지 스택 구현에 유리
- QUIESCE 기능은 블록 디바이스의 무중단 업데이트/마이그레이션에 필요

---

## Zoned 루프 디바이스

### 변경사항
- **새 zoned 루프 블록 디바이스 드라이버** — 기존 파일시스템의 여러 파일을 사용하여 zoned 블록 디바이스를 에뮬레이션하는 새 드라이버 (LWN.net)

### 배경 및 영향
- SMR HDD나 ZNS SSD 없이도 zoned 블록 디바이스 개발/테스트 환경 구축 가능
- zoned 스토리지 관련 파일시스템(Btrfs, XFS zoned mode, f2fs) 개발 및 디버깅에 유용한 도구
- 기존 null_blk의 zoned 에뮬레이션보다 더 현실적인 테스트 환경 제공

---

## Device Mapper

### 변경사항
- **dm-bufio**: 최대 age 기반 eviction 제거
- **dm-mpath**: 명시적 active 경로 프로빙 인터페이스
- **dm-crypt**: 래핑된 인라인 암호화 키 패스스루 지원

### 배경 및 영향
- dm-crypt의 래핑된 인라인 암호화 키 패스스루는 하드웨어 암호화(inline encryption) 엔진을 dm-crypt 스택에서 활용하는 기능
- Android의 파일 기반 암호화(FBE)에서 인라인 암호화 엔진과 dm-crypt의 조합에 관련
- dm-mpath의 명시적 프로빙은 다중 경로 스토리지 환경에서 경로 상태 관리의 정확도 향상

---

## md (소프트웨어 RAID)

### 변경사항
- **sync_io_depth API** — 동기화 I/O 깊이를 제어하는 새 API

### 배경 및 영향
- RAID 동기화(resync, recovery) 시 I/O 병렬성을 제어하여 동기화 성능과 일반 I/O 성능 간 균형 조정

---

## 이전 버전과의 연속성

- **바운스 버퍼 제거**: 오랜 deprecation 이후 6.16에서 완전 삭제. 블록 I/O 스택 현대화의 이정표
- **ublk**: 6.15의 vectored registered buffer에 이어 6.16에서 서버 스레드 디커플링과 quiesce로 아키텍처 성숙
- **zram**: Android의 핵심 메모리 관리 컴포넌트. 알고리즘별 파라미터로 더 세밀한 튜닝 가능
- **dm-crypt 인라인 암호화**: Android FBE의 인라인 암호화 지원 확장

---

## 참고 자료

- [kernelnewbies.org — Linux 6.16](https://kernelnewbies.org/Linux_6.16)
- [LWN.net — The first half of the 6.16 merge window](https://lwn.net/Articles/1022512/)
- [LWN.net — The second half of the 6.16 merge window](https://lwn.net/Articles/1023075/)
