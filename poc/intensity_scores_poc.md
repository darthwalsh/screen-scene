# Spike 1 — Where do intensity scores come from?

**Question:** Lowest-effort, highest-accuracy way to get intensity scores that match parent intuition?

**Status:** method chosen. Five-film gold set is in `data/scores.yaml` and **needs human review** before we scale to ~50.

## Method chosen

**Original judgment against a written rubric** (`data/RUBRIC.md`).

Not: scrape CSM, scrape Kids-In-Mind, parse TMDB keywords, or “LLM, pretend you are CSM.”

Why this wins for v0:

- We need dimensions CSM does not separate (scary vs violence, heavy vs scary).
- Copying their numbers fails Spike 0 and also can’t express our categories.
- Five films is small enough to argue about every integer.
- The calibration product only works if the relative *shape* of scores is right (Up is a heavy spike, Incredibles is a violence spike, Luca is a floor). Absolute precision to CSM’s dots is not the goal.

LLM-assisted drafts are allowed later as a *pre-fill* for films 6–50, then a human pass. They are not the source of truth.

## Dimensions (changed from PLAN)

PLAN MVP listed `scary`, `sad`, `violence` on 1–10.

Published v0 uses five dimensions on **0–5**:

| PLAN | v0 | Why |
|---|---|---|
| scary | scary | Keep. Split from violence. |
| violence | violence | Keep. |
| sad | **heavy** | Broader than crying: death, grief, abandonment, the thing that lingers. |
| — | **language** | PLAN’s own CSM gap. Cheap to store, rare in Pixar, so a 3 actually means something when we add PG-13. |
| — | **crude** | Innuendo / potty / body — the other thing parents disagree about. |

Scale **0–5** instead of 1–10: 0 is a real value (“none”), it matches how parents already read CSM dots, and a 5-film set does not have 11-point resolution. We can double later if review says we need the extra bit.

## Gold set (intentionally spiky)

| Film | Scary | Language | Violence | Crude | Heavy | max() | Why it’s in the set |
|---|---|---|---|---|---|---|---|
| Luca (2021) | 1 | 1 | 2 | 1 | 2 | 2 | Floor. “Fine for us” anchor. |
| Up (2009) | 2 | 1 | 3 | 1 | **5** | 5 | Heavy spike, not a scare spike. |
| Finding Nemo (2003) | **4** | 1 | 3 | 1 | 4 | 4 | G-rated trap. Scare + heavy in act 1. |
| The Incredibles (2004) | 2 | 2 | **4** | 2 | 2 | 4 | Violence spike, low heavy. |
| Toy Story 3 (2010) | **5** | 1 | 4 | 1 | 4 | 5 | Scare ceiling. Another G-rated trap. |

If your intuition wants to swap any two columns, say so before we write film 6.

## What this already shows about Spike 2 (`max()`)

Uncalibrated rank by `max()`:

1. Up, Toy Story 3 (5)
2. Nemo, Incredibles (4)
3. Luca (2)

That is a decent “how intense is this movie” list, and it is also **wrong for a real kid**:

- Fine with grief, not fire → Up is OK, TS3 is not. `max()` ties them.
- Fine with action, not the Nemo opening → Incredibles is OK, Nemo is not. `max()` ties them.

So: `max()` is an acceptable **default sort** when the parent has not calibrated. It is a bad **filter**. PLAN’s per-dimension thresholds are the product; Spike 2 should test threshold logic, not pick `max()` vs `avg()` as the only ranking.

Still open: parent sanity-test of the threshold rule once these five scores are accepted.

## Next

1. Review `data/scores.yaml` against `data/RUBRIC.md`.
2. Lock the five integers (or change the rubric).
3. Only then expand toward the ~50-film list (two cohorts in PLAN).
