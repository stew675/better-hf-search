# AGENTS.md — Better HF Search

## Project Overview

Single-file, zero-dependency HTML/JS app that explores HuggingFace base models in a 4-level hierarchy. All logic lives in `better-hf-search.html`.

## File Structure

```
better-hf-search.html   — Single-file app (HTML + CSS + JS)
README.md               — User documentation
AGENTS.md               — This file
```

## Architecture

### Data Flow
1. **Init**: Fetch top 500 models per pipeline task → deduplicate → filter by base-model status → store all in `window._allFetched`
2. **Render**: `computeAuthorData()` applies date + param slider ranges to `_allFetched` → groups by author → renders L1
3. **L1 expand**: Fetch full author model list (1000) → filter base models → cache full list → apply date + param slider filters → render L2 → deepen unknown `paramB` in batches of 5 via individual model API
4. **L2 expand**: Search HF API for children by parent ID and model name → match on `cardData.base_model` or quant tags
5. **L3/L4**: Group children by quant author, apply active filters, render sortable table

### State Management
- `window._authorData` — L1 author records (mutable, updated on L1 expand, slider changes)
- `window._allFetched` — All base models fetched during init (no date/param filter)
- `cache` — Global in-memory cache keyed by:
  - `"{author}"` → L2 base models array
  - `"{author}_models"` → raw API response for author
  - `"children-{parentId}"` → L3/L4 children array
- `detailSort` — Per-section sort state keyed by `"l2-{idx}"`, `"l3-{l2}-{model}"`, `"l4-{l2}-{model}-{g}"`
- `activeFilters` — Set of enabled quant type strings
- `sliderFrom` / `sliderTo` — Date slider positions (-25..0, months relative to now)
- `paramSliderFrom` / `paramSliderTo` — Param size slider positions (0..29, mapped via `PARAM_VALUES`)

### Key Functions

| Function | Purpose |
|----------|---------|
| `renderMain(authorData)` | Renders L1 author table |
| `renderL2(idx, models, container)` | Renders L2 base models |
| `renderL3(l2Idx, modelIdx, parentId, children, container)` | Renders L3 quant author groups |
| `renderL4(l2Idx, modelIdx, gIdx, quants, container)` | Renders L4 individual quants |
| `loadAuthorModels(idx, author, container)` | Async fetch for L2, deepens unknown `paramB` in batches of 5 |
| `loadChildren(l2Idx, modelIdx, parentId, container)` | Async fetch for L3/L4 |
| `refreshAllExpanded()` | Re-renders all open sections (filter/slider changes) |
| `matchesFilter(qMethod)` | Checks if a quant method passes active filters |
| `computeAuthorData()` | Applies date + param filters to `_allFetched`, groups by author |
| `isInDateRange(createdAt)` | Date slider range check |
| `isInParamRange(paramB)` | Param slider range check |
| `getParamCount(model)` | Extracts param count from `safetensors.total` or B/M suffix in model ID |
| `paramValueToLabel(val)` | Formats param count for display (int ≥5B, 1 decimal ≥1B, int M <1B) |
| `buildDateSlider()` | Builds the date dual-range slider |
| `buildParamSlider()` | Builds the param size dual-range slider |
| `sortRows(rows, key, asc)` | Generic sort for any level |
| `updateArrows()` | Updates sort arrow indicators on L1 table header |

### Constants

- `TASKS` — Pipeline tags to search (`text-generation`, `image-text-to-text`, `image-to-text` — see full list at `packages/tasks/src/pipelines.ts` in huggingface.js)
- `LIMIT` — Models fetched per task (500)
- `PARAM_VALUES` — 30 positions mapping to param sizes: 0..10 (step 1), 10..100 (step 10), 100..1000 (step 100), >1T (Infinity)
- `Q_METHODS` — All quantization keywords for detection
- `FILTER_DISPLAY` — Subset shown in filter bar (AWQ, GPTQ, GGUF, EXL2, Marlin, BitsAndBytes, AQLM, EETQ, MLX)

## Conventions

- **No external dependencies** — everything inline
- **No comments in code** — keep it compact
- **Dark theme** — GitHub/HF color palette (`#0d1117`, `#161b22`, `#58a6ff`, etc.)
- **Indentation** — 2 spaces
- **Event delegation** — attach handlers after `innerHTML` injection
- **ID scheme** — `t{level}-{idx}` for toggles, `d{level}-{idx}` for detail rows, `i{level}-{idx}` for inner containers

## Testing

Open `better-hf-search.html` in a browser. Validate:
1. L1 loads with authors and counts
2. Clicking an author expands to L2 with matching count
3. Clicking a base model expands to L3 (quant author groups)
4. Clicking a quant author expands to L4 (individual models)
5. Filter checkboxes update all expanded sections
6. Column headers toggle sort direction
7. Links open model pages in new tabs
8. Date slider changes re-render L1 and all open L2 sections
9. Param slider changes re-render L1 and all open L2 sections
10. L2 shows "Params" column; unknown params fetch in batches of 5

## Common Pitfalls

- **Data attribute names**: `data-l3-model-idx` ≠ `data-l3-model`. Always verify matching.
- **ID collisions**: L3/L4 IDs include all parent indices (`d3-{l2}-{model}-{g}`).
- **Filter refresh**: `refreshAllExpanded` queries DOM for visible detail rows — must re-render after any filter change.
- **Cache keys**: `children-{parentId}` uses the full model ID (e.g., `Qwen/Qwen2.5-7B`).
- **Author name stripping**: L2 displays `m.id.split("/").slice(1).join("/")` to avoid repeating the author.
- **L1 sort selector**: Uses `#main-table > thead > tr > th` to avoid L2/L3 nested `<th>` triggering. Do NOT delegate to `#main-table thead`.
- **Param deepening**: Only fires for the single author the user expands, in batches of 5 — prevents spamming the API for all unknowns at once.
- **Search endpoint limitations**: The search API (`/api/models?search=...`) never returns `safetensors` or `config` data even with `full=true`. The individual model API (`/api/models/{id}?full=true`) does, which is why deepening is needed for models without B/M in their name.
