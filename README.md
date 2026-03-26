# Parts Tracker

> Track SolidWorks parts across jobs — instantly.

A lightweight desktop app that combines **Everything's HTTP API** with **PRF Excel data** to give you a searchable, filterable view of every `.sldprt` and `.sldasm` file across your entire job structure — with smart part-number management built in.

---

## Features

- **Instant search** across all indexed SolidWorks parts and jobs
- **PRF integration** — auto-reads catalog number and enclosure size from Production Release Forms
- **Part categories** — Metal, Copper, Flexibar, Galvanized, Subassembly, Insulation Barrier
- **Per-user filtering** — scope results to your own part number prefix
- **Next Numbers tab** — see your next available part number per category at a glance
- **Gap detection** — automatically finds missing part numbers in your range using Everything, so gaps get filled before new numbers are assigned
- **Copy to fill gaps** — copying a gap number removes it from the list and advances to the next one
- **Background gap scanning** — gaps are checked automatically on startup with no UI freeze
- **Local SQLite database** — fast queries, zero server needed
- **Dark UI** — clean Catppuccin Mocha theme built with PyQt6

---

## Requirements

| Requirement | Details |
|---|---|
| Python | 3.10 or later |
| [Everything](https://www.voidtools.com/) | HTTP Server enabled on port `8080` |
| Network drive | `Z:\FOXFAB_DATA\ENGINEERING\2 JOBS` mapped |

---

## Setup

**1. Create and activate a virtual environment**

```bash
python -m venv venv
venv\Scripts\activate
```

**2. Install dependencies**

```bash
pip install -r requirements.txt
```

**3. Enable Everything HTTP Server**

```
Everything → Tools → Options → HTTP Server → Enable (port 8080)
```

**4. Run**

```bash
python parts_tracker.py
```

Or double-click `run.bat`.

> **Note:** Always activate the virtual environment before running (`venv\Scripts\activate`).

---

## Tabs

### My Parts
Shows all parts belonging to your user prefix. Filterable by category and file type.

### All Parts
Full view of every scanned part across all users and jobs. Searchable and sortable.

### Jobs
Lists every scanned job with its PRF data (catalog number, enclosure size, part count). Supports filtering by size and catalog.

### Next Numbers
Your part-number dashboard. For each category:

- **Latest** — the highest part number currently on disk for your prefix
- **Next** — your next number to use, with gap-awareness:
  - If any gaps exist in your range, **Next shows the lowest missing number first** (marked with a `GAP` badge) so gaps are filled before new numbers are assigned
  - Once a gap is copied, it's removed from the list and the next gap (or `latest + 1`) is shown
  - When all gaps are filled, Next falls back to `latest + 1` automatically
- **Gap Analysis panel** — below the category cards, shows a breakdown of all missing numbers per category, expandable per category, with a **Scan Gaps** button to re-check at any time

---

## Part Numbering

Parts follow the format `CAT-NNNNN.ext`:

| Code | Category |
|---|---|
| `003` | Top Level Assembly |
| `100` | Subassembly |
| `200` | Metal |
| `240` | Copper |
| `245` | Flexibar |
| `250` | Galvanized |
| `295` | Insulation Barrier |

Each user is identified by a numeric **prefix** (e.g. `9` → numbers `90000`–`99999`). The app scopes all searches and gap checks to your prefix automatically. Per-category prefix overrides are also supported.

---

## Data Storage

The SQLite database is stored locally per user at:

```
%APPDATA%\PartsTracker\parts.db
```

No installation required — just run and go.

---

## Tech Stack

- [PyQt6](https://pypi.org/project/PyQt6/) — UI framework
- [openpyxl](https://pypi.org/project/openpyxl/) — PRF Excel parsing
- [requests](https://pypi.org/project/requests/) — Everything HTTP API
- [SQLite3](https://docs.python.org/3/library/sqlite3.html) — local database (stdlib)
