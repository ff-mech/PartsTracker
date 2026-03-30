# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

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

[Unreleased]: https://github.com/ffLambert/parts-tracker/compare/v1.4.0...HEAD
[1.4.0]: https://github.com/ffLambert/parts-tracker/compare/v1.3.0...v1.4.0
[1.3.0]: https://github.com/ffLambert/parts-tracker/compare/v1.2.0...v1.3.0
[1.2.0]: https://github.com/ffLambert/parts-tracker/compare/v1.1.0...v1.2.0
[1.1.0]: https://github.com/ffLambert/parts-tracker/compare/v1.0.0...v1.1.0
[1.0.0]: https://github.com/ffLambert/parts-tracker/releases/tag/v1.0.0
