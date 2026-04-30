# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

## [1.5.1] - 2026-04-30

### Fixed

- **Orphans / Archive tab robustness** — three filters added to drop noise that was previously polluting the Orphans tab. Empirical impact on the 9-prefix dataset: 65 → 15 surviving entries (77% reduction; the 50 dropped were SolidWorks lock files and vendor-library hardware whose filenames embedded a FoxFab-shaped substring).
  1. **Filename pattern** — only files whose names start with `<CAT>-<NNNNN>` are considered. Filters out `~$*.sldprt` / `~$*.sldasm` SolidWorks lock files and vendor parts like `SS Cam Adaptor_200-9598.00-00000.SLDPRT` where the FoxFab-shaped string is just an embedded substring.
  2. **Folder scope** — files must live inside a `\201 CAD\` path segment (case-insensitive). Skips vendor subfolders (`\Dirak\`, hardware-library directories, etc.) inside a job tree.
  3. **Folder exclusions** — paths containing `\Backup\` or `\old\` segments are skipped. `\Archive\` continues to be excluded for the Orphans tab and required for the Archive tab.

### Added

- New `ORPHAN_PART_RE` regex (permissive: `^\d{3}-\d{5}.*\.(sldprt|sldasm)$`) used by the Orphan / Archive scanner. Unlike the strict `PART_RE`, it accepts rev-suffix files (`200-90123-rA.sldprt`) and numbered-copy files (`200-90123 (1).sldprt`), which are real parts but were previously rejected.
- Orphan / Archive list entries now display the full filename stem (`200-90123-rA`) rather than just `<CAT>-<NNNNN>`, preserving rev and combined-part info on screen.

## [1.5.0] - 2026-04-30

### Added

- **Duplicate Parts panel** in the standalone PyQt6 Parts Tracker (previously Tk-hub-only). Sits below Gap Analysis inside the Next Numbers tab — matches the layout of the hub-embedded version. Lists part numbers that appear at more than one path under the active user's prefix; archive folders excluded; double-click a path to open in Explorer.
- **Crash logger** at `%APPDATA%\PartsTracker\crash.log`. Captures three failure channels in one file: (a) uncaught Python exceptions via `sys.excepthook`, (b) Qt-level fatal / critical / warning messages via `qInstallMessageHandler` (which previously caused silent `abort()` on certain Qt asserts), (c) wrapped slot exceptions in `_on_gap_scan_done`, `_copy_next`, `_tab_changed`, and `_safe_refresh`. PyQt 6.1+ aborts the app on uncaught slot exceptions by default — the excepthook keeps the window alive instead.
- Trace breadcrumbs in the log: startup banner with Python version, `_tab_changed` events with target tab name, and `_start_gap_scan` / `_on_gap_scan_done` markers. Enough to correlate an intermittent crash to the operation that triggered it without spamming the file.
- New `Database.get_duplicate_parts()` method on the PyQt6 standalone, ported verbatim from `parts_tracker_tk.py`. Includes the read-time `Path.exists()` filter so resolved duplicates (one of the two copies deleted or moved) drop out of the report without needing a full DB rescan.

### Fixed

- **Gap finder + collision check false-negative bug.** Both `find_gaps_via_everything()` and `is_part_number_taken()` had hard-coded their Everything HTTP queries to `path:"Z:\FOXFAB_DATA\ENGINEERING\2 JOBS"`. Result: any part assigned in `DESIGNERS\...`, `MODEL LIBRARY\`, `0 PRODUCTS\`, `For Vikram\Demo Unit\CAD\`, or any designer scratch area was invisible to both — gap analysis reported it as missing, and the click-time collision check returned "free", risking a Copy that clobbered another engineer's file. New constant `ENG_ROOT = r"Z:\FOXFAB_DATA\ENGINEERING"` widens the search scope. `find_gaps_via_everything()` now buckets each Everything hit twice — `primary_present` (file is under JOBS_ROOT, drives Latest/Next so a stray number elsewhere doesn't yank Latest) and `broader_present` (anywhere under ENG_ROOT, used to verify gaps). A candidate gap is reported only if absent from `broader_present`. `is_part_number_taken()` switched to ENG_ROOT outright. `find_003_folders()` left scoped to JOBS_ROOT — it anchors on real 003 job assemblies and is unaffected.
- Mirrored across `parts_tracker.py` (PyQt6 standalone) and `parts_tracker_tk.py` (Tk-embedded in the hub).

### Internal

- `_safe_refresh` (the 5-second poll tick) was silencing exceptions with `pass`. Now logs to `crash.log` so transient issues are surfaced instead of hidden.

## [1.4.0] - 2026-03-30

### Added
- New **Archive tab** that displays part files found inside `archive` folders, scanned via Everything and styled in amber.

### Changed
- Orphan tab now excludes files located inside `archive` folders to avoid false positives.

## [1.3.0] - 2026-03-26

### Added
- Gap-aware Next Numbers that account for gaps in the existing number sequence.
- Background gap scanning to detect missing part numbers without blocking the UI.
- Updated README to document the new gap-aware behaviour.

## [1.2.0] - 2026-03-26

### Added
- Support for multi-config jobs, allowing multiple scan configurations to run together.
- Archive detection to identify files stored in archive locations.
- Gap checker to surface missing numbers within a part category.
- Per-category prefix overrides for more granular control over number formatting.

## [1.1.0] - 2026-03-25

### Added
- Live Next Numbers update driven by a file system watcher.
- Disk verification step to confirm that suggested next numbers do not already exist on disk.

## [1.0.0] - 2026-03-25

### Added
- Initial release of Parts Tracker.
- Core file logging and parts tracking functionality.

[Unreleased]: https://github.com/ff-mech/PartsTracker/compare/v1.5.1...HEAD
[1.5.1]: https://github.com/ff-mech/PartsTracker/compare/v1.5.0...v1.5.1
[1.5.0]: https://github.com/ff-mech/PartsTracker/compare/v1.4.0...v1.5.0
[1.4.0]: https://github.com/ff-mech/PartsTracker/compare/v1.3.0...v1.4.0
[1.3.0]: https://github.com/ff-mech/PartsTracker/compare/v1.2.0...v1.3.0
[1.2.0]: https://github.com/ff-mech/PartsTracker/compare/v1.1.0...v1.2.0
[1.1.0]: https://github.com/ff-mech/PartsTracker/compare/v1.0.0...v1.1.0
[1.0.0]: https://github.com/ff-mech/PartsTracker/releases/tag/v1.0.0
