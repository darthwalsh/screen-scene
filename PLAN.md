# Kid-Appropriate Movie Ranking Tool — PLAN

## Status (2026-09-23)

Working in this repo, not a generated app. First deliverable is editorial scores, not UI.

| | Original PLAN | Decision |
|---|---|---|
| Product | Calibrate from 2–4 known movies → per-dimension thresholds → filter | **Keep. This is the product.** A review-site with sliders is a weaker version of the same idea. |
| Dimensions | `scary`, `sad`, `violence` | **`scary`, `language`, `violence`, `crude`, `heavy`.** `heavy` is PLAN’s `sad`, broader. Language + crude were the gaps PLAN already called out in CSM. |
| Scale | 1–10 | **0–5.** 0 = none. Rubric in `data/RUBRIC.md`. |
| Score source | Manual CSM lookup listed first | **Original judgment.** CSM/KIM numbers are not copied. Spike 0, 1 closed. |
| Stack | Static HTML + Tailwind CDN, GitHub Pages | **Keep.** No app this pass. |
| Catalog | ~50, two cohorts | Start with **5 gold films**, review, then expand. |

Open spikes: **2** (threshold aggregation — `max()` is not the filter) and **3** (TMDB ids / poster paths).

---

## Problem

Parents frequently encounter movies they thought would be fine but turned out to contain a scene that was too frightening, too sad, or too mature for their specific child. Generic ratings (G, PG) don't capture this, especially for a "G" movie our parents grew up with! The goal is a tool that lets parents calibrate against movies they already know — ones their kid loved, and ones that were too much — and infer a safe list from that.

**Core insight:** Intensity scores let you treat movie selection as a threshold problem. A parent isn't rating "movies in general" — they're locating their child's sensitivity floor in a few dimensions (scary, heavy, violent, plus language and crude), and finding movies that stay under it.

**Assumptions (editable):**
- Initial movie catalog is focused on two cohorts: (1) movies kids are currently excited to see, (2) movies millennials grew up with where the "that one scene" is forgotten
- Scores exist and are accurate enough to be directionally useful
- Parents can identify 2–4 calibration movies by name without help
- A single "max" aggregation across intensity sub-scores is a reasonable **default sort**, not the filter (see Spike 2)
- The product's core value is the calibration workflow, not the catalog size

---

## Competitive Analysis

**Does anything already solve this?**

Short answer: no. Existing tools either give you reference data (read-only) or let you mute/skip scenes in real time. None do parent-calibrated threshold inference from known movies.

### Common Sense Media

- **What it does:** Expert-written reviews with dimension scores (0–5 scale) for Violence & Scariness, Language, Sex/Nudity, Drinking/Drugs, Positive Role Models, Diverse Representations, etc. Per-title "youngest appropriate age" label. Mobile app available.
- **Fine-grained age ratings:** Has age brackets (2+, 5+, 7+, etc.) but these are editorial judgments, not calibration from *your child's* known reactions.
- **API:** Developer Center is contact-us / partnership, not a self-serve hobbyist API. Not a live data source for us.
- **Gap — the calibration problem:** Common Sense Media (CSM) doesn't know that *your* 5-year-old was fine with *Moana* but fell apart at *Finding Nemo*. Their "age 5+" is a population average, not specific to a child's sensitivity profile. You can't say "my kid handled X — what else is at that level?"
- **Gap — dimension sensitivity:** CSM gives you Violence & Scariness as one combined bucket. A parent who's worried about jump scares but not sad endings can't filter on that distinction. Profanity-sensitive families can't easily tune "language" up without raising everything else.
- **Gap — the millennial nostalgia problem:** CSM reviews are accurate but you have to search per-title. There's no "show me everything from your childhood that has a scene parents tend to forget about" view.
- **Verdict:** Best *reference* for scene facts in your own head. **Not a score source we copy.** Not a replacement for the calibration workflow.

### Screenwise

- **What it does:** ~5-minute survey of the family's digital habits / screen time, then comparison to other families. Free, anonymous.
- **Gap:** Not a movie product. Survey is not anchored to movies you already know. No "I know Luca was fine, what else like it?"
- **Verdict:** Closer in spirit (personalization) but a different problem. Not redundant.
- [x] Tried [this](https://screenwiseapp.com/learn) — digital-life survey, not movie calibration.

### Kids-In-Mind

- **What it does:** Blunt 1–10 scores for Sex/Nudity (S), Violence/Gore (V), and Profanity (L). Clinical, no narrative. Free, no login.
- **Gap:** Read-only reference only — no filtering UI, no recommendations, no calibration. Three fixed dimensions; no sad/emotional weight, no jump-scares distinction.
- **ToS:** Editorial content is copyrighted. Commercial reproduction without permission is prohibited. Do not scrape into this repo.
- **Verdict:** Useful to read. Not a data feed. Not a product.

### VidAngel ($9.99/month)

- **What it does:** Real-time scene-level skipping/muting across Netflix, Prime, Peacock, Apple TV+. Highly granular filters (kissing subcategories, specific language types, etc.).
- **Gap:** Reactive tool (mutes bad things) rather than predictive (recommends safe things). Requires active subscription per stream. No "what should I watch next?" feature.
- **Gap — catalog:** Works on streaming content, not a browse/discover interface. Doesn't solve "what movie should we put on tonight?"
- **Verdict:** Compelling v3 spike — their scene-level intensity data may map to score signals. Not a replacement for the recommendation workflow.

### JustWatch

- **What it does:** Streaming aggregator — where to watch any movie, across services. Parental controls exist only as MPAA rating filter.
- **Verdict:** No sensitivity features at all.

### The gap nobody fills

The specific combination that doesn't exist:
1. Parent inputs 2–4 movies they already know (calibration anchors)
2. App infers child's sensitivity thresholds per dimension from those anchors
3. App ranks/filters a curated catalog against those thresholds
4. Curated "millennial nostalgia" cohort — movies adults assume are fine but aren't

**→ Worth building.** Score data is original (`data/scores.yaml`), not a CSM mirror.

---

## MVP

**Goal:** Validate product fit. Can parents calibrate their child's thresholds using movies they already know, and get a useful recommendation from that?

**Success metric:** A parent sits down, enters 2 "too scary" and 2 "fine" movies, and the recommendations feel correct to them without explanation.

### MVP Features

- Static HTML + Tailwind (CDN, no build step required)
- Movies file: start from `data/scores.yaml` (5 gold films), expand to ~50 (current-excitement + millennial-nostalgia cohorts)
- Intensity scores: `scary`, `language`, `violence`, `crude`, `heavy` on 0–5
- Calibration UI: parent marks movies as "Fine for us" or "Too much"
- Threshold engine: derive **per-dimension** thresholds from calibration inputs; filter the movie list
- Display: responsive poster grid (2-col mobile, 5-col desktop), movie title, year
- LocalStorage: remember calibration across sessions
- Posters: TMDB `poster_path` stored in data; images loaded from TMDB’s image host at runtime (not downloaded into the repo — 6-month cache limit)
- Plain white theme is fine

### MVP Spikes (before building)

See Spikes section below. 0 and 1 are done. Do not start the HTML app until the gold-set scores survive review.

### MVP Non-goals (moved to v2/v3)

- Search by title
- "Why was this hidden/ranked too scary?" explanations
- "This or That" quiz workflow
- Toast notifications on calibration actions
- Sub-reason refinement ("Was it the shark or the separation?")
- Score breakdown UI (showing raw numbers to parents)
- Any server-side component
- User accounts or sharing
- Filtering by category/genre
- More than ~50 movies at launch
- Adding movies not in the initial curated list

---

## v2 great things to add

- Expand to ~200 movies (full Disney/Pixar + popular non-Disney family films)
- "This or That" calibration game: show two movies, ask which was more appropriate, tighten threshold
- Sub-reason refinement: "Was it the shark or the separation?" → adjust per-dimension weight
- Toast/inline feedback: "I've hidden 12 movies more intense than this one"
- Score breakdown: parent can optionally view raw dimension scores
- Trigger tags: `["sharks", "death of a parent", "loud noises"]` shown on hover or card flip
- Export/share: URL-encoded threshold state (no server needed, just query params)

---

## v3 trickier ideas

- Title search: find any movie (e.g., Terminator 1 vs Terminator 2) and see dimension scores side-by-side
- Per-movie scene breakdown: "Scene 3 of 8: The shark attack — Scary: 4, Duration: ~2min"
- VidAngel investigation spike: does their scene-level skip data map to intensity deltas? (e.g., "skip this 5 seconds and the fright score drops from 4 to 2")
- Problematic content dimensions: harmful tropes, representation issues, outdated biases (beyond scary/violent)
- Community score layer: parent-submitted scores to supplement or override the editorial baseline

---

## Spikes

Spikes follow PoC conventions: single question, straight-line code, placed in `poc/`, conclusion documented.

### Spike 0: Competitive / data source due diligence — DONE

**Write-up:** [`poc/spike-0-data-sources.md`](poc/spike-0-data-sources.md)

| Question | Answer |
|---|---|
| CSM API free tier? | No. Contact / partnership. Not a live source. |
| CSM numeric dimensions via API? | Irrelevant — we would not republish them even if we had them. |
| Kids-In-Mind scrape? | ToS: copyrighted editorial; commercial reproduction prohibited. Don't. |
| Screenwise movie calibration? | No. Digital-habits survey. |
| VidAngel public scene data? | Not found. Stays v3. |

**Data strategy:** original scores in `data/scores.yaml`; TMDB for metadata/poster *paths*; never copy CSM/KIM numbers.

### Spike 1: Where do intensity scores come from? — DONE (pending review)

**Write-up:** [`poc/intensity_scores_poc.md`](poc/intensity_scores_poc.md)

Lowest-effort method that we can defend: **write the numbers ourselves against `data/RUBRIC.md`.**

Gold set (needs review): Luca, Up, Finding Nemo, The Incredibles, Toy Story 3.

Do not scale to 50 until those five shapes are accepted (Up = heavy spike, Incredibles = violence spike, TS3 = scare ceiling, Nemo = G-rated trap, Luca = floor).

### Spike 2: Rank-ordering with multiple intensity dimensions — OPEN

**Question:** Is `max(scary, language, violence, crude, heavy)` a good enough aggregation to produce a rank order that matches parent judgment?
How to include individual child's scores? is it `scary = base_scary - child_baseline_scary`

**Hypothesis from the gold set:** `max()` is an OK **uncalibrated sort** and a bad **filter**. Up and Toy Story 3 both max to 5 for different reasons. The product has to keep per-dimension thresholds (PLAN original). A parent who is fine with grief and not fire must see Up and not TS3.

**PoC approach (no app needed):**
- Take the reviewed gold set (then 10–15 movies)
- Print a rank order using `max()`, then `avg()`, then `weighted avg`
	- For weight, asking the parent for multipliers based on their kid relative to peers and the parent's own priorities: "Charlie is 3 y.o., but for fright is scare of movies like other 2 y.o. we see? Can handle sad scenes like peers. In our family, we want to see swearing reduced compared to typical media"
- Also print the **threshold rule**: fine-set → per-dimension max allowed; too-much set → at least one dimension over the line
- Sit down with 1–2 other parents who know these movies well
- Ask: "Does this ranking feel right? What would you swap?"
- Document which aggregation best matched intuition

**Questions to resolve:**
- Does `max()` over-penalize movies with one scary moment but otherwise gentle?
- Should heaviness be weighted less than fear for younger kids?
- Is there a child-age modifier? (sad matters more at 4, scary matters more at 7?)
- Calibration rule: threshold[d] = max(score[d] among "fine") , hide if any dimension exceeds? Or hide only if it exceeds a "too much" example on that same dimension?

**Conclusion to document:** Which aggregation method survives the parent sanity-test? Note any cases that all methods got wrong.

### Spike 3: TMDB poster fetch — OPEN

**Question:** Given a list of 50 movie titles, can I reliably get `tmdb_id` and `poster_path` from TMDB's free API with a one-time script?

**ToS notes (Spike 0):** attribution required; non-commercial; do not host/cache image files >6 months. Store `poster_path`, hotlink TMDB image URLs. No runtime API key needed for the static site.

**Output:** A `movies.yaml` snippet for 5 movies with `tmdb_id`, `poster_path`, and the reviewed scores.
**Conclusion to document:** Does TMDB's search return correct results for Disney titles? Any disambiguation failures (e.g., Cinderella 1950 vs 2015)?

---

## Tech Stack

- **HTML + Tailwind CSS (CDN)**: no build step, no Node toolchain required for MVP
- **Vanilla JS**: threshold filter logic; localStorage persistence
- **data/scores.yaml**: editorial scores. Compile to JSON at data-prep time if the browser shouldn’t parse YAML. No runtime API calls for scores.
- **TMDB API**: one-time use at data-prep time only (ids + `poster_path`). Images from `image.tmdb.org` at runtime.
- **Hosting**: GitHub Pages

**Dark mode anti-FOUC pattern:**

```html
<script>
  if (localStorage.theme === 'dark' ||
      (!('theme' in localStorage) && window.matchMedia('(prefers-color-scheme: dark)').matches)) {
    document.documentElement.classList.add('dark');
  }
</script>
```

Place this in `<head>` before any CSS link tags. Tailwind dark mode must be set to `'class'` mode.

---

## Movie Catalog Curation Notes

Two cohorts for initial list of ~50:

**Cohort 1 — Currently exciting to kids**
- Movies kids are actively asking to see now (theater releases, recent Disney+)
- Source: current box office + Disney+ new releases
- v0 stand-in: *Luca*

**Cohort 2 — Millennial nostalgia with "that one scene"**
- Movies adults remember fondly but forget had a traumatizing moment
- Examples: The Lion King (stampede), Finding Nemo (opening), Pinocchio (Pleasure Island), Bambi, Snow White, Fantasia
- v0 stand-ins: *Up* (Ellie), *Finding Nemo* (barracuda), *The Incredibles* (plane), *Toy Story 3* (incinerator)
- The whole point is surfacing these so a parent doesn't discover the scene alongside their 4-year-old

**v3 expansion:** Any movie by title. Terminator 1 and Terminator 2 would show different stats because their scene-level intensity differs significantly.

---

## Related Resources

- **Common Sense Media** — best existing *reference* for what a review considers; no public hobbyist API; do not scrape scores
- **Kids-In-Mind** — 1–10 scores for Violence/Gore/Profanity; copyrighted; do not scrape
- **TMDB API** — free for non-commercial metadata and poster paths; attribution required
- **VidAngel** (v3 spike) — scene-level skip data; investigate whether their data could drive scene-level intensity deltas

---

## Open Questions / Risks

- **Score accuracy**: v0 numbers are one editor’s judgment. Spot-check the gold set before expanding. Do not “fix” them by copying CSM.
- **Aggregation**: `max()` is the simplest default *sort* but collapses Up vs TS3. Needs parent sanity-test of the **threshold** rule (Spike 2).
- **Catalog size at launch**: 50 movies may feel thin. If calibration movies aren't in the list, the whole workflow fails. Gold set must be films parents actually name.
- **ToS**: TMDB allows non-commercial use with attribution and a 6-month cache cap. CSM/KIM are not score sources.
