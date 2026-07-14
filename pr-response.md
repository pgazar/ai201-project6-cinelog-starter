# PR Response Doc — CineLog Watchlist Feature

## AI Usage
<!-- Fill in at the end — how you used AI tools during this project -->

## Comment 1 — Rename
**What I did:** Renamed `save_to_watchlist()` to `add_to_watchlist()` in `services/watchlist_service.py` to follow the project's `verb_to_noun` naming convention already used by `add_to_collection()`. Updated both the import and the function call in `routes/watchlist/watchlist.py`.
**How I verified:** Ran `grep -ri "save_to_watchlist" --include="*.py" .` before and after the change — found 3 occurrences (the definition, an import, and a call site), and confirmed the search returned zero results after the rename, meaning no call sites were missed. Also ran the full test suite (`pytest tests/ -v`) to confirm no regressions.

## Comment 2 — Deduplication
**What I did:** Added a duplicate check in `add_to_watchlist()`, modeled directly on the existing pattern in `add_to_collection()`. Before creating a new `WatchlistEntry`, the function now queries for an existing entry matching the same `user_id` and `film_id`. If one is found, it raises a new `AlreadyInWatchlistError` (defined locally in `watchlist_service.py`, following the same naming convention as `AlreadyInCollectionError`) instead of silently creating a duplicate.
**How I verified:** Wrote `test_add_to_watchlist_duplicate_raises` in `tests/test_watchlist.py`, which adds a film to a watchlist, then attempts to add the same film again and asserts `AlreadyInWatchlistError` is raised. The test also confirms only one entry exists in the database afterward. This test passes.

## Comment 3 — Missing test
**What I did:** Created `tests/test_watchlist.py` and wrote `test_add_to_watchlist_nonexistent_film_raises`, modeled on `test_add_to_collection_nonexistent_film_raises` in `test_collection.py`. It calls `add_to_watchlist()` with a film_id that doesn't exist and asserts `FilmNotFoundError` is raised. I also added `test_add_to_watchlist_creates_entry` to cover the basic success case, following the same fixture structure (`app`, `sample_user`, `sample_film`) used in `test_collection.py`.
**How I verified:** Ran `pytest tests/ -v` — all 7 tests pass (4 existing collection tests + 3 new watchlist tests), confirming the new tests are correctly written and the underlying logic they test is working.

## Comment 4 — Default visibility
**My position:**
**Reasoning:**
**Tradeoff acknowledged:**

## Comment 5 — Sort order
**My position:**
**Reasoning:**
**Engagement with reviewer's point:**

## Comment 6 — Rebase
**What conflicted:**
**How I resolved it:**
**How I verified no conflict remains:**

## PR Description
<!-- Written at the end — feature overview, design decisions, manual testing steps -->