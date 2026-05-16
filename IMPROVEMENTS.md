# Code Review: Remaining Items

## 21. Hardcoded limit of 500 models per task

For popular tasks this misses tail models. Could show a "Limited to top 500" note.

## Optimization Ideas

| Area | Suggestion |
|------|-----------|
| UI | **Author search/filter** for L1 table (50+ authors, no way to filter) |
| L1 table | **Virtual scrolling** for >50 authors |
| API | **Deduplicate `fetchJson` calls** — if two callers request the same URL concurrently, merge into one inflight promise |
| Cache | Prune `cache` entries by LRU when total exceeds a threshold (e.g. 50MB estimated) |
