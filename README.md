# Vehicle Market Velocity Analysis

Analyzes **market velocity** (ownership transfer rates) and **market share** across the Georgian vehicle market using official registration data from the Ministry of Internal Affairs ([police.ge](https://police.ge)).

**Market velocity** = ownership transfers / active registered vehicles, normalized to a monthly rate. It reveals how "liquid" different vehicle segments are — which brands, models, and model years change hands most frequently relative to their fleet size.

## Interactive Explorer

An interactive D3.js scatter plot visualization lets you explore market velocity vs. market share across every time period in the dataset (2017-2025).

**Features:**
- **Time slider** with play animation to watch market dynamics evolve
- **Three-level drill-down:** Brands → Models → Manufacture Years
- **Multi-select:** Ctrl+click to select multiple brands or models before drilling down
- **Zoom & pan:** Ctrl+scroll to zoom into dense areas, click+drag to pan, double-click to reset
- **Search** to highlight specific brands or models
- **Log/linear scale** toggle
- **Four quadrants** divided at global median velocity and market share
- **Fixed axes** across all time periods for consistent visual comparison

### Running the Explorer

```bash
cd output
python -m http.server 8080
```

Then open `http://localhost:8080/velocity_explorer.html` in your browser.

## Data Pipeline

### Input

559 Excel files scraped from police.ge:

- **Menu 139** (119 files): Monthly/quarterly snapshots of active registered vehicles with brand, model, manufacture year, fuel type, engine displacement, region, and quantity
- **Menu 140** (81 files): Monthly/quarterly ownership transfer records with the same vehicle attributes

### Processing

`vehicle_velocity.py` handles the full pipeline:

1. **File discovery** — Parses Georgian month/quarter names from filenames, groups Part I/II splits
2. **Caching** — Converts xlsx to parquet on first run (~2 min/file), subsequent runs load in ~2s/file
3. **Standardization** — Normalizes brand/model names (strips brand prefix from models, merges aliases like DAIMLER-BENZ → MERCEDES-BENZ)
4. **Velocity computation** — Joins transfer counts onto active fleet counts at multiple granularity levels
5. **Market share** — Computes each segment's share of total active vehicles

### Output

CSV files at six granularity levels:

| File | Grouping |
|------|----------|
| `velocity_overall.csv` | National aggregate per period |
| `velocity_by_brand.csv` | Per brand |
| `velocity_by_model.csv` | Per brand + model |
| `velocity_by_model_year.csv` | Per brand + model + manufacture year |
| `velocity_by_model_engine_fuel.csv` | Per brand + model + fuel type + engine displacement |
| `velocity_by_region.csv` | Per region (transfers estimated proportionally) |

Each CSV contains: `year`, `period`, grouping columns, `active_count`, `transfers`, `velocity`, `market_share`.

## Usage

```bash
# Install dependencies
pip install pandas openpyxl pyarrow

# Run the pipeline (first run ~45 min, cached runs ~5 min)
python vehicle_velocity.py

# Generate visualization data
# (inline script — see viz_data.json generation in project history)

# Serve the explorer
cd output && python -m http.server 8080
```

## Tech Stack

- **Python** + pandas for data processing
- **D3.js v7** for interactive visualization
- **Parquet** caching layer for fast xlsx re-reads

## License

Copyright 2025 Strata Labs. All rights reserved.
