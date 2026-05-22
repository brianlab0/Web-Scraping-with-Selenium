# Steam Ranking Data Scraping

A full-stack data project that scrapes the Steam Top Sellers chart for the Taiwan region across 2021 to 2024 on a weekly cadence, stores the results into a structured CSV dataset, and provides a desktop GUI application for interactive analysis and visualization of the business model trend (free vs. paid games) and popularity rankings.

---

## Overview

The project is split into two main stages:

1. **Data acquisition** — Uses Selenium with ChromeDriver to automate a real browser, navigates to the Steam regional top-sellers chart for each weekly date across four years, expands the full top-100 list, and extracts every game title together with its price (or "Free to Play" marker).

2. **Interactive analysis** — A Tkinter desktop GUI lets the user pick any year or any specific week from the scraped dataset and instantly generate visual analytics: pie charts comparing free vs. paid game ratios, ranked text listings of weekly and yearly hits, and a multi-year line-chart trend of the free-versus-paid business model on the Steam platform.

The final output is both a clean reusable CSV dataset (`steam_yearly_data.csv`) and a single-file desktop application for non-technical viewers.

---

## Tech Stack

- Python 3
- Selenium (browser automation for scraping)
- ChromeDriver (target browser engine)
- pandas (data loading and aggregation)
- matplotlib (pie charts and trend line charts)
- Tkinter (cross-platform desktop GUI)

---

## Pipeline Architecture

```
Steam Top Sellers (Taiwan region)
        |
        v
Selenium + ChromeDriver  (week-by-week iteration, 2021 to 2024)
        |
        v
pandas DataFrame   ->   steam_yearly_data.csv
        |
        v
Tkinter GUI (SteamAnalyzerGUI)
        |
        +-- Weekly Game Type Analysis    (free vs paid pie chart)
        +-- Annual Game Type Statistics  (free vs paid pie chart)
        +-- Weekly Game Ranking          (text-based top list)
        +-- Annual Top 100 Games         (aggregated by appearances and average rank)
        +-- Four-Year Top 100 Games      (aggregated across 2021 to 2024)
        +-- Four-Year Type Trend         (free vs paid percentage line chart)
```

---

## Scraping Logic

- For each target date, the scraper opens `https://store.steampowered.com/charts/topsellers/TW/{date}` in a real Chrome instance.
- After the page is rendered, it scrolls to the bottom and clicks the "view all" dialog button to expand the chart from the default short list into the full top-100 view.
- The script then collects all matching title and price DOM elements, pairs them up, and appends them to the global collection along with the snapshot date.
- A small sleep is inserted between requests to avoid hammering the Steam servers, with a longer warm-up wait on the very first page load.

The list of target dates covers every weekly chart from January 2021 through mid-December 2024 (over 200 distinct snapshots).

---

## Dataset Schema

The resulting `steam_yearly_data.csv` contains one row per (date, ranked game) pair:

| Column | Type | Description |
| --- | --- | --- |
| `date` | string | The chart snapshot date in `YYYY-M-D` format. |
| `title` | string | The displayed game title (in Traditional Chinese where applicable). |
| `price` | string | The displayed price, including the literal value `免費遊玩` (Free to Play) for free games. |

---

## GUI Application Features

The `SteamAnalyzerGUI` class wires up a Tkinter window (default size 1600x1000) with the following analyses:

| Button | Description |
| --- | --- |
| Weekly Game Type Analysis | Pie chart of free vs. paid games for the selected week. |
| Annual Game Type Statistics | Pie chart of free vs. paid games aggregated across the selected year. |
| Weekly Game Ranking | Text-based ranked listing of the selected week's chart. |
| Annual Top 100 Games | Aggregated yearly ranking by number of chart appearances plus average ranking. |
| Four-Year Top 100 Games | Same aggregation but spanning the full 2021 to 2024 period. |
| Four-Year Type Trend | Line chart showing the yearly free vs. paid percentage trend across all four years. |

The UI uses cascading combo boxes — selecting a year automatically updates the available week list — so the user can quickly drill into any of the 200+ collected weekly snapshots.

---

## Project Structure

```
Steam_Ranking_Data_Scraping/
  Steam_Ranking_Data_Scraping.ipynb   Main notebook containing scraper + GUI app
  steam_top_sellers.csv               Single-snapshot CSV output (first prototype cell)
  steam_yearly_data.csv               Full 2021-2024 weekly dataset (main scraper output)
```

The notebook is organized into three logical sections:

1. A prototype single-date scraper (proof of concept on the 2024-12-17 chart).
2. A production multi-year scraper that iterates over every weekly date.
3. A Tkinter GUI application class that consumes the produced CSV.

---

## How to Run

### Prerequisites

- Python 3.8 or later.
- Google Chrome installed locally.
- A compatible ChromeDriver available on `PATH` (Selenium 4.6+ will auto-resolve in most cases).
- Required Python packages:

```bash
pip install selenium pandas matplotlib
```

(`tkinter` ships with the standard Python distribution on most platforms.)

### Step 1 — Scrape the data

Run the second cell of the notebook (the `crawl_yearly_data()` block). A real Chrome window will open and the script will iterate through every weekly date from 2021 to 2024. Depending on network conditions the full run can take a substantial amount of time.

The script writes the consolidated dataset to `steam_yearly_data.csv` in the working directory.

### Step 2 — Launch the analyzer

Run the third cell (the `SteamAnalyzerGUI` block). The desktop window will open. Pick a year and (optionally) a specific week from the dropdowns and click any of the six analysis buttons to render the visualization or ranking in the result panel.

---

## Highlights

- End-to-end pipeline from raw web scraping to interactive desktop analytics.
- Robust four-year time-series dataset (over 200 weekly snapshots of the top 100 Taiwan Steam sellers).
- Real-browser scraping using Selenium handles dynamically rendered chart content that simple HTTP scraping cannot capture.
- Multiple analytical lenses on the same dataset: weekly snapshots, annual aggregates, and four-year longitudinal trends.
- Self-contained Tkinter GUI requires no web server, no database, and no extra build tooling — just Python.

---

## Notes and Limitations

- The Steam page DOM classes used (`_1n_4-zvf0n4aqGEksbgW9N`, `_3j4dI1yA7cRfCvK8h406OB`) are auto-generated hashes and may change over time. If Steam re-deploys with new class names, the selectors will need updating.
- The first cell shows an example `NoSuchWindowException` from a test run; the production scraper in the second cell uses a safer selector strategy and explicit scroll-to-bottom before clicking the expand button.
- The current implementation targets the Taiwan (TW) regional chart. Adapting to other regions only requires changing the URL country code.
- Scraping respects polite delays between requests but you should review Steam's Terms of Service before running large-scale repeated scrapes.
