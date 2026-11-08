# SehatCheck — 🩺 Free public drug-safety lookup

**Type any medicine. See its reported side effects from FDA data, in plain language.**

SehatCheck is a free, public, no-signup drug-safety lookup powered by the
[openFDA API](https://open.fda.gov) — no API key needed. Search a brand or
generic name and get: total FAERS adverse-event reports, the top 10 reported
side effects, a serious-outcome breakdown (death / hospitalisation /
disability), and the FDA label's boxed warnings, warnings and dosage forms —
all explained in plain words.

An original clean-room build from the free-public-APIs startup series
(public-apis/public-apis — 1,359 no-key APIs). It is the **public**
version of the drug-safety idea: the unlisted MediScribe research
prototype (`/drugsafety`) stays a prototype; SehatCheck is the simple,
public, standalone one.

## What it does

- **Drug search** — brand (Panadol, Brufen, Risek, Augmentin…) or generic
  (paracetamol, metformin, amoxicillin…). Pakistani brands are mapped to
  generics; INN names are mapped to USAN (paracetamol → acetaminophen).
- **FAERS summary** — total reports, serious reports, and the three
  serious outcomes separately, each with its share of reports.
- **Top 10 reported effects** — ranked by report count, with a bar chart.
- **FDA label** — boxed warning (highlighted), warnings, adverse
  reactions, dosage & administration, dosage forms, brand names.
- **Plain-language reading** — every number is explained ("shares of
  reports, not your personal risk").
- **JSON API** — `GET /sehatcheck/api/lookup?q=metformin` returns the
  same data for reuse.

## Honest limits (shown on every page)

- FAERS reports are **voluntary** — a report is not proof the drug caused
  the event. **Correlation ≠ causation.**
- Serious outcomes are heavily **over-reported**; mild ones mostly are not.
- Outcome counts **overlap** — one report can list death *and*
  hospitalisation, so they do not add up to the serious total.
- Data is **US-centric** (2004 → today). Pakistan has no public
  adverse-event API.
- **NOT medical advice** — ask a pharmacist or doctor.
- On API failure or no results the page says so plainly. Nothing is
  invented. (One real catch found while building: openFDA's `serious:1`
  means serious and `serious:2` means *non*-serious — verified live;
  SehatCheck uses `serious:1`.)

## Verified live data (2026-11-08, openFDA `last_updated` 2026-10-28)

| Drug | Total reports | Serious | Death | Hospitalisation | Disability |
|---|---|---|---|---|---|
| Acetaminophen | 497,983 | 426,113 | 103,026 | 159,923 | 26,980 |
| Metformin | 439,327 | 298,232 | 36,501 | 156,096 | 11,995 |
| Amoxicillin | 116,648 | 97,080 | 8,613 | 46,675 | 3,416 |

Top reported effect per drug: acetaminophen — DRUG DEPENDENCE (51,264);
metformin — NAUSEA (30,628); amoxicillin — DIARRHOEA (7,039).

## Run it

SehatCheck lives as a route (`/sehatcheck`) inside the clinic-demo
FastAPI app. The module here is self-contained and works in both layouts
(`sehatcheck` package or `app.sehatcheck`):

```bash
# tests (fixtures are real openFDA responses saved 2026-11-08)
python -m pytest tests/ -q
# live smoke test (hits the real openFDA API)
SEHATCHECK_LIVE=1 python -m pytest tests/test_live_smoke.py -q
```

## Files

- `sehatcheck/engine.py` — openFDA client, parsing, brand/INN mapping, cache
- `sehatcheck/pages.py` — HTML templates (home, results, error)
- `sehatcheck/routes.py` — FastAPI routes (`/sehatcheck`, `/sehatcheck/search`, `/sehatcheck/api/lookup`)
- `tests/fixtures/` — real openFDA responses (trimmed for size)

## Deploy status

Built 2026-11-08. Route committed to clinic-demo; **deploy is blocked**
pending the owner's FastAPI Cloud sign-in (login expired — Khulasa, Dak
and Sukoon are queued behind the same sign-in).
