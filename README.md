# Parts Tracker

> Track SolidWorks parts across jobs — instantly.

A lightweight desktop app that combines **Everything's HTTP API** with **PRF Excel data** to give you a searchable, filterable view of every `.sldprt` and `.sldasm` file across your entire job structure.

---

## Features

- **Instant search** across all indexed SolidWorks parts and jobs
- **PRF integration** — auto-reads catalog number and enclosure size from Production Release Forms
- **Part categories** — Metal, Copper, Flexibar, Galvanized, Subassembly, and more
- **Per-user filtering** — scope results to your own part prefix
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

**1. Install dependencies**

```bash
pip install -r requirements.txt
```

**2. Enable Everything HTTP Server**

```
Everything → Tools → Options → HTTP Server → Enable (port 8080)
```

**3. Run**

```bash
python parts_tracker.py
```

Or double-click `run.bat`.

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

---

## Data Storage

The SQLite database is stored at:

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
