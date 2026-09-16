# NOTE — FBVNK_SIM_LEDGER_KIT1 · REVISION 4

From: travel-map-da cloud session (subagent to FBVNK PROJECT MANAGER)
Job: Step 7 GO, per PM's KIT1 review message, read 22:15, 15 Sep 2026
Revision 4: answers K7c (PASS WITH NOTES, `REVIEW.md` md5 7adcb685) and folds in R17, Records'
aerodrome-coordinate study (`NOTE.md` md5 4dbd0a19). Revisions 1-3 stay frozen.

**KIT1 R4 REPORTED.** A new folder again, so the three before it stay frozen exactly as Records
reviewed them and the md5s K7, K7b and K7c verified still point at the bytes they saw.

## What revision 4 changes

| | K7c finding | What was actually wrong |
|---|---|---|
| **C1** | the touchdown clamp is inverted | **`LEG_CONTRACT.md` only. The book's code was already right.** See below — this is the important distinction, because the converter is being built from that document. |
| **C2** | the `aircraft` burn rung needs the refuel guard | **The book's code.** A real defect, fixed and pinned by V24. |
| **C3** | pro rata must round net-first | Leader decision. Applied; V25 and V26 pin it. |

### C1 — the text was wrong, the code was not

Revision 3 of `LEG_CONTRACT.md` said "0 or **positive** after negating → 0, labelled a float". Read
literally that clamps every normal landing to zero, because a normal landing *is* positive after
negating, and the hard-landing check would then never fire again. Records was right to stop on it.

**What the book does, checked on the branch before this note was written**, not asserted from the
source:

| touchdownFpm in | rate the book keeps |
|---|---|
| +180, a normal landing | **180** |
| +520 | **520** |
| +560 | **560**, and it raises *"Hard landing at 560 fpm — inspection required"* |
| 0 | 0, labelled a float |
| −180, a converter that forgot to negate | 0, and said out loud |

So the code clamps on **negative** and keeps positive, which is the correct rule, and it has done
since it was written. Only the prose had it backwards. The text is corrected here and the code is
untouched, because it did not need touching. Worth naming plainly: a document the Builder codes
from is as live as the code, and this one would have produced a converter that agreed with a
sentence rather than with the book.

### C2 — this one was the code

The `aircraft` rung reads `NK_LB_L_FUEL_ARR − NK_LB_L_FUEL_DEP`. Those are FP-5L **USED**
readings, and a refuel resets USED, so across a refuel the subtraction comes out negative or tiny —
V274 itself reports "no leg burn". Revision 3 let the rung fire anyway. A leg that burnt about 70 L
would have been billed as 9.

Fixed: the `aircraft` rung now carries the same `refuelInLeg` guard as `rem`. With a refuel and no
logger the book falls to the uplift; with no uplift either it bills nothing and says so. Verified
across the whole chain, and the `logger` rung still fires across a refuel — it sums increases
instead of subtracting endpoints, which is the entire reason it exists.

### C3 — net first, and why it hid

A pro-rata service now rounds in the same order as every other posting: the normalised fraction
applies to the **net**, that net is rounded, VAT comes off the rounded net, and gross is the sum.
Revision 3 said `price_gross × fraction`, which rounds VAT into the gross before the fraction and
can land a cent high.

It hid because **none of the three existing services moves**: V1 is 513.65, V15 is 1.80 and V16 is
4.02 under either order. It only shows at other point counts, which is why Records had to price a
one-point and a fifteen-point service to surface it:

| | net | VAT | gross | gross-first would give |
|---|---|---|---|---|
| **V25** brakes, 1 point | 10.89 | 2.50 | **13.39** | 13.40 |
| **V26** brakes, 15 points | 163.41 | 37.58 | **200.99** | 201.00 |

### R17 — the coordinates, and two codes that were swapped

Records' study lands the ARP for all eight fields from the national AIPs, each row carrying its AIP
string, its chapter and whether a second read confirmed it (LPCS, LEVD and LETL confirmed; the
other five single-read, and labelled so). The closest pair is LPCO–LPVL at 67.5 nm against the
book's 8 nm resolve radius, so nothing can be confused.

It also found that **the book had two Spanish codes swapped since the first build**: `LEAP` was
labelled Teruel and `LEMP` was labelled Empuriabrava. Records asked that this be checked against
the source the figures came from before anything moved, which was right, and the check found
something better than the AIP — **an invoice of Claus's own**:

> BP Energía España **2002864264**, 20 Nov 2025: *"Location of Delivery: **TEV - LETL - TERUEL**"*,
> billing F-BVNK for **155.000 L** uplifted on **24 OCT 25**, total **€366.14**.

366.14 / 155.000 = **€2.3622/L**, against the **2.36** the "Teruel" row has carried since the first
build. So the figures under that row really are Teruel's, and Teruel really is LETL. The row moves
to `LETL` on the strength of Claus's own paperwork rather than a web page.

`LEMP` is Los Martínez del Puerto in Murcia. Nothing of Claus's uses it — searched, one hit, which
is R17's own note — so the entry is **dropped** rather than left as an aerodrome the book pretends
to know. The Empuriabrava figures move to `LEAP`, but on the name alone: no invoice was found for
that aerodrome, so that move is Records' assumption and this session could not better it. Said here
rather than left to look equally solid.

**His planning was never wrong.** `LPCS - LETL` then `LETL - LEAP` on 24 Oct 2025 reads Cascais →
Teruel → Ampuriabrava, one route east across Spain to the Costa Brava. The book's table was wrong.
This did not need to go to Claus, and the R3 open item is closed rather than escalated. The book now
resolves that exact route from positions alone — verified — and posts his 155 L at €2.36/L at LETL.

### Two things about Claus's bookkeeping, raised because someone should know

Neither changes a figure in this kit, and neither is this kit's to settle.

1. **That Teruel invoice is not in the 2025 accounts file.** `F-BVNK_2025_Accounts_1.xlsx` holds 42
   invoices de-duplicated from a 106-page PDF, and `2002864264` is not among them. It sits in the
   THANNERHOME accounting folder under *Novembro*.
2. **It is VAT-exempt** — *"Transaction exempt from VAT and is treated as an export. Law 37/1992
   Art 22"* — while the two Spanish BP invoices that *are* in that file carry 21 %. The kit keeps
   21 % for Spain, because one exempt invoice does not establish a rule; flagged, not changed.

### Also fixed, per K7c

- **Provenance counts** were stale: 4 invoiced-with-id, **1** without (hangar), 3 calculator, **17**
  assumption. Revision 3 still carried the counts from before F1 moved brakes.
- **V16's id** no longer says `vat_21` — F1 made that line Portuguese and V18 carries the 21 % check.
- **V18's litres dropped.** It stated 111.88 L at 3.42, which is 382.63 and not the 382.50 printed.
  Dropped rather than back-solved to 111.842: the vector is about the VAT rate and the rounding, and
  fitting a litre figure to the answer would be the same mistake facing the other way.
- **The sim-rate note was wrong for TACH.** Revision 3 said every hours variable undercounts at sim
  rates above 1x and that the reserves under-bill because of it. TACH adds *unclamped* elapsed-time
  deltas; only ENG_HRS and AF_HRS take the 1-second-clamped tick. The reserves accrue on TACH, so
  they do not under-bill. (`assumption:` that elapsed time itself follows the sim rate.)
- **STRIP_SPEC line numbers** moved to the installed V274: `act()` 186, `this.set(item.key, 100)`
  192, the `MX_LOG` +1 at 202 — which is also the only place in the package that writes `MX_LOG`,
  so the battery-pad finding holds.
- **`LEG_CONTRACT.md`** gains the installed `NK_LB_*` names, the statement that **`burnL` has no
  stored variable** and how each rung derives it, the logger's leg boundaries (`lb_open` 0 → 1 to
  the `lb_n` increment), and the **year wrap**: a departure on day 365/366 whose arrival, take-off
  or landing crosses midnight belongs to `year + 1`.

**No field name has changed.** The names are exactly those of `90020d89`, the copy the Builder is
working to.

**Provenance, per K7 and Claus's standing rule.** The old two-way real/assumption split put a paid
invoice and a planning figure in the same bucket. Now three: `invoiced` (5 — of which 4 quote an
invoice id, and `invoice_id_cited` says which), `calculator` (3 — Claus's own 2025 cost calculator,
real documents of his but planning figures for work not yet done, so no invoice can exist),
`assumption` (17 — set for the sim, no source; brakes joined them in R3). The two reserve rates now cite the calculator
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
| `TEST_VECTORS.json` | Twenty-seven sequences (V1-V26, with V2 split into 2a/2b) of LocalVar reads → expected postings in euro. New this revision: **V18** the real Spanish invoice at 21 % and a `.xx5` rounding case, and **V19-V23** the `burnSource` fallback chain in order, including the two cases a mid-leg refuel breaks. **V6** now pins the rounding mode, **V14** the overhaul consuming its own step, **V16** the repriced brakes. |
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
`fbvnk-sim-ledger/KIT1_R4/` on `claude/fbvnk-cost-sheet-maintenance-svcsw2`, so the folder is
checkable against a durable copy. The PM checks them again on the laptop bytes per the usual
practice. **KIT1 R4 REPORTED** — the folder is frozen from here; Records reviews it as K7d.

Every euro figure in this revision was recomputed independently in Python `Decimal` with
`ROUND_HALF_UP`, the decided mode, not in float: V1 513.65, V6 31.19, V15 1.80, V16 4.02,
V18 80.33 VAT on a printed gross of 462.83, and the two new net-first services at 13.39 and 200.99.
The laptop book's own `eur2()` gives the same answers on all 23 of its test cases.
