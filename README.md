# Better HF Search

A browser-based, 4-level hierarchical explorer for HuggingFace base models and their quantizations — no server, no install, just open the HTML file.

## Quick Start

Open `better-hf-search.html` in any modern browser. That's it.

## How It Works

| Level | What You See | Sortable By |
|-------|-------------|-------------|
| **1 — Authors** | Organizations with base models matching selected range | Author, Base Models, Total Downloads, Total Likes |
| **2 — Base Models** | Individual base models by that author | Model ID, Params, Created, Downloads, Likes |
| **3 — Quant Authors** | Who made quantizations, grouped by author | Models, Downloads |
| **4 — Quantizations** | Individual quantized models with method badges | Model ID, Quant, Downloads, Likes, Created, Updated |

## Dual-Range Sliders

Two sliders above the table let you filter by **date** (months ago, up to 25 months back or "Anytime") and **parameter size** (30 positions from 0 to >1T). From slider is red, to slider is green. Changes immediately re-render L1 and all open sections.

## Quantization Filter

Checkbox bar above the table lets you toggle quant types on/off:
AWQ, GPTQ, GGUF, EXL2, Marlin, BitsAndBytes, AQLM, EETQ, MLX.

Use **All** / **None** buttons for quick reset. Filters apply instantly to all expanded sections.

## Features

- **Zero dependencies** — single HTML file, no build step
- **Dark theme** matching GitHub/HF styling
- **Dual-range sliders** for date and parameter size filtering
- **Column sorting** at every level (click headers)
- **Expandable rows** — click any row (not a link) to expand
- **Cached results** — re-expanding is instant
- **Param deepening** — unknown param counts fetched in batches of 5 via individual model API
- **Live data** — fetches directly from the HuggingFace API
- **Quant badges** — color-coded by method (blue = quant, green = derivative)

## Data Source

Uses the public HuggingFace Hub API (`https://huggingface.co/api/models`). No API key required.

## Browser Requirements

Modern browser with ES2020+ support (Chrome 90+, Firefox 90+, Safari 14+).

## Future Ideas

- Expand pipeline tags beyond the current 3 (56 total available)
- Search/filter by model name at any level
- Export results to CSV
- Bookmarkable state (URL params for filters)
- Model architecture tags in L2
- Trending / likes-per-day metrics
