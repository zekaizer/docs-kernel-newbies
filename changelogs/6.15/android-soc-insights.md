# Linux 6.15 — Android SoC 개발 인사이트

> 릴리스: Linux 6.15 (2025-05-15)
> 작성일: 2026-02-10

---

## 개요

Linux 6.15는 Android SoC BSP 팀에게 메모리 관리 인프라 변화(per-VMA 락 재구현, per-CMA 락, defrag_mode), ARM SMMUv3 중첩 가상화 인프라 확장, F2FS Mount API 전환 착수, 그리고 Rust 기반 GPU 드라이버(NOVA) 초기 코드 머지가 핵심이다. 현재 Android GKI는 android16-6.12 기반이므로 6.15 변경사항은 차기 GKI 커널 선정 시 직접적 영향을 미칠 수 있다.

---

## BSP 영향도 분석

### 즉시 대응 필요 (High Impact)

- **Per-VMA 락 refcounting 재구현** — VMA 락 메커니즘이 전면 재작성됨. SoC 벤더의 커스텀 메모리 관리 코드(예: ION 대체 DMA-BUF heaps, GPU 메모리 매핑)가 VMA 락에 의존하는 경우 동작 변경 확인 필요
- **F2FS Mount API 전환 시작** — Android의 주요 데이터 파티션 파일시스템. 마운트 인터페이스 전환이 시작되었으므로, BSP의 init.rc 및 fstab 설정에서 F2FS 마운트 옵션 호환성 검증 필요
- **F2FS nat_bits 마운트 옵션화** — 기존 빌드 타임 설정이 런타임 마운트 옵션으로 변경. BSP fstab에서 명시적 설정이 필요할 수 있음
- **iommufd PASID attach/replace + ARM SMMUv3 중첩 MSI 매핑** — ARM SMMUv3 기반 SoC에서 디바이스 패스스루 및 가상화 시나리오에 직접 영향. Qualcomm/Samsung SoC의 SMMU 드라이버 업데이트 확인 필요
- **Training Solo (Spectre v2 변종) 완화** — ARM64 프로세서에도 적용될 수 있는 분기 예측기 공격 완화 패치. CTS 보안 요구사항 충족을 위해 패치 적용 검토

### 중기 검토 필요 (Medium Impact)

- **Per-CMA 락 도입** — CMA 영역별 개별 락으로 동시 할당 성능 향상. 모바일 SoC에서 GPU, 카메라, 디스플레이가 동시에 CMA를 사용하는 시나리오에서 경합 감소 → 프레임 드롭 개선 가능성
- **defrag_mode sysctl** — huge page 단편화 방지를 위한 새 sysctl. 모바일 환경에서 THP를 사용하는 경우 장시간 운용 시 단편화 문제 완화에 활용 가능
- **대형 folio batched unmap lazyfree** — MADV_FREE 기반 메모리 회수 효율 향상. Android의 LMKD(Low Memory Killer Daemon)와의 상호작용 테스트 권장
- **mseal 시스템 매핑 지원** — 보안 강화 매핑. SELinux 정책 및 seccomp 필터와의 호환성 확인
- **Perf wall-time 지연시간 프로파일링** — 모바일 앱의 프레임 지연, ANR 원인 분석에 유용한 새 프로파일링 도구. 성능 팀에 소개 가치 있음
- **Ext4 슈퍼블록 업데이트 간격 튜닝** — system 파티션의 불필요한 메타데이터 쓰기 감소 → eMMC/UFS 수명 연장에 기여
- **sched_ext per-NUMA idle cpumask** — big.LITTLE/DynamIQ 구성의 모바일 SoC에서 클러스터별 idle CPU 선택 최적화에 활용 가능성 (향후 EAS 통합 시)

### 참고 사항 (Low Impact / Watch)

- **Rust pin-init 독립 크레이트, hrtimer API, mm 바인딩** — Rust 커널 드라이버 생태계 성숙 중. NOVA GPU 드라이버 초기 코드가 머지되어, 향후 SoC GPU 드라이버의 Rust 전환 가능성 모니터링 필요
- **io_uring zero-copy 수신** — 서버 워크로드 중심이나, Android의 고성능 네트워킹(5G 모뎀 인터페이스) 시나리오에서 장기적 관심 대상
- **AMD INVLPGB** — x86 전용으로 ARM SoC에 직접 해당 없으나, TLB 무효화 최적화의 ARM 대응 작업(TLBI 브로드캐스트 최적화) 모니터링
- **fwctl 서브시스템** — 펌웨어 디버깅/설정 인터페이스 표준화. SoC 펌웨어(모뎀, NPU, DSP) 관리에 향후 적용 가능성
- **Bcachefs 기능 확장** — Android에서는 사용하지 않으나, 파일시스템 기술 트렌드로 참고
- **VFS idmapped mount 확장, fanotify 마운트 알림** — 컨테이너/멀티유저 환경에서의 마운트 관리 개선. Android의 Work Profile, 멀티유저 기능과 관련 가능성

---

## 드라이버 & HAL 영향

### GPU / 디스플레이
- **NOVA DRM 드라이버 초기 코드 머지** — Rust 기반 NVIDIA GPU 드라이버가 메인라인에 첫 진입. 현재 모바일 GPU(Adreno, Mali, Xclipse)에 직접 영향은 없으나, DRM 서브시스템의 Rust 추상화 확대가 향후 모바일 GPU 드라이버 구조에 영향을 줄 수 있음
- **DRM 서브시스템에 Rust 추상화 추가** — 6.16에서 본격 확장 예정. 장기적으로 GPU HAL과의 인터페이스 변화 가능성

### SoC 플랫폼
- **Exynos ACPM 프로토콜 드라이버 추가** — Google GS101(Tensor) SoC용 전력 관리 펌웨어 통신 드라이버. Pixel 6 Pro(Raven) 보드 지원 추가
- **Exynos2200, Exynos7870 DT 바인딩** — PMU, ChipID, SYSREG 바인딩 추가
- **Qualcomm SM8750 UFS PHY, 모뎀 remoteproc** — 차세대 Qualcomm SoC의 UFS 및 모뎀 관리 드라이버
- **Qualcomm iris 비디오 디코더 드라이버** — 미디어 서브시스템에 새 비디오 디코더 추가
- **Exynos Auto v920 UFS, CPU 캐시 정보** — 차량용 SoC 지원 확대

### 카메라 / 센서 / 오디오
- 6.15에서 주요 카메라/센서 프레임워크 변화 없음. 드라이버 레벨 버그 수정 및 DT 바인딩 업데이트 위주

---

## 전력 관리 & 성능

- **Per-CMA 락** — GPU/카메라/디스플레이의 동시 CMA 할당 시 락 경합 감소. 카메라 프리뷰 + GPU 렌더링 동시 수행 시나리오에서 프레임 안정성 개선 가능
- **defrag_mode sysctl** — 장시간 운용 시 huge page 가용성 유지. 게이밍 및 멀티미디어 워크로드에서 THP 활용 시 성능 안정성 향상
- **Batched unmap lazyfree** — 대형 folio의 효율적 회수. 앱 전환 시 메모리 해제 속도 개선 가능
- **Perf wall-time 프로파일링** — jank/ANR 원인 분석 시 off-CPU 시간(락 대기, I/O 대기, 스케줄링 지연) 식별 가능. systrace/perfetto를 보완하는 새 분석 도구
- **sched_ext 이벤트 카운터 및 per-NUMA cpumask** — big.LITTLE 구조에서의 스케줄러 동작 관측성 향상. 커스텀 EAS 정책 개발 시 디버깅 용이

---

## 보안 & 인증

- **Training Solo (Spectre v2 변종) 패치** — CPU 취약점 완화. Android Security Bulletin 및 CTS 보안 패치 레벨 요구사항 충족을 위해 백포트 검토
- **io_uring 보안 훅** — SELinux가 io_uring 연산에 대해 정책 제어 가능. Android의 SELinux 정책(neverallow 규칙 등)과의 호환성 확인
- **UBSAN overflow 패턴 제외** — 커널 빌드 시 UBSAN 활성화 환경에서 false positive 감소. VTS 커널 테스트 통과에 긍정적
- **iommufd PASID + vIOMMU** — 디바이스 격리 강화. 향후 Android Virtualization Framework(AVF)의 pKVM과 연계 시 디바이스 보안 모델에 영향
- **Runtime Verification 스케줄러 모니터** — 프로덕션 환경에서 스케줄러 동작 정확성 검증 가능. 자동차/산업용 Android 파생 제품의 안전성 인증에 활용 가능성

---

## 메모리 & 스토리지

### 메모리
- **Per-VMA 락 refcounting 재구현** — mmap_lock 병목 해소의 핵심 작업. 멀티스레드 앱(게임, 브라우저)의 mmap/munmap 경합 감소
- **Per-CMA 락** — GPU/카메라/디스플레이의 CMA 동시 사용 시 성능 향상
- **mseal 지원 확장** — guard region이 파일 기반 매핑으로 확대되어 보안 강화
- **DAMON 자동 튜닝, 필터 개선** — proactive reclaim의 정확도 향상. Android의 memory pressure 대응(LMKD, kswapd 튜닝)에 참고

### 스토리지
- **F2FS Mount API 전환 시작** — Android 데이터 파티션의 핵심 변경. 마운트 시 동작 변화 가능성 테스트 필요
- **F2FS nat_bits 마운트 옵션화** — 런타임 제어 가능. fstab에서 명시적 설정 필요 여부 확인
- **F2FS folio 전환 계속** — 내부 구조 변화로 성능 특성 변화 가능. 벤치마크 검증 권장
- **Ext4 dentry 검색 최적화, 슈퍼블록 업데이트 간격 조정** — system/vendor 파티션 성능 및 수명에 긍정적
- **Qualcomm SM8750 UFS PHY 드라이버** — 차세대 Qualcomm SoC용 UFS 스토리지 지원

---

## Android GKI 호환성

### 현재 상태
- **Android 16 GKI**: android16-6.12 기반 (2025년 활성)
- **Android 15 GKI**: android15-6.6 기반 (2025-12-01 respin cutoff, 2026-06-01 deprecation)
- **Linux 6.15는 현재 GKI 커널이 아님** — LTS로 지정되지 않았으며, 6.15.11을 마지막으로 EOL

### GKI 관점에서의 6.15 변경사항 의미
- 6.15의 변경사항은 차기 Android GKI 커널 선정 시 반영될 수 있음 (6.18 또는 그 이후가 LTS로 지정될 경우)
- Per-VMA 락, per-CMA 락, F2FS Mount API 전환 등은 android-mainline에 이미 반영되어 있으므로, 향후 GKI 커널로의 전환 시 자동 포함
- **KMI 참고**: GKI 커널 버전 간 KMI 호환성이 없으므로, 6.12 → 차기 GKI 전환 시 모든 벤더 모듈 재빌드 필수

### Vendor Hook 관련
- 6.15에서 새로운 vendor hook 추가 여부는 android-mainline ACK 브랜치에서 별도 확인 필요
- Per-VMA 락 재구현으로 기존 VMA 관련 vendor hook의 동작 변경 가능성 있음. ACK 패치 확인 권장

---

## 권장 액션 아이템

1. **[High]** F2FS Mount API 전환 및 nat_bits 옵션화에 대해 BSP fstab/init.rc 호환성 테스트 수행
2. **[High]** Per-VMA 락 재구현이 벤더 커스텀 메모리 관리 코드(DMA-BUF heaps, GPU mmap)에 미치는 영향 검증
3. **[High]** iommufd PASID/SMMUv3 중첩 MSI 변경사항이 SoC SMMU 드라이버에 미치는 영향 확인
4. **[High]** Training Solo Spectre v2 완화 패치의 ARM64 적용 여부 확인 및 보안 패치 레벨 반영
5. **[Medium]** Per-CMA 락의 GPU/카메라 동시 CMA 할당 시나리오 벤치마크 수행
6. **[Medium]** Perf wall-time 프로파일링 도구를 성능 팀에 소개 및 jank 분석 워크플로우에 통합 검토
7. **[Medium]** defrag_mode sysctl의 모바일 워크로드 효과 평가 (게이밍, 카메라 프리뷰)
8. **[Medium]** Ext4 슈퍼블록 업데이트 간격 조정으로 system 파티션 eMMC/UFS 쓰기 부하 감소 검토
9. **[Low]** Rust 커널 드라이버 생태계(NOVA, DRM 추상화) 모니터링 — 차세대 SoC GPU 드라이버 전략에 참고
10. **[Low]** sched_ext 이벤트 카운터를 활용한 big.LITTLE 스케줄링 분석 도구 가능성 탐색

---

## 참고 자료

- [kernelnewbies.org — Linux 6.15](https://kernelnewbies.org/Linux_6.15)
- [CNX Software — Linux 6.15 ARM, RISC-V, MIPS](https://www.cnx-software.com/2025/05/26/linux-6-15-release-main-changes-arm-risc-v-and-mips-architectures/)
- [Android — Generic Kernel Image (GKI) project](https://source.android.com/docs/core/architecture/kernel/generic-kernel-image)
- [Android — android16-6.12 release builds](https://source.android.com/docs/core/architecture/kernel/gki-android16-6_12-release-builds)
- [Android — Develop kernel code for GKI](https://source.android.com/docs/core/architecture/kernel/kernel-code)
- [Android — Stable KMI](https://source.android.com/docs/core/architecture/kernel/stable-kmi)
- [kernel.org — IOMMUFD Documentation](https://docs.kernel.org/userspace-api/iommufd.html)
- [Phoronix — Sched_Ext Linux 6.15](https://www.phoronix.com/news/Linux-6.15-Sched-Ext)
- [Phoronix — Many Rust Changes Linux 6.15](https://www.phoronix.com/news/Linux-6.15-Rust)
