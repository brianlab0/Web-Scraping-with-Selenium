# Steam Ranking Data Scraping

## Overview

The project is split into two main stages:

### 1. Data Acquisition
Uses **Selenium with ChromeDriver** to automate a real browser, navigates to the Steam regional top-sellers chart for each weekly date from 2024 to 2026, expands the full top-100 list, and extracts every game title together with its price (or `免費遊玩` / Free to Play marker).

### 2. Interactive Analysis
A **Tkinter desktop GUI** lets the user pick any year or specific week from the scraped dataset and instantly generate visual analytics:

- Pie charts comparing free vs. paid game ratios
- Ranked text listings of weekly and yearly hits
- Multi-year line-chart trend of the free-versus-paid business model on Steam

The final deliverables are a clean, reusable CSV dataset (`steam_yearly_data.csv`) and a single-file desktop application accessible to non-technical viewers.

---

## Tech Stack

| Layer | Technology |
|-------|------------|
| Language | Python 3.8+ |
| Web Scraping | Selenium + ChromeDriver |
| Data Processing | pandas |
| Visualization | matplotlib |
| GUI Framework | Tkinter (standard library) |

---

## Pipeline Architecture

```
Steam Top Sellers (Taiwan region)
              │
              ▼
   Selenium + ChromeDriver
   (week-by-week iteration, 2024 → 2026)
              │
              ▼
       pandas DataFrame
              │
              ▼
     steam_yearly_data.csv
              │
              ▼
   Tkinter GUI (SteamAnalyzerGUI)
              │
   ┌──────────┼──────────────────────────────┐
   │          │                              │
   ├─ Weekly Game Type Analysis    (pie chart)
   ├─ Annual Game Type Statistics  (pie chart)
   ├─ Weekly Game Ranking          (ranked text list)
   ├─ Annual Top 100 Games         (aggregated yearly)
   ├─ Three-Year Top 100 Games     (aggregated 2024–2026)
   └─ Three-Year Type Trend        (free vs paid line chart)
```

---

## Scraping Logic

For each target date, the scraper:

1. Opens `https://store.steampowered.com/charts/topsellers/TW/{date}` in a real Chrome instance
2. Waits for the page to render, then **scrolls to the bottom** to trigger lazy loading
3. Clicks the **"view all"** dialog button to expand the chart from the default short list into the full top-100 view
4. Collects all matching title and price DOM elements, pairs them up, and appends them to the global collection along with the snapshot date
5. Inserts a small **sleep between requests** to avoid hammering Steam servers, with a longer warm-up wait on the very first page load

The list of target dates covers every weekly chart from **January 2024 through May 2026** — over **120 distinct snapshots**.

---

## Dataset Schema

The resulting `steam_yearly_data.csv` contains one row per `(date, ranked game)` pair:

| Column | Type | Description |
|--------|------|-------------|
| `date` | string | The chart snapshot date in `YYYY-M-D` format |
| `title` | string | The displayed game title (in Traditional Chinese where applicable) |
| `price` | string | The displayed price, including the literal value `免費遊玩` (Free to Play) for free games |

**Total records:** ~12,000 rows across the three-year period.

---

## GUI Application Features

The `SteamAnalyzerGUI` class wires up a Tkinter window (default size **1600 × 1000**) with the following analyses:

| Button | Description |
|--------|-------------|
| **Weekly Game Type Analysis** | Pie chart of free vs. paid games for the selected week |
| **Annual Game Type Statistics** | Pie chart of free vs. paid games aggregated across the selected year |
| **Weekly Game Ranking** | Text-based ranked listing of the selected week's chart |
| **Annual Top 100 Games** | Aggregated yearly ranking by number of chart appearances + average rank |
| **Three-Year Top 100 Games** | Same aggregation but spanning the full 2024–2026 period |
| **Three-Year Type Trend** | Line chart showing the yearly free vs. paid percentage trend across all three years |

The UI uses **cascading combo boxes** — selecting a year automatically updates the available week list — allowing users to quickly drill into any of the 120+ collected weekly snapshots.

---

## Project Structure

```
Steam_Ranking_Data_Scraping/
├── Steam_Ranking_Data_Scraping.ipynb   # Main notebook: scraper + GUI app
├── steam_top_sellers.csv               # Single-snapshot CSV (first prototype)
└── steam_yearly_data.csv               # Full 2024–2026 weekly dataset
```

The notebook is organized into **three logical sections**:

1. **Prototype scraper** — Proof of concept on a single chart snapshot
2. **Production scraper** — Iterates over every weekly date across the three years
3. **GUI application** — Tkinter class that consumes the produced CSV

---

## How to Run

### Prerequisites

- Python **3.8 or later**
- **Google Chrome** installed locally
- A compatible **ChromeDriver** available on `PATH` (Selenium 4.6+ auto-resolves in most cases)
- Required Python packages:

```bash
pip install selenium pandas matplotlib
```

> `tkinter` ships with the standard Python distribution on most platforms.

### Step 1 — Scrape the Data

Run the second cell of the notebook (the `crawl_yearly_data()` block).

A real Chrome window will open and the script will iterate through every weekly date from 2024 to 2026. Depending on network conditions, the full run can take **30 minutes to several hours**.

The script writes the consolidated dataset to `steam_yearly_data.csv` in the working directory.

### Step 2 — Launch the Analyzer

Run the third cell (the `SteamAnalyzerGUI` block). The desktop window will open.

1. Pick a **year** from the dropdown
2. (Optionally) pick a **specific week** from the auto-populated list
3. Click any of the six analysis buttons to render the visualization or ranking in the result panel

---

## Highlights

- **End-to-end pipeline** — From raw web scraping to interactive desktop analytics
- **Robust longitudinal dataset** — Over 120 weekly snapshots of the top 100 Taiwan Steam sellers across three years
- **Real-browser scraping** — Selenium handles dynamically rendered SPA content that simple HTTP scraping cannot capture
- **Multiple analytical lenses** — Weekly snapshots, annual aggregates, and three-year longitudinal trends on the same dataset
- **Self-contained desktop app** — No web server, no database, no extra build tooling — just Python
- **Cascading UI controls** — Year-to-week dropdown chaining for fast drill-down

---
