# docs-kernel-newbies

Linux 커널 버전별 변경사항 분석 및 Android SoC 개발 인사이트를 정리하는 문서 저장소입니다.

## 구조

```
changelogs/<version>/
  overview.md              # 버전 개요
  filesystems.md           # 서브시스템별 상세 분석
  memory-management.md
  ...
  android-soc-insights.md  # Android SoC BSP 영향도 분석

trends/                    # 크로스 버전 트렌드 분석
references/                # 참고 자료
```

## 커버리지

| 버전 | 릴리스 | 상태 |
|------|--------|------|
| [6.15](changelogs/6.15/) | 2025-05-15 | 완료 |
| [6.16](changelogs/6.16/) | 2025-07-27 | 완료 |

## 소스

- [kernelnewbies.org](https://kernelnewbies.org/LinuxVersions)
- [LWN.net](https://lwn.net)
- [kernel.org](https://kernel.org)
