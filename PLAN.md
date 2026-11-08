# SehatCheck Public — build plan (prepared 2026-11-08 ~12:30 PKT)

## Order
User, 2026-11-08 11:47 PKT (main chat): "SehatCheck Public banao — openFDA drug lookup".
This supersedes the 06:34 UTC pending shortlist that ranked ZakatCalc / DawaCheck
first — the user picked SehatCheck (option 3 from the public-apis shortlist).

## What it is — and what it is NOT
- Public, no-login, Urdu-first drug safety lookup: type a brand (Pakistani or
  international) or generic → FAERS adverse-effect counts + FDA label sections.
- NOT a rebuild of Drug Safety Lookup, and NOT DawaCheck:
  - `/drugsafety` is already LIVE with the same openFDA backend (see Reuse).
  - DawaCheck (`~/workspace/dawacheck/BRIEF.md`) is price comparison (DRAP MRP
    vs pharmacy prices) — a different product; do not merge price data in v1.

## Live-verified endpoints (checked 2026-11-08 ~12:30 PKT, all keyless, no signup)
- openFDA drug/event: `api.fda.gov/drug/event.json?search=patient.drug.medicinalproduct:paracetamol`
  → total 111,341 reports, last_updated 2026-10-28. Works.
- openFDA drug/label: `api.fda.gov/drug/label.json?search=openfda.brand_name:Panadol`
  → total 11, last_updated 2026-11-06. Works.
- RxNorm: `rxnav.nlm.nih.gov/REST/rxcui.json?name=paracetamol` → RxCUI 161. Works.
- DailyMed: `dailymed.nlm.nih.gov/dailymed/services/v2/drugnames.json?drug_name=paracetamol`
  → HTTP 200. Works.
- (PICKS.md had flagged these as NOT live-verified; they now are — for SehatCheck's four.)

## Reuse — do not rebuild from scratch
- Backend: `~/workspace/clinic-demo/app/drugsafety/drugsafety.py`
  (`normalize()` + `lookup()`, 1-hour in-memory cache, 10s timeout,
  optional `OPENFDA_API_KEY` env for higher limits; `pk_brands.json` = 60 PK
  brand→generic mappings; INN→USAN map for paracetamol/salbutamol/adrenaline).
- UI patterns: `~/workspace/clinic-demo/app/drugsafety/routes.py`
  (search page, results with top-reactions bars, boxed warning, print/PDF
  report route, PWA manifest + service worker, disclaimer).
- New route: `/sehatcheck` in clinic-demo (mounted in `app/main.py`, same
  pattern as line 84 `app.include_router(drugsafety_router)`), reusing
  `DS.normalize` / `DS.lookup`.

## What is actually new in v1 (differentiation vs /drugsafety)
1. Urdu-first / Roman-Urdu UI (Urdu labels + Roman-Urdu search tolerant),
   public landing page written for sharing (SEO title/meta, shareable
   `/sehatcheck/search?q=...` links).
2. RxNorm name normalisation before lookup — fixes misspellings and unknown
   brands instead of only the hardcoded 60-brand map.
3. DailyMed full label sections added: dosage & administration,
   contraindications, drug interactions (beyond the two sections /drugsafety
   shows: adverse reactions + warnings).
4. Keep the honest framing verbatim: "Educational demo, not medical advice.
   FAERS = voluntary reports, reporting ≠ proven causation, US-centric
   2004→today, confirm with official label + pharmacist/doctor."
   No Pakistan adverse-event API exists publicly — say so, as /drugsafety does.

## Build steps (main agent)
1. New `app/sehatcheck/` package: routes + templates reusing `DS` backend;
   tests: paracetamol, Panadol, Brufen, metformin searches return 200 + counts.
2. GitHub push per standing rule (public repo under zubair0153715).
3. Deploy to clinic-demo `/sehatcheck` — BLOCKED until his FastAPI Cloud
   sign-in (same expired login queuing Sukoon, Khulasa, Dak). Build + push
   now; deploy after he signs in. Backup LabLink before any deploy (standing rule).

## Open decisions for the user (one line each)
- Name/route: `/sehatcheck` (proposed) vs extend `/drugsafety` in place?
- Language default: Urdu-first UI, or English UI with Urdu toggle like Hotel PMS?
