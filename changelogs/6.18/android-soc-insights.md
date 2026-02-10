# Linux 6.18 — Android SoC 개발 인사이트

> 릴리스: Linux 6.18 (2025-11-30)
> 작성일: 2026-02-10

---

## 개요

Linux 6.18은 Android SoC BSP 팀에게 **전략적으로 매우 중요한 릴리스**이다. 첫째, 2025년 LTS 커널로 지정될 가능성이 높아 장기 유지보수 대상이 된다. 둘째, Rust Binder 드라이버와 Tyr Mali GPU Rust 드라이버가 병합되어 Android 핵심 컴포넌트의 Rust 전환이 시작되었다. 셋째, Swap Table Phase I과 Slub Sheaves로 메모리 서브시스템이 근본적으로 개선되었으며, 이는 메모리 제약이 큰 모바일 환경에 직접적 영향을 미친다.

---

## BSP 영향도 분석

### 즉시 대응 필요 (High Impact)

#### 1. Rust Binder 드라이버 병합 — BSP 빌드 구성 검토 필수
- Rust Binder와 C Binder가 빌드 시 선택 가능한 구조로 공존
- **즉시 조치**: BSP 빌드 시스템에서 CONFIG_ANDROID_BINDER_IPC와 Rust Binder 선택 옵션 검토. 현 시점에서는 C 드라이버 유지 권장
- **검증 계획**: Rust Binder 활성화 시 CTS/VTS Binder 관련 테스트 전수 수행 필요
- **리스크**: CVE-2025-68260(경쟁 조건으로 인한 커널 패닉)이 이미 발견됨. 프로덕션 적용 전 패치 확인 필수

#### 2. Bcachefs 제거 — BSP 빌드 영향 확인
- Bcachefs 관련 Kconfig 옵션 참조가 있는 BSP는 빌드 오류 발생 가능
- **즉시 조치**: defconfig에서 CONFIG_BCACHEFS_FS 관련 옵션 제거

#### 3. LTS 커널 가능성 — 장기 유지보수 전략 수립
- 6.18이 2025년 LTS(2027년 12월까지 지원)로 지정될 가능성 높음
- **즉시 조치**: 차기 GKI 베이스라인으로 6.18 채택 가능성 평가. Android 15/16 GKI 호환성 매트릭스 확인

#### 4. memdesc_flags_t 도입 — 벤더 드라이버 메모리 API 호환성
- struct page/slab/folio 분리를 위한 새 플래그 타입 도입
- **즉시 조치**: 벤더 드라이버에서 struct page 내부 플래그에 직접 접근하는 코드 감사. memdesc_flags_t API로 전환 필요 여부 확인

### 중기 검토 필요 (Medium Impact)

#### 5. Swap Table Phase I — zram 성능 검증
- 스왑 처리량 5-20% 향상, 96 작업/10GB zRAM 환경에서 시스템 시간 50% 감소
- **검증 필요**: Android 디바이스 zram 환경(보통 2-4GB)에서의 성능 향상 측정
- **벤치마크 권장**: 앱 스위칭 시나리오, 저메모리 압박 상황에서의 반응성 측정
- 스왑 클러스터 스캐닝 개선으로 mTHP 기반 대형 페이지 할당 실패율 감소

#### 6. Slub Sheaves — 메모리 할당 성능 변화
- per-CPU 캐시로 할당/해제 동기화 오버헤드 감소
- **검증 필요**: 앱 시작 시간, 프로세스 생성 속도 등 할당 집약적 시나리오 벤치마크
- NUMA 지역성 관리 변화에 따른 빅코어/리틀코어 메모리 접근 패턴 확인

#### 7. DAMON ARM32 LPAE — 레거시 SoC 모니터링
- ARM32 LPAE 환경에서 DAMON 사용 가능
- **적용 대상**: 32비트 ARM SoC 기반 레거시 Android 디바이스에서 메모리 접근 패턴 모니터링 가능
- stat-purpose DAMOS 필터로 메모리 정책의 정밀 조정 가능

#### 8. KASAN hw-tags write_only — 프로덕션 메모리 안전성
- ARM MTE 기반 KASAN에서 쓰기만 감시하는 모드
- **적용 가치**: 프로덕션 빌드에서도 KASAN을 낮은 오버헤드로 활성화하여 use-after-free/버퍼 오버플로 감지 가능
- Android의 MTE 정책과 연계하여 보안 강화 전략 수립

#### 9. VMA per-vma lock 확장 — /proc 기반 모니터링 성능
- /proc/pid/maps 읽기에 per-vma lock 적용
- **효과**: Android 프로파일링 도구(simpleperf, perfetto), 디버거, 보안 모니터링의 성능 향상

#### 10. F2FS 개선 — 데이터 파티션 안정성
- 특권 사용자 예약 노드: 디스크 풀 상황에서 시스템 안정성 향상
- casefold lookup_mode: Android의 대소문자 무시 파일 시스템 지원 제어
- 노드 블록 readahead: 프리캐시 모드 성능 향상

### 참고 사항 (Low Impact / Watch)

#### 11. Tyr Mali GPU Rust 드라이버 — 장기 전략 모니터링
- 현재 실험적 상태로 프로덕션 적용 불가
- **모니터링**: Panthor 대체 타임라인 추적. 다운스트림에서는 이미 SuperTuxKart 수준 동작 확인
- Android Mali GPU 드라이버 전략에 장기적 영향

#### 12. Rust 핵심 언어 승격 — 드라이버 개발 전략
- Rust가 커널 핵심 언어로 공식 승격
- **전략적 함의**: 향후 새 커널 서브시스템이 Rust로 작성될 가능성. 벤더 드라이버 팀의 Rust 역량 확보 계획 필요
- USB Rust 바인딩, debugfs/sg_table/iov_iter Rust 추상화 확대 추적

#### 13. BPF 서명 프로그램 — Android eBPF 보안
- BPF 프로그램 암호화 서명 인프라
- Android의 eBPF 네트워크 정책(netd, trafficcontroller)에 서명 검증 적용 가능성
- 비특권 BPF 로딩 경로는 앱 레벨 eBPF 활용 가능성 시사

#### 14. io_uring mixed-size CQE / multishot
- io_uring 기능 확장 지속
- Android에서의 io_uring 채택이 확대될 경우 비동기 I/O 성능에 영향

---

## 드라이버 & HAL 영향

### GPU / 디스플레이
- **Tyr Mali GPU**: CSF 기반 Mali GPU의 Rust 드라이버. Android Mali GPU HAL에 직접 영향은 없으나 장기 전략적 모니터링 필요
- **Rockchip NPU 드라이버**: RK3588 NPU 업스트림으로 NNAPI/Neural Networks HAL 통합 가능성
- **AMDGPU 체크포인트/복원**: GPU 컴퓨팅 워크로드의 안정성 향상
- **RK3588 DPTX/CSI-2**: 디스플레이 출력 및 카메라 인터페이스 메인라인 지원 확대

### 카메라
- **RK3588 MIPI CSI-2 DPHY**: 카메라 센서 인터페이스 메인라인 지원
- **eUSB2V2 웹 카메라**: 임베디드 USB 카메라 지원

### 오디오
- **Sony DualSense 오디오 잭**: Bluetooth 컨트롤러 오디오 지원
- **TASCAM USB 오디오**: USB 오디오 클래스 드라이버 확장

### 연결성
- **RP1 이더넷**: RPi5 이더넷 메인라인 지원
- **Qualcomm PPE**: 패킷 처리 엔진 드라이버
- **Microchip LAN969x**: SoC 네트워킹 지원

### 입력
- **Google 햅틱 터치패드**: Chromebook 햅틱 터치패드. 향후 Android 태블릿/2-in-1에 적용 가능성

---

## 전력 관리 & 성능

### CPU / 스케줄러
- **migrate_enable/disable 인라인화** — 마이그레이션 제어 오버헤드 감소. big.LITTLE 환경에서의 태스크 마이그레이션 효율 향상
- **CFS 스로틀 지연** — 사용자 공간 복귀 시점까지 스로틀 지연으로 커널 내 크리티컬 섹션 중단 방지
- **rseq 최적화** — per-CPU 연산 효율화. glibc/bionic 라이브러리 성능에 영향
- **Cgroup per-threadgroup resem** — 컨테이너 밀도 높은 환경의 확장성 향상. Android의 앱별 cgroup 관리에 유용
- **softirq PREEMPT_RT** — 실시간 지연 시간 개선. 오디오/카메라 등 실시간 워크로드에 유용

### 메모리 전력
- **Swap Table Phase I** — zram 스왑 효율 향상으로 메모리 압박 시 CPU 사용률 감소 → 전력 절감
- **OOM 피해자 해동** — suspend/resume 중 OOM 상황 회복력 향상. 모바일 전력 관리에 중요
- **THP PR_SET_THP_DISABLE 확장** — 앱별 THP 제어로 메모리/성능 트레이드오프 조정 가능

### 열 관리
- **Intel Panther Lake 전력 슬라이더** — x86 기반 Android/ChromeOS 디바이스의 열 관리

---

## 보안 & 인증

### CTS/VTS 영향
- **Rust Binder**: 활성화 시 CTS Binder 관련 테스트 전수 검증 필요. C Binder와의 행동 일치 확인
- **BPF 서명**: 향후 CTS에서 서명된 BPF 프로그램만 허용하는 정책 추가 가능성
- **KASAN write_only**: MTE 기반 보안 테스트(CTS MTE 모듈) 호환성 확인

### SELinux
- 6.18에서 주요 SELinux 변경 없음 (6.16의 디렉터리 접근 캐시/wildcard genfscon이 최신)

### 커널 보안 강화
- **KVM CET 가상화**: pKVM 환경에서의 제어 흐름 보호. ARM64 CET 대응 기술 모니터링
- **fanotify 퍼미션 워치독**: 보안 모니터링 데몬의 안정성 향상
- **비GPL 모듈 제한 강화**: 벤더 드라이버의 GPL 라이선스 준수 검토 필요

---

## 메모리 & 스토리지

### 메모리 관리
- **Slub Sheaves**: per-CPU 캐시로 할당 성능 향상. 앱 시작, 프로세스 생성 등에 영향
- **Swap Table Phase I**: zram 스왑 5-20% 처리량 향상. 저메모리 디바이스에서 가장 큰 체감 효과 기대
- **스왑 클러스터 스캐닝**: mTHP 기반 대형 페이지 할당 실패율 감소
- **memdesc_flags_t**: struct page 분리 프로젝트의 시작. 벤더 드라이버 API 호환성 영향
- **DAMON ARM32 LPAE**: 32비트 레거시 SoC에서 메모리 모니터링 가능
- **VMA per-vma lock /proc**: 프로파일링/디버깅 도구 성능 향상
- **Userfaultfd TLB 배칭**: userfaultfd 기반 메모리 관리(Android ART GC)의 성능 향상 가능

### 스토리지
- **F2FS**: 예약 노드, casefold lookup_mode, readahead 개선
- **exFAT**: 비트맵 로딩 최적화 → SD 카드 마운트 시간 단축
- **SquashFS**: SEEK_DATA/SEEK_HOLE → system 파티션 이미지 처리 효율화
- **FUSE**: COPY_FILE_RANGE_64 → /sdcard 대용량 파일 복사 효율 향상
- **dm-pcache**: CXL 메모리 캐시. 서버/자동차 플랫폼에 관련성
- **Ext4 슈퍼블록 ioctl**: 마운트된 상태에서 안전한 파라미터 변경

---

## Android GKI 호환성

### ABI 영향
- **memdesc_flags_t**: 새 타입 도입으로 struct page 관련 ABI 변경 가능성. GKI 모듈 ABI 호환성 검증 필요
- **Slub Sheaves**: SLUB 내부 구조 변경. kmalloc/kfree API는 동일하나 내부 동작 변화에 의한 성능 특성 변화 가능
- **Bcachefs 제거**: CONFIG_BCACHEFS_FS 제거. GKI defconfig에서 해당 옵션 사용 여부 확인

### 벤더 훅
- **Binder Rust**: C→Rust 전환 시 기존 벤더 훅 호환성 검증 필요. Rust Binder가 동일한 벤더 훅 인터페이스를 유지하는지 확인
- **스케줄러**: sched_ext 에러 상태 개선. 벤더 스케줄러 정책과의 상호작용 확인

### 모듈 인터페이스
- **Rust 추상화 확대**: USB, debugfs, sg_table, request_irq 등 새 Rust 추상화가 GKI 모듈 인터페이스에 미치는 영향 평가
- **DMA 매핑 API 전환**: 물리 주소 기반 API 이전. 벤더 드라이버의 DMA 매핑 코드 업데이트 필요 여부 확인

---

## 권장 액션 아이템

1. **[긴급]** BSP defconfig에서 CONFIG_BCACHEFS_FS 관련 옵션 제거. 6.18 빌드 검증
2. **[긴급]** Rust Binder 활성화 옵션 검토. 현 시점 C 드라이버 유지 권장, CVE-2025-68260 패치 적용 확인
3. **[높음]** memdesc_flags_t 도입에 따른 벤더 드라이버 struct page 직접 접근 코드 감사
4. **[높음]** 6.18 LTS 지정 확정 시 차기 GKI 베이스라인 전략 수립. Android 16 GKI 호환성 매트릭스 대비
5. **[높음]** DMA 매핑 물리 주소 API 전환에 따른 벤더 드라이버 업데이트 계획 수립
6. **[중간]** Swap Table Phase I + 스왑 클러스터 스캐닝의 zram 환경 성능 검증 (앱 스위칭, 저메모리 시나리오)
7. **[중간]** KASAN hw-tags write_only 모드의 프로덕션 빌드 적용 가능성 평가 (MTE 정책 연계)
8. **[중간]** F2FS 예약 노드/casefold lookup_mode의 Android 데이터 파티션 영향 검증
9. **[중간]** Cgroup per-threadgroup resem의 Android 컨테이너 환경 성능 검증
10. **[낮음]** Tyr Mali Rust GPU 드라이버 발전 추적. 장기 Android Mali 드라이버 전략 검토
11. **[낮음]** BPF 서명 프로그램 인프라 발전 추적. Android eBPF 보안 정책 로드맵 반영
12. **[낮음]** Rust 팀 역량 확보 계획. Rust가 커널 핵심 언어로 승격됨에 따라 벤더 드라이버 Rust 전환 대비

---

## 참고 자료

- [kernelnewbies.org — Linux 6.18](https://kernelnewbies.org/Linux_6.18)
- [Phoronix — Linux 6.18 Released](https://www.phoronix.com/news/Linux-6.18-Released)
- [Its FOSS — Linux 6.18 LTS](https://itsfoss.com/news/linux-kernel-6-18/)
- [Rust for Linux — Android Binder Driver](https://rust-for-linux.com/android-binder-driver)
- [Collabora — Tyr advances Rust in Linux](https://www.collabora.com/news-and-blog/news-and-events/kernel-618-tyr-advances-rust-in-linux.html)
- [CNX Software — Linux 6.18 ARM/RISC-V](https://www.cnx-software.com/2025/12/01/linux-6-18-release-main-changes-arm-risc-v-and-mips-architectures/)
