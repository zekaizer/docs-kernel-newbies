# CLAUDE.md — docs-kernel-newbies

## Project Overview

This repository tracks, analyzes, and documents Linux kernel version changes sourced from [kernelnewbies.org](https://kernelnewbies.org). It serves as a structured reference for understanding kernel evolution across releases, with a focus on the 6.x series (6.13–6.19+).

The primary artifact is `changelogs/kernel-changelog-6.13-6.19.md`, a comprehensive markdown report covering release timelines, per-version feature breakdowns, and cross-version trend analysis.

## Repository Structure

```
docs-kernel-newbies/
├── CLAUDE.md                          # Project guide for AI assistants (English)
├── changelogs/                        # Per-version and multi-version changelog reports
│   ├── kernel-changelog-6.13-6.19.md  #   Combined report: 6.13–6.19 (Korean)
│   ├── kernel-changelog-6.20.md       #   Single version report example
│   └── ...
├── trends/                            # Cross-version trend analysis reports
│   ├── kernel-trend-analysis-6.13-6.19.md
│   └── ...
└── references/                        # Supplementary reference material
    ├── subsystem-overview.md           #   Kernel subsystem quick reference
    └── ...
```

Directory purposes:
- **`changelogs/`** — Version-specific changelog reports (single or multi-version)
- **`trends/`** — Cross-version trend analysis and longitudinal studies
- **`references/`** — Supplementary material: subsystem overviews, glossaries, LTS tracking

## Data Sources

Primary and supplementary sources for kernel analysis:

| Source | URL | Usage |
|--------|-----|-------|
| kernelnewbies.org | https://kernelnewbies.org/LinuxVersions | Primary: per-version feature summaries |
| Per-version pages | `https://kernelnewbies.org/Linux_6.XX` | Primary: subsystem-level breakdown |
| kernel.org | https://kernel.org | Release dates, tarball info, LTS status |
| LKML (mailing list) | https://lkml.org | Supplementary: patch discussions, design rationale |
| LWN.net | https://lwn.net | Supplementary: in-depth feature analysis articles |
| Git commit logs | `git log` on kernel source | Supplementary: commit messages, diffstats |

## Research Requirements

**All reports must be based on thorough, in-depth research.** Superficial summaries are not acceptable.

### Research Methodology

When creating or updating a report, follow this research process:

1. **Primary source collection** — Fetch the kernelnewbies.org page for the target version and extract all listed changes per subsystem
2. **Cross-reference verification** — Verify key claims (performance numbers, new syscalls, API changes) against at least one additional source (LWN.net, LKML, kernel commit logs)
3. **Contextual analysis** — For each headline feature, understand *why* it was introduced (what problem it solves, what workloads benefit)
4. **Quantitative data** — Actively seek out benchmark results, performance deltas, and measurable impacts; do not omit available numbers
5. **Trend continuity** — Check how each change relates to ongoing multi-version trends (e.g., folio adoption, Rust expansion) and update trend tracking accordingly

### Research Depth Expectations

| Report Type | Minimum Research Depth |
|-------------|----------------------|
| Single-version changelog | Full kernelnewbies page + LWN feature articles for headline items |
| Multi-version changelog | All individual version sources + cross-version diff analysis |
| Trend analysis | All version sources + LKML discussions + commit history for the subsystem |

### Quality Criteria
- Every headline feature must include a one-sentence explanation of its purpose/impact
- Performance claims must cite a source (kernelnewbies, LWN, or commit message)
- New syscalls and APIs must include the call signature or usage context
- Omissions are acceptable only when a subsystem has genuinely zero changes for a release
- When information is uncertain or unverifiable, mark it explicitly (e.g., "확인 필요")

## Key Conventions

### Language Policy
- **CLAUDE.md** and other project configuration files: **English**
- **All generated reports** (changelog, trend analysis): **Korean (한글)**
- Kernel technical terms (e.g., folio, io_uring, sched_ext) remain in English as-is
- Proper nouns, function names, syscall names, and commit messages stay in English
- Descriptive text, analysis, and summaries must be written in Korean
- Example: "Btrfs: 비동기 체크섬 계산으로 쓰기 성능 15% 향상"

### File Naming Convention

All report files follow this naming pattern:

```
kernel-changelog-<version-range>.md
```

| Type | Directory | Filename Example | Description |
|------|-----------|------------------|-------------|
| Multi-version report | `changelogs/` | `kernel-changelog-6.13-6.19.md` | Combined changelog across versions |
| Single-version report | `changelogs/` | `kernel-changelog-6.20.md` | Individual version analysis |
| Trend analysis | `trends/` | `kernel-trend-analysis-6.13-6.19.md` | Cross-version trend report |
| Reference material | `references/` | `subsystem-overview.md` | Supplementary docs |

Rules:
- Lowercase only, hyphen (`-`) as separator (no underscores `_`)
- Version ranges use hyphen: `6.13-6.19`
- Extension is always `.md` (Markdown)

### Report Structure (Unified Template)

All kernel changelog reports must follow this structure (written in Korean):

```markdown
# Linux 커널 버전 변경사항 보고서

> 출처: kernelnewbies.org
> 대상 버전: Linux X.XX – X.XX
> 작성일: YYYY-MM-DD

---

## 릴리스 타임라인
(Table: 버전 | 릴리스 날짜 | 개발 주기)

---

## Linux X.XX

### 주요 기능 (Headline Features)
- Top 5–7 key changes

### 파일시스템
### 메모리 관리
### io_uring
### 스케줄러
### 네트워킹
### 보안
### 트레이싱 & BPF
### Rust 지원
### 가상화
### 블록 레이어
### 아키텍처
### 드라이버
### 시스콜

(Omit subsystem sections with no changes for that version)

---

## 교차 버전 트렌드 분석

### 1. Trend name
(Per-version tracking)

---

## LTS 버전 현황
(Table: 버전 | LTS 상태)

---

## 요약 통계
```

Key rules:
- Each version section starts with **주요 기능 (Headline Features)**
- Subsystem order: 파일시스템 → 메모리 관리 → io_uring → 스케줄러 → 네트워킹 → 보안 → 트레이싱 & BPF → Rust 지원 → 가상화 → 블록 레이어 → 아키텍처 → 드라이버 → 시스콜
- Omit subsystem sections entirely if no changes exist for that release
- Every report must end with **교차 버전 트렌드 분석** (Cross-Version Trend Analysis)
- Release timeline and LTS status use tables

### Writing Style
- Concise, factual descriptions — no hype or marketing language
- Performance numbers included when available (e.g., "쓰기 성능 15% 향상", "37% 감소")
- Technical terms used without over-explanation (target audience: kernel developers and enthusiasts)
- Bullet points preferred over prose for scanability

### Version Coverage
- Current coverage: Linux 6.13 (Jan 2025) through 6.19 (late 2025)
- New versions should be appended as they are released (~every 63 days)
- LTS versions tracked separately (currently 6.1, 6.6, 6.12)

## Tracked Kernel Subsystem Categories

When analyzing a new kernel release, extract changes in these categories:

1. **Headline Features** — Top 5-7 most significant user/developer-visible changes
2. **Filesystems** — Btrfs, Ext4, XFS, NFS, F2FS, Bcachefs, FUSE, EROFS, OverlayFS
3. **Memory Management** — Folios, VMA locking, DAMON, swap, reclaim, huge pages, page tables
4. **io_uring** — New operations, zero-copy, completion queue changes, polling
5. **Scheduler** — Preemption modes, sched_ext, EEVDF, NUMA balancing, priority inheritance
6. **Networking** — TCP/UDP optimizations, zero-copy, congestion control, driver APIs
7. **Security** — Confidential computing (TDX, CCA), SELinux, signing, control flow
8. **Tracing & BPF** — BPF verifier, kfuncs, perf enhancements, ftrace, runtime verification
9. **Rust Support** — New abstractions, driver bindings, standalone crates
10. **Virtualization** — KVM, virtio, live update, hypervisor changes
11. **Block Layer** — Device mapper, RAID, NVMe, io scheduling
12. **Architecture** — x86, ARM64, RISC-V specific changes, new instructions
13. **Drivers** — GPU, storage, networking, USB, audio hardware support
14. **Syscalls** — New or modified system calls

## Cross-Version Trends to Track

These are the major multi-release trajectories identified so far:

| Trend | Versions Active | Status |
|-------|-----------------|--------|
| Rust language integration | 6.13–6.19+ | Expanding every release |
| io_uring capabilities | 6.13–6.19+ | New ops each release |
| Large folio adoption | 6.13–6.19+ | Spreading across subsystems |
| Per-VMA locking | 6.15–6.18+ | Maturing |
| Confidential computing | 6.13–6.19+ | ARM CCA → Intel TDX → PCIe encryption |
| struct page/slab/folio separation | 6.18+ | Early stages |
| Bcachefs maturation | 6.15–6.16+ | Rapid feature additions |
| Atomic write support | 6.13–6.16+ | XFS → Ext4 → broader |
| Scheduler evolution (EEVDF) | 6.13–6.19+ | Continuous refinement |
| BPF signing & unprivileged loading | 6.18+ | Infrastructure in place |

## Workflow for Adding a New Kernel Version

1. **Research** — Fetch the kernelnewbies page (`https://kernelnewbies.org/Linux_6.XX`) and supplementary sources (LWN, LKML) for headline features
2. **Extract** — Collect changes following the subsystem categories listed above
3. **Verify** — Cross-reference performance claims and key features against additional sources
4. **Analyze** — Identify the top 5-7 headline features with context on purpose and impact
5. **Write** — Create the report **in Korean** following the unified report template
6. **Place** — Save to the correct directory (`changelogs/kernel-changelog-6.XX.md`)
7. **Update timeline** — Update the Release Timeline table
8. **Update trends** — Update `trends/` reports with any new data points
9. **Update LTS** — Update the LTS table if the new version is designated LTS
10. **Update CLAUDE.md** — Add new subsystem categories or trends if they emerge

## Important Context for AI Assistants

- The ~63-day release cycle means a new kernel version roughly every 2 months
- kernelnewbies.org pages may not be immediately available on release day; check back if a page returns 404
- Performance claims should always cite the source (kernelnewbies or kernel commit messages)
- When in doubt about a feature's significance, include it — the report aims to be comprehensive
- The report is structured for both sequential reading and quick reference lookup
- LTS versions (6.1, 6.6, 6.12) are particularly important as they receive long-term maintenance
- **All reports must be written in Korean** — maintain the same technical precision as English
- **CLAUDE.md stays in English** — it is a project configuration file, not a report
- Always follow the file naming convention and unified report template

## Build / Tooling

This is a pure documentation repository. No build system, tests, or CI/CD pipelines exist. The workflow is:
- Edit markdown files directly
- Commit with clear messages describing what kernel versions or sections were updated
- Push to the appropriate branch

## Commit Message Convention

```
docs: add Linux 6.XX changelog analysis
docs: update cross-version trend analysis for 6.XX
docs: fix typo in 6.XX filesystem section
```

Prefix all commits with `docs:` since this is a documentation-only repository.
