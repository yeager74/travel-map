# NOTE — FBVNK_SIM_LEDGER_KIT1 · REVISION 2

From: travel-map-da cloud session (subagent to FBVNK PROJECT MANAGER)
Job: Step 7 GO, per PM's KIT1 review message, read 22:15, 15 Sep 2026
Revision 2: answers K7 (PASS WITH NOTES, `REVIEW.md` md5 8d3943e6), and Claus's two board answers
of 16 Sep 2026 — pro-rata billing, and staging in a new folder

Staged here on Claus's own instruction, board item, 16 Sep 2026 14:47:05 local, relayed word for word
by the PM: "Yes - stage it in a new folder in FBVNK_STAGING". `FBVNK_SIM_LEDGER_KIT1/` stays frozen
exactly as Records reviewed it, so the md5s K7 verified still point at the bytes K7 saw.

## What revision 2 changes

**Two gaps K7 asked to close before the strip is built — both closed.**

- **V14, overhaul.** The only event that draws the engine reserve down, and the largest posting in
  the book. Pins the underfunded case: a 30,000.00 fund against a 35,000.00 overhaul leaves the
  fund at 0, not negative, and the full 35,000.00 still posts. Also pins the discriminator against
  Reset to delivery — `ENG_HRS` 0 is an overhaul, 300 with the full seed set is a Reset.
- **V15, the normalisation actually dividing, at R=1000.** K7 was right that nothing exercised this:
  V1 is R=1 (divisor 1, a no-op), V6 is R=0 (short-circuits), V5 is flagged rather than priced. So
  the arithmetic pick 2 is entirely about had never been pinned by a vector.

**Three lower-priority notes, also closed.** V16 covers the scaled-by-R group priced rather than
flagged, a second dividing R, and the only 21 % VAT line in the table (brakes, Nicolau, Valladolid)
— the single vector that would catch a hard-coded 23. V17 covers the starter's `min(R,100)` cap and
lands on no-price-on-file.

**The contradiction found while writing V15 — settled by Claus, and this revision carries one reading.**
Revision 1 said two different things about how a normalised loss becomes euro: `wear_rate_normalisation`
described dividing the observed loss by R and billing that, while `events.service` said post the full
catalogue price. Every vector computed the full price, so K7 verified the arithmetic of one reading while
the stated rule described the other. It was put to Claus rather than settled here.

He answered on the Annunciator board, 16 Sep 2026 14:47:46 local, relayed word for word by the PM:
**"Pro rata - bill only the wear that would have happened at REAL"**. The full-price reading is gone from
this folder. A service now posts `price_gross × (normalised_points / 100)`, where `normalised_points` is
the depletion the service restored divided by the R that produced it.

What that does to the three affected vectors, recomputed from the prices in this folder:

| Vector | Part | R | Condition at service | Posts |
|---|---|---|---|---|
| V1 | magneto, 885.60 gross | 1 | 42 | **513.65** |
| V15 | cylinder, 2,767.50 gross | 1000 | 35 | **1.80** |
| V16 | brakes, 1,318.21 gross | 100 | 70 | **3.95** |

**One figure was wrong and the decision is what exposed it.** V15 stated 50 observed loss points for a
cylinder going from 35 to 100, which is 65. Under the full-price reading the number changed nothing, so it
survived review; under pro rata it *is* the posting. It is corrected here: 1.80, not the 1.38 the
side-by-side listed.

**The data gap, closed as far as this session can close it.** A pro-rata posting needs the condition
before the service, and a service is detected by the condition reading 100 — by which time the pre-service
value is gone unless something kept it. The PM asked for the exact variables. They are named in
`PRICES_AND_BILLING_RULES.json` → `wear_rate_normalisation.capture_requirement` and summarised in
`STRIP_SPEC.md`. Eleven reads, every one an existing package variable: `NK_ST_C_ALT`, `NK_ST_C_BRK`,
`NK_ST_C_CARB`, `NK_ST_C_CYL`, `NK_ST_C_LTS`, `NK_ST_C_MAG`, `NK_ST_C_OIL`, `NK_ST_C_STR`,
`NK_ST_C_TYR`, `NK_ST_C_VAC`, `NK_ST_OIL_QT`, plus `NK_ST_WEAR` (58) for the divisor. Two designs for
persisting them — an accumulator at 22 new LocalVars, exact across a rate change, or a snapshot at 11,
exact only while R holds since the last service. This session recommends the accumulator and has not
chosen: that is a Builder/Records call, and it is logged as an open item.

For the laptop book there is no persistence problem. It takes legs as records now (contract
`FBVNK_LEG/1`), so a record carrying a service carries `conditionBefore` and `conditionAfter` keyed by
those same eleven names, plus the R in force. The aeroplane holds the snapshot and sends it.

**Provenance, per K7 and Claus's standing rule.** The old two-way real/assumption split put a paid
invoice and a planning figure in the same bucket. Now three: `invoiced` (6 — of which 4 quote an
invoice id, and `invoice_id_cited` says which), `calculator` (3 — Claus's own 2025 cost calculator,
real documents of his but planning figures for work not yet done, so no invoice can exist),
`assumption` (16 — set for the sim, no source). The two reserve rates now cite the calculator
instead of nothing. And K7's catch on `insp50` is fixed: `FAC 26/6` is a 2026 series number and is
labelled as such rather than implied to be a 2025 invoice.

Moving the five real-but-uninvoiced figures to "assumption" would have been its own inaccuracy — it
would imply this session invented 35,000.00 for an engine overhaul when that is Claus's own number.

**Strip.** The PM verified in the package that `NK_ST_FP5L_USED` is LocalVar 10, `TACH` 11,
`MX_LOG` 61 and `WEAR` 58 — so the total, the cost per hour and the fuel figure are all derivable
from what exists today, with no new SimVar and no new reader. He also verified that none of the
seven proposed `L:NK_LG_*` names exists anywhere in the package, so each one that must survive a
session needs a `systems.cfg [LocalVars]` entry. That is package work: a Builder kit and the PM's
install, not this session's. Both facts are recorded in `STRIP_SPEC.md`.

**Unchanged by design.** Oil top-up, fault repair, battery replacement, propeller overhaul and the
detected scheduled check stay as "no posting, raise the flag", per the PM.

## What's in this folder

| File | What it is |
|---|---|
| `PRICES_AND_BILLING_RULES.json` | The single price/rules source both the laptop ledger and the future HANGAR strip read. Prices transcribed verbatim from `FBVNK-LEDGER.html` commit `5e72853`. Wear-rate normalisation, posting-cursor and Reset-to-delivery rules transcribed from the reviewed Job Card (rounds 2-3, checked against `SPEC.md` md5 `c06f7305`) and this KIT1 review message. |
| `TEST_VECTORS.json` | Eighteen sequences (IDs V1-V17, with V2 split into 2a/2b) of LocalVar reads → expected postings in euro, covering: a normal action, both shapes of reload/crash, Reset to delivery, new install/new machine, a rate change mid-leg (flagged, not guessed), Wear rate OFF, the laptop book's refuel read via FP-5L, a service event on a part with no price yet, one marked open pending a missing variable, and four covering the strip's own fuel accumulation (no refuel, a refuel mid-sequence, a reload freezing the fuel baseline, and a price change between legs). |
| `STRIP_SPEC.md` | What the HANGAR strip shows, its scope, what pro-rata pricing costs it in new persisted state, and this session's proposed `L:NK_LG_*` variables (none exist yet — proposal for Records/Builder to accept or adjust). No package code. |
| `MD5.txt` | `md5sum -c` format, this note included. |

## Not in KIT1 (per the PM's message)

`nk_telemetry.py` (Builder's file), anything in the package, any download. This session wrote
three plain-text/JSON/Markdown files and this note — nothing else touched.

## Resolved this round

- **"The fpl5" identified**: the Electronics International FP-5L fuel-flow/pressure instrument,
  `html_ui/Pages/VCockpit/Instruments/AH/FP5L/FP5L.js` (md5 `ed8f5deb`). Full detail in
  `PRICES_AND_BILLING_RULES.json` → `fuel_source`.
- First-load defaults correction: MX_LOG 0 (not the Reset-to-delivery "+1"), TACH 0 then seeded to
  1246.58. Full detail in `PRICES_AND_BILLING_RULES.json` → `reset_to_delivery.new_install_case`.
- **Both board items Claus was asked, tapped and closed** (PM relay, word for word):
  - `ledger_fixed_costs_book`, 15 Sep 22:32:22: "Keep them in the laptop book, out of the cockpit
    figure (as designed)". Hangar/insurance/CAO stay laptop-only, exactly as this folder already
    built it. `PRICES_AND_BILLING_RULES.json` → `cockpit_strip_scope`, `STRIP_SPEC.md` → "Strip
    scope".
  - `ledger_fuel_in_strip`, 15 Sep 22:34:50: "Yes: include fuel from the FP-5L USED reading
    (recommended)". New rule added this round: the strip accumulates positive
    `L:NK_ST_FP5L_USED` deltas into a persisted litres total, priced at the Ramp default
    (2.30 EUR/L) and folded into the running euro figure; a drop (refuel) never posts a negative
    cost, and the whole thing is gated by the same (MX_LOG, TACH) reload cursor as everything
    else. `PRICES_AND_BILLING_RULES.json` → `fuel_source.strip_accumulation_rule`, `STRIP_SPEC.md`
    → "Fuel", `TEST_VECTORS.json` → V10-V13.

## Open — needs a person, not a guess

Six open technical items, listed in full under `PRICES_AND_BILLING_RULES.json` → `open_items`:
the `NK_ST_FAULTS` bit-to-component map, the exact hours-since-last-check variable, which hours
source (`NK_ST_TACH` vs `NK_ST_ENG_HRS`) the reserve accrual should read, the battery-replacement
detection signal, and five `PRICES` keys with no price yet (carburettor, starter, oil pump/lines,
exterior lamps, oil top-up), and which of the two pre-service capture designs the Builder takes.
None of these were guessed at to fill a gap — each is flagged instead.

## Verification

`md5sum -c MD5.txt` from inside this folder should report all four files OK. This session
computed the same hashes locally before upload; PM checks them again on the laptop bytes per the
usual practice, and the folder freezes once "KIT1 REPORTED" is posted on the Job Card.
