# Code Review: `streamlined-hf-model-search.html`

## Resolved Items

The following issues from this review have been addressed (see commit message for details):

| # | Issue | Fix |
|---|-------|-----|
| 1 | Pipeline filters don't affect L2 expanded view | Stored `pipeline_tag` in L2 cache; added `modelPassesAllFilters()` helper; used in `computeAuthorData`, `loadAuthorModels`, and `refreshAllExpanded` |
| 6 | `incApiCalls` overflow at > 9999 | Removed `padStart(4)` — now uses bare `String(apiCalls)` |
| 7 | Per-row/per-th event listeners on every render | Replaced with **delegated listeners** on persistent containers (L1 on `#main-table`, L2/L3/L4 on each `.detail-inner`) |
| 8 | `refreshAllExpanded` always scans full DOM | Added `expandedSections` Set tracked in JS; cleared on `renderMain`; used for early-exit + targeted ID lookups |
| 10 | No slider debouncing | Added `debounce(fn, ms)` utility; applied to both date and param slider `change` handlers (200ms) |
| 11 | No loading indicator for param deepening | Shows "Resolving parameters for {author}…" in status bar during batch fetch |
| 18 | Empty catch blocks during deepening | Added `console.warn` with model ID and error details |

---

## Remaining Issues

### Critical Bugs

### 2. `reseedActiveTaskFilters` overrides user deselecting default tags (line 541-549)
If a user clicks an activated-type chip to remove `text-generation` (a `DEFAULT_ACTIVE_TAGS` entry), then changes a From/To filter, `reseedActiveTaskFilters` silently re-adds it.

### 3. `pipeline_tag` overwritten on dedup in `fetchTasks` (line 1148)
```js
m.pipeline_tag = tasks[i]; // overwrites the model's actual pipeline tag
```

If a model appears in top-500 for multiple tasks, only the **first** task's tag survives. This means `matchesTaskFilter` and `computeAuthorData` use a possibly incorrect pipeline tag for the model.

### 4. Nested org IDs broken in `loadChildren` (line 782)
```js
const [parentAuthor, modelName] = parentId.split("/");
```

A model ID like `org/suborg/model` would lose the `model` part — `modelName` becomes `"suborg"`.

### Moderate Issues

### 5. `_apiTimestamps` concurrent access during back-to-back `await`s (line 289-303)
If two `fetchJson` calls interleave at the `await` point, both can filter + clear + repush the same array, losing entries. Practically causes mild over-ratelimit rather than a crash.

### Performance

### 9. `syncFilterChips` recreates all `act-tag` elements (line 433-434)
```js
existing.forEach(el => el.remove()); // then re-creates every tag
```

Only the diff (added/removed tags) needs DOM surgery.

### UI/UX

### 12. No author search/filter
With potentially 50+ authors, there's no way to filter the L1 table by name.

### 13. Truncated model IDs lack tooltips (line 727)
```js
truncate(m.id.split("/").slice(1).join("/"), 40)
```

No `title` attribute means users can't see the full model name on hover.

### 14. "Get Results" button never changes label (line 158)
After the first click, it still says "Get Results" — consider "Refresh" or show fetch progress inline.

### 15. Inactive chip contrast (line 67)
```css
.filter-chip.inactive { color: #484f58; }
```

On `#0d1117` background, WCAG AA fails (ratio ~3.3:1). Bump to `#6e7681`.

### 16. No collapse-all or expand-all for sections
Each L1/L2/L3 row must be clicked individually to expand/collapse.

### 17. Param slider non-linear mapping is visually misleading (line 262)
30 positions, but 0–10 are step-1, 10–100 step-10, 100–1000 step-100. A user dragging the slider sees position linearly but values jump non-linearly. A logarithmic scale or shown numeric label next to the thumb would help.

### Code Quality & Maintainability

### 19. No API retry logic (line 289-304)
HF API can return 429 (rate limit) or 5xx. A simple retry with exponential backoff would improve reliability.

### 20. `_allFetched` only grows, never prunes (line 1046-1054)
Switching from `text-generation` tasks to `image-to-text` tasks still keeps the `text-generation` models in memory. A user doing many filter cycles accumulates unbounded memory.

### 21. Hardcoded limit of 500 models per task (line 253)
For popular tasks this misses tail models. Could show a "Limited to top 500" note.

### Optimization Ideas

| Area | Suggestion |
|------|-----------|
| L1 table | **Virtual scrolling** for >50 authors |
| API | **Deduplicate `fetchJson` calls** — if two callers request the same URL concurrently, merge into one inflight promise |
| Cache | Prune `cache` entries by LRU when total exceeds a threshold (e.g. 50MB estimated) |
