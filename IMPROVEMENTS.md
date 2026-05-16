# Code Review: `streamlined-hf-model-search.html`

## Resolved Items

The following issues from this review have been addressed (see commit message for details):

| # | Issue | Fix |
|---|-------|-----|
| 1 | Pipeline filters don't affect L2 expanded view | Stored `pipeline_tag` in L2 cache; added `modelPassesAllFilters()` helper; used in `computeAuthorData`, `loadAuthorModels`, and `refreshAllExpanded` |
| 3 | `pipeline_tag` overwritten on dedup in `fetchTasks` | Changed to `if (!m.pipeline_tag) m.pipeline_tag = tasks[i]` so the API's original tag is preserved |
| 4 | Nested org IDs broken in `loadChildren` | Use `parentId.split("/").slice(1).join("/")` to get the full model name portion (handles `org/suborg/model`) |
| 5 | `_apiTimestamps` concurrent access in `fetchJson` | Replaced filter-clear-repush pattern with shift-based pruning for atomicity across `await` points |
| 6 | `incApiCalls` overflow at > 9999 | Removed `padStart(4)` — now uses bare `String(apiCalls)` |
| 7 | Per-row/per-th event listeners on every render | Replaced with **delegated listeners** on persistent containers (L1 on `#main-table`, L2/L3/L4 on each `.detail-inner`) |
| 8 | `refreshAllExpanded` always scans full DOM | Added `expandedSections` Set tracked in JS; cleared on `renderMain`; used for early-exit + targeted ID lookups |
| 9 | `syncFilterChips` recreates all `act-tag` elements | Changed to diff-based DOM update — only create, remove, or reclass as needed using a `tagMap` |
| 10 | No slider debouncing | Added `debounce(fn, ms)` utility; applied to both date and param slider `change` handlers (200ms) |
| 11 | No loading indicator for param deepening | Shows "Resolving parameters for {author}…" in status bar during batch fetch |
| 13 | Truncated model IDs lack tooltips | Added `title` attribute to L2 and L4 model ID links |
| 14 | "Get Results" button never changes label | Switches to "Refresh" after first successful fetch in `applyFilters` |
| 15 | Inactive chip contrast | Changed `#484f58` to `#6e7681` for WCAG AA compliance |
| 16 | `expandedSections` orphans on parent collapse | On L1/L2/L3 collapse, cascade-delete all descendant IDs from `expandedSections` |
| 17 | Param slider non-linear stepping | Replaced 30-position stepped slider with logarithmic 200-position continuous scale; added tooltips and visual legend |
| 18 | Empty catch blocks during deepening | Added `console.warn` with model ID and error details |
| 19 | No API retry logic | Added exponential backoff (`1×2^n`, cap 30s, max 3 retries) for 429, 5xx, and network errors |

---

## Remaining Issues

### 21. Hardcoded limit of 500 models per task (line 253)
For popular tasks this misses tail models. Could show a "Limited to top 500" note.

### Optimization Ideas

| Area | Suggestion |
|------|-----------|
| UI | **Author search/filter** for L1 table (50+ authors, no way to filter) |
| L1 table | **Virtual scrolling** for >50 authors |
| API | **Deduplicate `fetchJson` calls** — if two callers request the same URL concurrently, merge into one inflight promise |
| Cache | Prune `cache` entries by LRU when total exceeds a threshold (e.g. 50MB estimated) |
