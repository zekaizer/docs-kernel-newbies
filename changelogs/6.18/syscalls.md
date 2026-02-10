# Linux 6.18 시스콜 변경사항

> 출처: kernelnewbies.org
> 릴리스: Linux 6.18
> 작성일: 2026-02-10

---

## 개요

Linux 6.18은 새 시스콜 추가보다는 기존 시스콜의 확장과 새 플래그/ioctl 추가에 초점을 맞추었다. 네임스페이스 파일 핸들 확장, RWF_NOSIGNAL 플래그, procfs pidns 마운트 옵션, 다수 파일시스템의 ioctl 추가가 포함된다.

---

## 네임스페이스 파일 핸들

### 변경사항
- **name_to_handle_at() 네임스페이스 확장** — 프로세스 네임스페이스의 파일 핸들 인코딩 지원
- **open_by_handle_at() 네임스페이스 확장** — 인코딩된 네임스페이스 파일 핸들을 통한 네임스페이스 열기
- **pidfd와 유사한 접근** — 네임스페이스를 pidfd처럼 파일 핸들로 참조 가능

### 배경 및 영향
- 컨테이너 관리 도구에서 네임스페이스를 파일 핸들로 안정적으로 참조 가능
- CRIU(Checkpoint/Restore In Userspace) 등의 도구에서 네임스페이스 복원 시 유용
- 기존의 /proc/[pid]/ns/ 심볼릭 링크 기반 접근보다 안정적인 참조 메커니즘

---

## RWF_NOSIGNAL 플래그

### 변경사항
- **pwritev2()에 RWF_NOSIGNAL 플래그 추가** — 쓰기 연산 시 SIGPIPE 시그널 생성 방지

### 배경 및 영향
- 닫힌 파이프/소켓에 쓰기 시 SIGPIPE가 프로세스를 종료시키는 문제 방지
- MSG_NOSIGNAL(소켓)의 파일 I/O 버전
- 멀티스레드 서버 애플리케이션에서 안정성 향상

---

## procfs "pidns" 마운트 옵션

### 변경사항
- **"pidns" 마운트 옵션** — procfs 마운트 시 대상 PID 네임스페이스를 명시적으로 지정 가능

### 배경 및 영향
- 컨테이너 런타임에서 특정 PID 네임스페이스의 procfs를 마운트할 때 유용
- 네임스페이스 격리의 유연성 향상

---

## 파일시스템 ioctl 확장

### FS_IOC_GETFSLABEL / FS_IOC_SETFSLABEL
- **exFAT**: 파일시스템 레이블 조회/설정 지원 추가
- **NTFS3**: 파일시스템 레이블 설정(FS_IOC_SETFSLABEL) 지원 추가

### Ext4 슈퍼블록 업데이트 ioctl
- **마운트된 상태 슈퍼블록 수정** — tune2fs 등의 도구가 마운트된 Ext4의 슈퍼블록 파라미터를 안전하게 변경 가능

### XFS_IOC_DIOINFO
- **vfs_getattr 기반 구현** — Direct I/O 정보 조회를 vfs_getattr으로 구현

---

## RWF_DONTCACHE 플래그

### 변경사항
- **NFS 클라이언트 초기 지원** — preadv2()/pwritev2()에서 캐시 회피 플래그의 NFS 클라이언트 지원

### 배경 및 영향
- 대용량 스트리밍 I/O에서 페이지 캐시 오염 방지
- 캐시가 유용하지 않은 일회성 대량 데이터 전송에 최적

---

## 이전 버전과의 연속성

- **네임스페이스 관리**: 네임스페이스 파일 핸들은 컨테이너/가상화 관리 인프라의 점진적 개선
- **pwritev2 플래그**: RWF_NOSIGNAL은 6.16의 RWF 플래그 확장 트렌드 지속
- **파일시스템 ioctl**: exFAT/NTFS3의 레이블 관리 ioctl은 VFS 표준 인터페이스 채택 확대

---

## 참고 자료

- [kernelnewbies.org — Linux 6.18](https://kernelnewbies.org/Linux_6.18)
