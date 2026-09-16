# NOTE — FBVNK_SIM_LEDGER_KIT1 · REVISION 3

From: travel-map-da cloud session (subagent to FBVNK PROJECT MANAGER)
Job: Step 7 GO, per PM's KIT1 review message, read 22:15, 15 Sep 2026
Revision 3: answers K7b (PASS WITH NOTES, `REVIEW.md` md5 1124762f), the PM's V274 install note,
and the Records addendum on the fuel chain. Revision 2 stays frozen.

**KIT1 R3 REPORTED.** A new folder again, so `FBVNK_SIM_LEDGER_KIT1/` and `FBVNK_SIM_LEDGER_KIT1_R2/`
both stay frozen exactly as Records reviewed them and the md5s K7 and K7b verified still point at
the bytes they saw.

## What revision 3 changes

| | Finding | Closed by |
|---|---|---|
| **F1** | The brake price is derived at the wrong VAT rate | Not the way K7b proposed. See below — it is bigger than a rate. |
| **F2** | No rounding mode stated, and V6 depends on it | `rounding` in the rules file; V6 is 31.19; V18 pins it a second time on a real invoice. |
| **F3** | An overhaul would also bill seven pro-rata services | `posting_cursor.rule_overhaul_consumes_the_step`; V14 extended with the conditions. |

### F1 — neither fix K7b offered is the right one

K7b was right that the entry is inconsistent: the net 1,089.43 is 1,340.00 / 1.23 while the entry
stored `vat_pct` 21. It offered net 1,107.44 at 21 %, or `vat_pct` 23. Both assume the invoice
exists.

It does not. This session read Claus's own 2025 accounts —
`BRIEFCASE/Aviation/RALLYES/F-BVNK - MS 893 E/AIRWORTHINESS - DOCS/EXPENSES 2025 COMBINED/F-BVNK_2025_Accounts_1.xlsx`,
42 invoices de-duplicated from a 106-page PDF — and **there is no Nicolau invoice in them, and no
brake overhaul at all**. The suppliers are BP Portugal Combustíveis, BP Energía España, Sevenair
Group and IAC Coimbra, plus Cascais Dinâmica for aerodrome fees. No Spanish maintenance shop.

So the entry could not stay `invoiced` whatever its VAT rate. It is an **assumption** now, at the
shop that does every other assumed job in the table, IAC Coimbra, Portuguese VAT 23 — which makes
the stored net and the 1,340.00 figure agree with each other for the first time. V16 posts **4.02**,
the figure K7b predicted for its 23 % branch. The 1,340.00 is kept rather than deleted, and labelled,
so nobody reads a silent change as a new fact.

**Everything else in the table was checked against those accounts while the file was open, and
matches to the cent:**

| Entry | Kit | Printed on the invoice |
|---|---|---|
| hangar 1,000.00 @ 23 % | 1,230.00 | 1,230.00, all 13 Sevenair invoices |
| cao `FAC 25/135` | 885.60 | 885.60 |
| insp25 `FAC 25/64` | 206.64 | 206.64 |
| insp100 `FAC 25/100` | 2,995.04 | 2,995.04 |
| insp50 `FAC 26/6` | 967.86 | 967.86, and a 2026 invoice as labelled |
| LPCS landing 22.24 | — | printed alone on three Cascais invoices |
| LPCS landing + 1 night 38.92 | — | the printed total, 28 Oct 2025 |

### The 21 % line had to move, so it moved somewhere better

F1 took the only 21 % line out of the kit, and that was the one thing that would catch a hard-coded
23. **V18** puts it back on a *real* invoice instead of an assumed price: BP Energía España
`2002747740`, 25 Apr 2025, net 382.50, VAT 80.33, gross 462.83 exactly as printed. It is a `.xx5`
case too (382.50 × 0.21 = 80.325), so it pins the rounding rule a second time. It is the only
vector in the kit checked against a figure printed on a real invoice rather than derived from the
price table.

### Three open items closed, and one of them bites

`SPEC.md` arrived this round, so:

- **The `NK_ST_FAULTS` bit map** — bits 0-7: static port, pitot, COM wiring, NAV wiring, landing
  light, taxi light, left magneto, right magneto. Bits 6 and 7 together are an engine failure in
  the wear model, not just two faults. Four of the eight have no price key at all.
- **Which hours source the reserves accrue on** — `NK_ST_TACH`. `NK_ST_ENG_HRS` is set to 0 by an
  overhaul and 300 by a Reset, both on purpose, so a delta on it is meaningless across either.
  `NK_ST_AF_HRS` is airborne time and nothing in the package reads it — worth knowing, because
  airborne time is what Claus's carnet totals.
- **The battery signal, and the catch.** `NK_ST_BAT_HEALTH` jumping to 100 with the four counters
  going to 0 is a new battery. But the ammeter pad is **not an iPad page action**, and `MX_LOG` is
  bumped only by page actions — so **a battery replacement never moves the posting cursor**, and a
  reader gated on `MX_LOG` alone would never see one. `NK_ST_BAT_HEALTH` has to be watched
  independently. That is the reason this item was worth closing rather than guessing at.

### Also fixed, per K7b

- The `fields` table's two-value `src` is documented rather than rewritten — moving a figure of
  Claus's into "assumption" because this session cannot find its invoice would be its own
  inaccuracy, the same reason the calculator figures were not moved in revision 2. Two LPCS figures
  *are* verified against the accounts; the rest are not verifiable here and say so.
- Stale text gone: V7's "strip inclusion still open", and the `nk_telemetry.py` line that V274's
  logger retires.
- **The oil unit rule**, written before a price exists. `NK_ST_OIL_QT` is quarts 0-8, not condition
  points: `quarts_restored / R`, and the posting is `price_per_quart ×` that. Dividing a quart
  figure by 100 would under-bill a full top-up by eight to one. Dormant until a price is set.
- The three FP-5L edge cases written into the fuel rule: unpowered burn arrives as one step and
  counts only under 2 L; any single drop of 2 L or more is ignored; litres burnt between the last
  confirmed read and a refuel are lost.

### Contract `FBVNK_LEG/2`

`LEG_CONTRACT.md`, new in this folder, is the whole thing in one file so the converter kit and
Records review against the same page. Positions instead of ICAO codes, Zulu times, a landing count,
the touchdown sign convention and its clamp, the `burnSource` chain, and V274's `noTakeOff` and
`stopInAir` flags. All of it is implemented and tested in the laptop book, not proposed:
`claude/fbvnk-cost-sheet-maintenance-svcsw2`.

Two consequences worth naming. An engine run that never got airborne is filed and posts but stays
**out of the carnet** — a ground run is not a flight, and the Logbook page says how many are being
left out. And a **negative** `touchdownFpm` is clamped to 0 and said out loud rather than quietly
flipped: it means either a converter that forgot to negate, in which case every landing comes
through soft and the hard-landing check silently never fires again, or an arrival with no descent.

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
| `PRICES_AND_BILLING_RULES.json` | The single price/rules source both the laptop ledger and the future HANGAR strip read. Prices transcribed verbatim from `FBVNK-LEDGER.html` commit `5e72853`, and in this revision checked against Claus's own 2025 accounts where an invoice exists for them. Wear-rate normalisation, posting-cursor, Reset-to-delivery, rounding and the hours/faults/battery rules cite `SPEC.md` (R5) and the K7/K7b reviews inline. |
| `TEST_VECTORS.json` | Twenty-four sequences (V1-V23, with V2 split into 2a/2b) of LocalVar reads → expected postings in euro. New this revision: **V18** the real Spanish invoice at 21 % and a `.xx5` rounding case, and **V19-V23** the `burnSource` fallback chain in order, including the two cases a mid-leg refuel breaks. **V6** now pins the rounding mode, **V14** the overhaul consuming its own step, **V16** the repriced brakes. |
| `LEG_CONTRACT.md` | **New.** Contract `FBVNK_LEG/2` in one file — every field, where V274 stores it, what the book does without it, the fuel chain, the touchdown clamp, the time base, and what the converter must derive. The converter kit starts from this page and Records reviews against it. |
| `STRIP_SPEC.md` | What the HANGAR strip shows, its scope, the rounding rule it must share with the laptop book, what pro-rata pricing costs it in new persisted state, and the three capture routes including the one Records found at the SERVICE action. No package code. |
| `MD5.txt` | `md5sum -c` format, this note included. |

## Not in KIT1

`nk_telemetry.py` and anything in the package (Builder's and the PM's), any download. This session
wrote four plain-text/JSON/Markdown files and this note — nothing else touched. Nothing was
installed and no package byte was read or written from here; the package facts in this folder are
the PM's and Records', cited to them.

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

Five, listed in full under `PRICES_AND_BILLING_RULES.json` → `open_items`. Three of revision 2's
six closed this round, off `SPEC.md`.

1. **Nine prices with no figure.** The five `PRICES` keys (carburettor, starter, oil pump/lines,
   exterior lamps, oil top-up per quart) and four of the eight fault components (static port,
   pitot, COM wiring, NAV wiring). A Claus decision.
2. **Which capture route the Builder takes** — the accumulator, the snapshot, or Records' capture
   at the SERVICE action. This session recommends the action capture *combined with* accumulation,
   because the action sees the exact value at the service while a mixed-R life still needs
   per-read division. Not chosen here.
3. **Aerodrome coordinates.** Eight pairs: LPCS, LPCO, LPVL, LEVD, LEAP, LEST, LEBG, LEMP. The
   contract has the aeroplane sending positions, so the book resolves them — but no coordinate is
   on file and this session will not put a position on a real aerodrome from recall.
4. **UTC or local in the carnet.** On the board. The book has a control either way.
5. **The `LEAP` / Teruel naming.** Claus's own flight planning has `LPCS - LETL` and `LETL - LEAP`
   for the same day, so at least one of the two names is wrong.

None of these were guessed at to fill a gap — each is flagged instead.

## Verification

`md5sum -c MD5.txt` from inside this folder should report all five files OK. This session computed
the same hashes locally before upload, and the same bytes are committed at
`fbvnk-sim-ledger/KIT1_R3/` on `claude/fbvnk-cost-sheet-maintenance-svcsw2`, so the folder is
checkable against a durable copy. The PM checks them again on the laptop bytes per the usual
practice. **KIT1 R3 REPORTED** — the folder is frozen from here; Records reviews it as K7c.

Every euro figure in this revision was recomputed independently in Python `Decimal` with
`ROUND_HALF_UP`, the decided mode, not in float: V1 513.65, V6 31.19, V15 1.80, V16 4.02,
V18 80.33 VAT on a printed gross of 462.83. The laptop book's own `eur2()` gives the same answers
on all 23 of its test cases.
