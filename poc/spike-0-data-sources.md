# Spike 0 — Competitive / data-source due diligence

**Question:** Does any competitor make this redundant? Which data sources are legally and practically usable?

**Status:** answered, 2026-09-23.

## Competitor: is this redundant?

No. Nobody does “tell us 2–4 movies your kid already reacted to → infer per-dimension thresholds → filter a catalog.”

| Product | What it actually does | Calibration from known movies? |
|---|---|---|
| Common Sense Media | Editorial 0–5 dimension dots + a population age label | No. Age 5+ is not “your kid survived Moana and melted at Nemo.” |
| Screenwise | ~5 min survey of digital habits / screen time; compares you to local families | No. Not a movie product. |
| Kids-In-Mind | Three 0–10 clinical scores, read-only pages | No UI, no recs. |
| VidAngel | Skip/mute scenes on subscribed streams | Reactive, not “what should we put on.” No public scene API found. |
| JustWatch | Where to stream; MPAA filter | No sensitivity features. |

PLAN’s gap statement still holds.

## Data sources

### Common Sense Media — do not use as a score source

- Developer Center exists (`commonsensemedia.org/developers`). Access is “contact us,” not a self-serve hobbyist API.
- PLAN’s note that `api.commonsense.org/api/v3` is partnership-gated is consistent with that.
- Their dimension scores are original editorial work. Copying them into `scores.yaml` (or prompting a model to “rate like CSM”) is republishing their product.
- **Usable:** public facts from having seen the films ourselves (“Nemo opens with the mother eaten”).
- **Not usable:** their numeric dots, age labels, or review text as our data.

### Kids-In-Mind — do not scrape

[Terms](https://kids-in-mind.com/terms.htm): features are copyrighted; “any commercial reproduction of anything appearing within this site without our written permission is illegal.” Personal print/share/link is fine. Dumping S/V/L triples into our catalog is not.

Also missing **scary vs violence** and **heavy/sad**, which is the whole point of this product.

### TMDB — metadata and posters, not intensity

Usable for title, year, `tmdb_id`, MPAA certification, `poster_path`.

Constraints that change the PLAN slightly:

- Attribution required (logo + “uses TMDB and the TMDB APIs but is not endorsed…”).
- Non-commercial license unless we sign a separate agreement.
- **Do not cache API payloads / images longer than 6 months.** Baking downloaded poster *files* into the repo is a ToS problem.
- **Do** store `poster_path` and load `https://image.tmdb.org/t/p/w342/{poster_path}` at runtime. That is an image URL, not an API call, and matches “no runtime API key in the static site.”
- Do not use TMDB content to train a model.

Spike 3 (fetch IDs/paths for ~50 titles) is still open. Disambiguation (*Cinderella* 1950 vs 2015) is the real risk.

### LLM “rate like CSM” — not for published scores

Fast, and it will hallucinate *Coraline*-class edge cases. Fine as a *draft* that a human overwrites. The published number has to be original judgment against `data/RUBRIC.md`.

## Recommended data strategy

| Layer | Source | When |
|---|---|---|
| Intensity scores | Original, in `data/scores.yaml`, against our rubric | Now, film by film |
| Title / year / MPAA / poster_path | TMDB, one-time script, paths not hosted files | Spike 3 |
| Scene facts | Our knowledge of the films; optional public plot confirmation | As we write notes |
| CSM / KIM numbers | Never | — |

**MVP scores are not a scrape.** They are an editorial file we can defend.
