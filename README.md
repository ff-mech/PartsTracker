# Parts Tracker

**A fast, gap-aware SolidWorks part number tracker for Windows engineering teams.**

![Python](https://img.shields.io/badge/python-3.10%2B-blue?logo=python&logoColor=white)
![Platform](https://img.shields.io/badge/platform-Windows-0078d4?logo=windows&logoColor=white)
![License](https://img.shields.io/badge/license-MIT-green)
![Theme](https://img.shields.io/badge/theme-Catppuccin%20Mocha-cba6f7)

---

Parts Tracker is a PyQt6 desktop application that brings order to SolidWorks part file management across a shared network job folder structure. It integrates with the [Everything](https://www.voidtools.com/) file indexer for instant file discovery, parses Production Release Form (PRF) Excel files for job metadata, and maintains a local SQLite database so engineers always know which part numbers are in use, which are orphaned, and which gaps are available for reuse.

---

## Table of Contents

- [Features](#features)
- [Prerequisites](#prerequisites)
- [Quick Start](#quick-start)
- [Configuration](#configuration)
- [Tabs Reference](#tabs-reference)
  - [My Parts](#my-parts)
  - [All Parts](#all-parts)
  - [Jobs](#jobs)
  - [Next Numbers](#next-numbers)
  - [Orphans](#orphans)
  - [Archive](#archive)
  - [History](#history)
- [Part Numbering](#part-numbering)
- [Folder Structure](#folder-structure)
- [Data Storage](#data-storage)
- [Tech Stack](#tech-stack)
- [Contributing](#contributing)
- [License](#license)

---

## Features

- Scans SolidWorks part (`.sldprt`) and assembly (`.sldasm`) files across all job folders on the network drive
- Parses PRF Excel files for catalog number, enclosure size, and other job metadata
- Per-user prefix filtering so engineers only see their own parts by default
- Gap-aware next-number suggestions — reuses the lowest available number before issuing a new one
- Background gap scanning on startup with no UI freeze
- Live filesystem watcher that refreshes part numbers automatically when files change on disk
- Orphan detection: identifies part files that exist on disk but are not tracked in any job folder
- Archive classification: parts inside any `archive` folder are separated from active parts
- Full job history with scanned date and part count
- Searchable history log of all part additions over time
- Catppuccin Mocha dark theme throughout

---

## Prerequisites

| Requirement | Version / Notes |
|---|---|
| Python | 3.10 or later |
| [Everything](https://www.voidtools.com/) | Must have the HTTP Server enabled on port 8080 |
| Network drive | Job root must be mapped to `Z:\` |
| Windows | Windows 10 or later (Windows-only application) |

---

## Quick Start

1. **Clone or download the repository.**

   ```bat
   git clone <repository-url>
   cd parts-tracker
   ```

2. **Create and activate a virtual environment.**

   ```bat
   python -m venv venv
   venv\Scripts\activate
   ```

3. **Install dependencies.**

   ```bat
   pip install -r requirements.txt
   ```

4. **Enable Everything's HTTP Server.**

   In Everything: *Tools > Options > HTTP Server*, check **Enable HTTP Server**, set the port to `8080`, and click OK.

5. **Map the network drive.**

   Ensure the engineering job root is accessible at:

   ```
   Z:\FOXFAB_DATA\ENGINEERING\2 JOBS
   ```

6. **Launch the application.**

   ```bat
   python parts_tracker.py
   ```

   Alternatively, double-click `run.bat`.

---

## Configuration

### User Prefix

Each engineer is assigned a numeric prefix that maps to a range of part numbers. The application filters all views — My Parts, Next Numbers, Orphans, Archive — to only show parts whose numbers fall within that prefix range.

| Prefix | Range covered |
|---|---|
| `9` | 90000 – 99999 |
| `51` | 51000 – 51999 |

The prefix is set in the application settings. Any numeric prefix is supported; the range is derived automatically from the prefix length (a one-digit prefix covers a 10,000-number block, a two-digit prefix covers a 1,000-number block, and so on).

### Per-Category Prefix Overrides

Some categories use a different prefix from the user's default. For example, an engineer whose default prefix is `9` might use prefix `51` exclusively for category `240` (Copper) parts.

Per-category overrides can be configured in the application settings. When an override is active, the Next Numbers card and gap analysis for that category use the override prefix instead of the global one.

---

## Tabs Reference

### My Parts

Displays all part files whose numbers match the active user's prefix (and any per-category overrides). The list can be filtered by category code and by file type (`.sldprt` or `.sldasm`). This is the default view on startup.

---

### All Parts

Displays every part file across all users and job folders currently in the database. A search bar filters results by part number or file path in real time. Useful for checking whether a number is already in use before assigning it.

---

### Jobs

Lists every scanned job folder with metadata extracted from the associated PRF file:

- Job number and name
- Catalog number
- Enclosure size
- Total part count
- Date last scanned

Jobs whose folder path contains an archive indicator are highlighted in amber to distinguish them from active work.

---

### Next Numbers

A dashboard of cards — one per category — showing the next available part number for the active user's prefix. The system is gap-aware:

- If one or more numbers have been deleted or skipped, the lowest missing number is shown first with a `GAP` badge.
- Clicking the copy button copies that number to the clipboard and removes it from the gap list, automatically advancing to the next gap or to `latest + 1` when no gaps remain.

Below the cards, the **Gap Analysis** panel lists every missing number per category so engineers can review and reclaim them.

Gap scanning runs in a background thread at startup so the UI remains responsive. A live filesystem watcher monitors the job root and refreshes all cards automatically when files are added or removed on disk.

---

### Orphans

Lists part files on disk that match the active user's prefix but are **not** recorded in any scanned job folder and are **not** inside an `archive` folder. Files are discovered via the Everything HTTP API.

Orphaned parts typically indicate files that were created outside the standard job workflow or whose job folder has not yet been scanned. Reviewing this tab regularly helps keep the database accurate.

---

### Archive

Lists part files on disk that match the active user's prefix and **are** located inside a folder named `archive` (case-insensitive, anywhere in the path). Files are discovered via the Everything HTTP API.

This tab provides a convenient audit trail of retired parts without removing them from the filesystem.

---

### History

A chronological log of every part addition recorded by the application, searchable by part number. Each entry shows the part number, category, file type, job association, and the date it was first tracked. Useful for answering "when was this part number first used?" without leaving the application.

---

## Part Numbering

### Format

```
CAT-NNNNN.ext
```

- `CAT` — three-digit category code
- `NNNNN` — five-digit sequential number within the user's prefix range
- `ext` — `sldprt` or `sldasm`

**Example:** `200-90123.sldprt`

### Categories

| Code | Name |
|---|---|
| 003 | Top Level Assembly |
| 100 | Subassembly |
| 200 | Metal |
| 240 | Copper |
| 245 | Flexibar |
| 250 | Galvanized |
| 295 | Insulation Barrier |

### Prefix System

The five-digit number following the category code is drawn from a block assigned to each engineer. The prefix is the leading digit or digits of that number:

- Prefix `9` owns numbers `90000` through `99999`
- Prefix `51` owns numbers `51000` through `51999`

All views in the application respect this prefix so engineers do not see or accidentally reuse each other's numbers. Per-category overrides allow a single engineer to draw from a different block for a specific category (see [Configuration](#configuration)).

---

## Folder Structure

The application expects the following directory layout under the job root:

```
Z:\FOXFAB_DATA\ENGINEERING\2 JOBS\
  J15302 Garner Road\                  <- job folder (job number + name)
    200 Mech\
      J15302-01\
        201 CAD\                       <- parts folder (scanned for .sldprt / .sldasm)
          003-90123.sldasm
          200-90124.sldprt
          archive\                     <- archived parts live here
            200-90100.sldprt
        PRF - J15302-01.xlsx           <- Production Release Form (searched upward from parts folder)
```

Key rules:

- The scanner searches upward from the `201 CAD` folder to locate the PRF Excel file.
- Any file whose path contains a folder named `archive` (case-insensitive) is classified as archived, regardless of nesting depth.
- Parts outside a recognised job folder structure are classified as orphans.

---

## Data Storage

| Item | Location |
|---|---|
| SQLite database | `%APPDATA%\PartsTracker\parts.db` |
| Application settings | Stored within the database |

The database is created automatically on first launch. No manual setup is required. The `%APPDATA%` path resolves to `C:\Users\<username>\AppData\Roaming\PartsTracker\` on a standard Windows installation.

---

## Tech Stack

| Package | Version | Purpose |
|---|---|---|
| Python | 3.10+ | Runtime |
| PyQt6 | >= 6.4.0 | Desktop UI framework |
| openpyxl | >= 3.1.0 | Reading PRF Excel files |
| requests | >= 2.28.0 | Everything HTTP API queries |
| sqlite3 | stdlib | Local part and job database |

---

## Contributing

Contributions are welcome. To get started:

1. Fork the repository and create a feature branch from `main`.
2. Follow the existing code style — clear variable names, minimal inline comments where the code is self-explanatory, and docstrings on public functions.
3. Test your changes against a real or mock Everything instance before opening a pull request.
4. Open a pull request with a clear description of what was changed and why.

Bug reports and feature requests are best submitted as GitHub issues with enough detail to reproduce the problem or understand the use case.

---

## License

This project is licensed under the [MIT License](LICENSE).
