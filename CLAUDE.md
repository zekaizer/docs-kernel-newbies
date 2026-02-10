# CLAUDE.md — docs-kernel-newbies

## Project Overview

This repository tracks, analyzes, and documents Linux kernel version changes sourced from [kernelnewbies.org](https://kernelnewbies.org). It serves as a structured reference for understanding kernel evolution across releases, with a focus on the 6.x series (6.13–6.19+).

The primary artifact is `kernel-changelog-report.md`, a comprehensive markdown report covering release timelines, per-version feature breakdowns, and cross-version trend analysis.

## Repository Structure

```
docs-kernel-newbies/
├── CLAUDE.md                    # This file — project guide for AI assistants
├── kernel-changelog-report.md   # Main report: kernel 6.13–6.19 changes & trends
└── (future additions below)
```

## Data Source

All kernel version data comes from **kernelnewbies.org**:
- Version index: https://kernelnewbies.org/LinuxVersions
- Per-version pages: `https://kernelnewbies.org/Linux_6.XX` (e.g., Linux_6.17)
- The site follows a consistent structure: prominent features, then subsystem-by-subsystem breakdown (filesystems, memory management, networking, security, drivers, architecture, tracing/BPF, Rust support)

## Key Conventions

### Report Format
- Each kernel version section starts with **Headline Features** (the most impactful user/developer-facing changes)
- Subsystem breakdowns follow: Filesystems, Memory Management, Networking, Security, Scheduler, io_uring, Tracing/BPF, Rust Support, Drivers, Architecture
- A **Cross-Version Trend Analysis** section at the end tracks multi-release trajectories for key subsystems
- Tables are used for release timelines and LTS version tracking

### Writing Style
- Concise, factual descriptions — no hype or marketing language
- Performance numbers included when available (e.g., "+15% write perf", "37% reduction")
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

1. Fetch the kernelnewbies page: `https://kernelnewbies.org/Linux_6.XX`
2. Extract changes following the subsystem categories listed above
3. Identify the top 5-7 headline features
4. Add a new section in `kernel-changelog-report.md` in chronological order
5. Update the Release Timeline table
6. Update the Cross-Version Trend Analysis section with any new data points
7. Update the LTS table if the new version is designated LTS
8. Update this CLAUDE.md if new subsystem categories or trends emerge

## Important Context for AI Assistants

- The ~63-day release cycle means a new kernel version roughly every 2 months
- kernelnewbies.org pages may not be immediately available on release day; check back if a page returns 404
- Performance claims should always cite the source (kernelnewbies or kernel commit messages)
- When in doubt about a feature's significance, include it — the report aims to be comprehensive
- The report is structured for both sequential reading and quick reference lookup
- LTS versions (6.1, 6.6, 6.12) are particularly important as they receive long-term maintenance
- Korean-language summaries or translations may be requested — maintain the same technical precision

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
