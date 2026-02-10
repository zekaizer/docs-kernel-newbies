# Linux 6.17 시스콜 변경사항

> 출처: kernelnewbies.org, LWN.net, LKML
> 릴리스: Linux 6.17
> 작성일: 2026-02-10

---

## 개요

Linux 6.17은 file_getattr()/file_setattr() 두 개의 새 시스템 콜과 FALLOC_FL_WRITE_ZEROES fallocate 플래그, scm_pidfd의 수확된 pidfd 수신, open_by_handle_at() 파일 핸들 기반 열기, syscall_user_dispatch PR_SYS_DISPATCH_INCLUSIVE_ON 지원이 핵심이다.

---

## file_getattr() / file_setattr()

### 변경사항
- **새 시스콜** — openat(2) 시맨틱으로 확장 파일 속성(inode attributes)을 조작하는 두 개의 시스템 콜 추가 (kernelnewbies.org, LWN.net)
- 파일을 열지 않고도 inode 속성 조회 및 설정 가능
- 기존 ioctl 기반 속성 조작의 현대적 대안

### 배경 및 영향
- 기존에는 파일 속성(immutable, append-only, compression 등)을 조작하려면 파일을 먼저 열고 ioctl(FS_IOC_GETFLAGS/FS_IOC_SETFLAGS)을 호출해야 했음
- file_getattr/file_setattr는 openat2 스타일의 경로 기반 접근으로 더 안전하고 유연한 속성 관리 제공
- 심볼릭 링크 추적 제어, 마운트 네임스페이스 제한 등 openat(2)의 보안 기능 활용 가능

---

## FALLOC_FL_WRITE_ZEROES

### 변경사항
- **새 fallocate(2) 플래그** — 디바이스 하드웨어(NVMe DEAC, SCSI UNMAP)를 활용한 효율적 제로 쓰기 (kernelnewbies.org)

### 배경 및 영향
- 상세 내용은 [블록 레이어 상세](block-layer.md) 참조
- 기존 FALLOC_FL_ZERO_RANGE와 달리 하드웨어 명령을 직접 활용하여 I/O 대역폭 소비 없이 제로 블록 생성

---

## scm_pidfd 확장

### 변경사항
- **수확된(reaped) pidfd 수신** — SCM_RIGHTS를 통해 수확된 pidfd를 수신하는 기능

### 배경 및 영향
- pidfd는 프로세스를 안전하게 참조하는 파일 디스크립터. 수확된 pidfd는 이미 종료된 프로세스의 정보를 안전하게 전달
- 프로세스 관리 도구, 컨테이너 런타임에서 프로세스 수명 주기 관리에 활용

---

## open_by_handle_at() 확장

### 변경사항
- **파일 핸들 기반 순수 열기** — open_by_handle_at()에서 파일 핸들만으로 파일 열기 (kernelnewbies.org)

### 배경 및 영향
- NFS 서버, 파일시스템 백업 도구에서 파일 핸들 기반 접근에 활용

---

## syscall_user_dispatch 확장

### 변경사항
- **PR_SYS_DISPATCH_INCLUSIVE_ON** — syscall_user_dispatch에서 포함 모드(inclusive) 지원 (kernelnewbies.org)

### 배경 및 영향
- syscall_user_dispatch는 유저스페이스에서 시스콜 에뮬레이션을 수행하는 메커니즘. Wine, 호환 레이어 등에서 사용
- inclusive 모드 추가로 더 유연한 시스콜 필터링 가능

---

## 이전 버전과의 연속성

- **시스콜 현대화**: 6.16 PTRACE_SET_SYSCALL_INFO/uselib() 제거 → 6.17 file_getattr/file_setattr/FALLOC_FL_WRITE_ZEROES
- **pidfd 진화**: 매 릴리스 pidfd 관련 기능 확대. 6.17에서 수확된 pidfd 수신 지원

---

## 참고 자료
- [kernelnewbies.org Linux 6.17](https://kernelnewbies.org/Linux_6.17)
- [LWN.net 6.17 release](https://lwn.net/Articles/1038266/)
