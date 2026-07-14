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
**My position:** Watchlist entries should default to `public=False` rather than `public=True`.
**Reasoning:** CineLog is described as a community film tracking app, which suggests a social feature is planned or likely,for example, people browsing what others want to watch. But a new user's very first watchlist entry is often added before they've explored privacy settings at all, which means public-by-default exposes that user at exactly the moment they're least prepared for it. A watchlist can also be more revealing than a "watched" history, since it shows present interest or curiosity rather than something already completed and settled. Defaulting to private protects users during that vulnerable first-use moment, and makes sharing a deliberate, informed choice rather than something they inherited without noticing.
**Tradeoff acknowledged:** Defaulting to private weakens the social/discovery experience out of the box — if friends seeing each other's watchlists is meant to be a core feature, requiring users to manually opt in per entry (or in account settings) adds friction that could reduce engagement with that feature. I think this tradeoff is worth it: protecting a new user's privacy by default matters more than frictionless discovery for users who haven't yet decided they want to be seen.

## Comment 5 — Sort order
**My position:** I agree with defaulting to date-added order (most recent first) instead of alphabetical.
**Reasoning:** A watchlist is something you check when you're deciding what to watch soon — and what you're most likely to want to watch next is often whatever you just added, since that's usually why you added it in the first place. Alphabetical order would only really help someone scanning a long list to find a specific title they remember, but most users' watchlists are not long enough for that to matter. For a short, actively-used list, recency is more useful than alphabetical order.
**Engagement with reviewer's point:** Dev-lead's reasoning — "most users want to see what they added recently" — matches how I think people actually use a watchlist: as a short-term queue of things they're planning to watch soon, not a long reference list they need to search through. I don't see a strong case for alphabetical here, since it optimizes for a scenario (searching a large list) that doesn't match typical watchlist size or use.

While implementing this change, I also discovered `WatchlistEntry` was missing a `film` relationship in `models.py` (present on the analogous `CollectionEntry` pattern), which caused `get_watchlist()` to fail with an `AttributeError`. I added the missing relationship as part of this fix.

## Comment 6 — Rebase
**What conflicted:** Two files conflicted during `git rebase origin/main`:
1. `.gitignore` — both branches independently added a `.gitignore` file. My version was missing a `.pytest_cache/` entry that main's version had; I kept both sets of ignore patterns.
2. `models.py` — main's refactor (`07ca580`) migrated `Film.id` and `CollectionEntry.film_id` from `Integer` to a UUID string (`db.String(36)`). My `WatchlistEntry` class didn't exist on main, so the conflict was an add-only conflict rather than competing edits. I kept my full `WatchlistEntry` class but changed its `film_id` column from `db.Integer` to `db.String(36)` to match the new `Film.id` type.

**How I resolved it:** For `.gitignore`, I merged both versions by keeping every ignore pattern from each side. For `models.py`, I kept my `WatchlistEntry` class definition intact and only changed the `film_id` column type to match the refactored `Film.id` type. I also checked `services/watchlist_service.py` for any code that assumed `film_id` was an integer (e.g., type conversions or arithmetic) and confirmed there was none — the function treats `film_id` as an opaque value throughout, so no logic changes were needed there.

**How I verified no conflict remains:** Ran `git status` after resolving to confirm a clean working tree with no unmerged paths, and confirmed `git rebase` reported "Successfully rebased." Then ran the full test suite (`pytest tests/ -v`) — all 8 tests passed, confirming the UUID type change didn't break the watchlist feature, including the deduplication logic and sort order added in earlier comments.

## PR Description
<!-- Written at the end — feature overview, design decisions, manual testing steps -->