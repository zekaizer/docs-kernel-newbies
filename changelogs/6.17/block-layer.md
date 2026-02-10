# Linux 6.17 블록 레이어 변경사항

> 출처: kernelnewbies.org, LWN.net, LKML
> 릴리스: Linux 6.17
> 작성일: 2026-02-10

---

## 개요

Linux 6.17 블록 레이어는 FALLOC_FL_WRITE_ZEROES 플래그로 플래시 스토리지의 효율적 제로 쓰기, WBT 최적화, ublk zero-copy 버퍼 등록 및 서버 종료 처리 속도 향상, UFS HID/MediaTek UFS 개선, pktcdvd 드라이버 최종 제거가 핵심이다.

---

## FALLOC_FL_WRITE_ZEROES

### 변경사항
- **새 fallocate(2) 플래그** — 사전 할당 파일에 대해 디바이스의 하드웨어 기능(UNMAP/DEAC 명령)을 활용하여 대역폭 소비 없이 효율적으로 제로를 쓰는 플래그 (kernelnewbies.org)
- NVMe SSD, SCSI 디바이스에서 실제 I/O 전송 없이 제로 블록 할당

### 배경 및 영향
- 기존 fallocate + ZERO_RANGE는 파일시스템 레벨에서 제로를 기록했으나, WRITE_ZEROES는 디바이스 하드웨어에 직접 제로 쓰기 명령 전달
- 플래시 스토리지의 마모(wear) 감소, 대용량 파일 사전 할당 속도 향상
- 데이터베이스 로그 파일, VM 디스크 이미지 사전 할당 시나리오에서 효과적

---

## WBT (Writeback Throttling) 최적화

### 변경사항
- **WBT 최적화 및 문서 업데이트** — Writeback Throttling 알고리즘 최적화 (kernelnewbies.org)
- **회전 디바이스 readahead 개선** — HDD의 readahead 로직 개선

### 배경 및 영향
- WBT는 writeback 큐가 과도하게 쌓이지 않도록 I/O를 조절하는 메커니즘. 최적화로 SSD/HDD 모두에서 쓰기 지연 감소
- 회전 디바이스 readahead는 HDD 기반 서버의 순차 읽기 성능 향상

---

## ublk 개선

### 변경사항
- **Off-daemon zero-copy 버퍼 등록** — 유저스페이스 블록 디바이스(ublk)에서 데몬 없이 zero-copy 버퍼 등록 (kernelnewbies.org)
- **서버 종료 처리 속도 향상** — ublk 서버 프로세스 종료 시 정리 과정 최적화

### 배경 및 영향
- ublk는 유저스페이스에서 블록 디바이스를 구현하는 프레임워크. zero-copy 버퍼 등록으로 데이터 복사 오버헤드 제거
- 서버 종료 속도 향상은 컨테이너 환경에서 ublk 디바이스의 빠른 정리에 도움

---

## UFS (Universal Flash Storage) 개선

### 변경사항
- **UFS HID(Host Initiated Defrag) 지원** — 호스트가 UFS 디바이스에 조각 모음 요청 (kernelnewbies.org)
- **MediaTek UFSCHI 하드웨어 버전 지원** — MediaTek UFS Host Controller Interface 버전 관리
- **MediaTek FDE(Full Disk Encryption) 클럭 스케일링** — 암호화 엔진의 클럭 주파수 조절
- **MediaTek Vcore 바인딩 클럭 스케일링** — Vcore 전압 도메인과 연동된 클럭 관리
- **Qualcomm QUnipro Internal Clock Gating** — QUnipro 내부 클럭 게이팅으로 전력 절약
- **Intel Wildcat Lake PCI 지원** — Intel 차세대 플랫폼 UFS 지원

### 배경 및 영향
- UFS HID는 안드로이드 기기의 스토리지 성능 유지에 중요. 장기 사용 시 성능 저하를 호스트 주도 조각 모음으로 방지
- MediaTek FDE/Vcore 클럭 스케일링은 모바일 SoC의 전력 효율과 암호화 성능 최적화에 직접 관련
- Qualcomm QUnipro 클럭 게이팅은 Snapdragon 기반 Android 기기의 UFS 전력 소비 감소

---

## Device Mapper 개선

### 변경사항
- **dm-thin crypto 기능** — thin provisioning에서 암호화 기능 지원 (kernelnewbies.org)
- **dm-verity 비동기 해시 제거** — 비동기 해시 경로 정리

### 배경 및 영향
- dm-thin crypto는 thin provisioned 볼륨의 암호화 지원. 컨테이너 스토리지 보안 강화
- dm-verity 비동기 해시 제거는 코드 정리로, 동기 해시 경로의 효율이 충분하다고 판단

---

## FS_IOC_GETLBMD_CAP ioctl

### 변경사항
- **새 ioctl** — 메타데이터/보호 정보 기능 조회를 위한 FS_IOC_GETLBMD_CAP ioctl 추가

### 배경 및 영향
- 블록 디바이스의 논리 블록 메타데이터(T10 PI, DIF/DIX) 지원 여부를 유저스페이스에서 확인
- 데이터 무결성 보호가 필요한 스토리지 환경에서 활용

---

## pktcdvd 드라이버 제거

### 변경사항
- **pktcdvd 패킷 쓰기 광학 드라이버 완전 제거** — 2016년 deprecated, 2023년 제거 시도 실패 후 6.17에서 최종 삭제 (kernelnewbies.org, OMG Ubuntu)

### 배경 및 영향
- CD/DVD 패킷 쓰기는 현대 시스템에서 사용되지 않는 레거시 기능. 코드 정리 차원의 제거

---

## MD/RAID

### 변경사항
- **md faulty rdev 제거** — resync 중 결함 디바이스 제거 처리 개선

### 배경 및 영향
- RAID 어레이의 안정성 향상. 재동기화 중 결함 디바이스를 안전하게 제거

---

## 이전 버전과의 연속성

- **ublk 진화**: 6.16 서버 스레드 디커플링/자동 bvec/quiesce → 6.17 zero-copy 버퍼/서버 종료 최적화
- **UFS SoC 지원**: 6.16 (기본 UFS) → 6.17 HID, MediaTek FDE/Vcore, Qualcomm QUnipro
- **코드 정리**: 6.16 바운스 버퍼 완전 제거 → 6.17 pktcdvd 드라이버 제거, dm-verity 비동기 해시 제거
- **블록 레이어 효율화**: 6.17 FALLOC_FL_WRITE_ZEROES로 하드웨어 제로 쓰기 표준화

---

## 참고 자료
- [kernelnewbies.org Linux 6.17](https://kernelnewbies.org/Linux_6.17)
- [LWN.net 6.17 merge window](https://lwn.net/Articles/1031713/)
- [Phoronix Linux 6.17 Features](https://www.phoronix.com/news/Linux-6.17-Features-Reminder)
