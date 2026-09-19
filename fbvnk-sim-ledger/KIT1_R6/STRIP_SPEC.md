# STRIP_SPEC — FBVNK_SIM_LEDGER_KIT1

What the HANGAR-page ledger strip shows, and the new persisted state it needs. No package code —
this is a spec for the Builder to turn into `HANGAR.js` and `systems.cfg [LocalVars]` bytes,
append-only, byte-proven, per CLAUDE.md.

Route: option C from the Job Card ("Three ways to wire it"). HANGAR already shows part rows, the
ENG/TT header, battery and FAULTS. The strip adds the one thing it doesn't: a running euro total
and cost per hour, priced from `PRICES_AND_BILLING_RULES.json` at build time, md5-pinned to it.

## What the strip shows

| Field | Format | Source |
|---|---|---|
| Running total | `€N,NNN.NN` | Accrued euro, variable-cost items only, fuel included (see scope below) |
| Cost per hour | `€NNN.NN/h` | Running total ÷ hours flown since the strip's own baseline |
| Scope footnote | `excl. hangar, insurance, CAO` | Fixed text — see "Strip scope" below |
| Fuel | folded into the running total, not a separate line | `L:NK_ST_FP5L_USED`, positive-delta accumulation — see "Fuel" below |

Not shown on the strip: individual invoices, VAT, the invoiced/assumption split, the Year Book
comparison. Those stay in the laptop ledger, which keeps the full book. The strip is the one
number HANGAR is missing, not a second copy of the ledger.

## Strip scope — DECIDED

Board item `ledger_fixed_costs_book`, Claus, 15 Sep 22:32:22, relayed word for word by the PM:
"Keep them in the laptop book, out of the cockpit figure (as designed)".

Running total = fuel + aerodrome fees + scheduled 25/50/100 h checks + unscheduled repairs +
engine reserve (€20.79/h) + propeller reserve (€1.36/h). Hangar, insurance and CAO post to the
laptop book in full and never reach this total. Footnote reads `excl. hangar, insurance, CAO`
exactly as designed — this is confirmed, not a placeholder pending an answer.

## Fuel — DECIDED

Board item `ledger_fuel_in_strip`, Claus, 15 Sep 22:34:50, relayed word for word by the PM:
"Yes: include fuel from the FP-5L USED reading (recommended)".

Full rule in `PRICES_AND_BILLING_RULES.json` → `fuel_source.strip_accumulation_rule`; summary:

- Read `L:NK_ST_FP5L_USED` (litres) on every confirmed cursor read (same (MX_LOG, TACH) gate as
  every other posting — see "New persisted state" below).
- `delta = current L:NK_ST_FP5L_USED − L:NK_LG_FUEL_BASELINE_L`.
- `delta > 0` → add `delta` to `L:NK_LG_FUEL_L_TOTAL`. This is normal consumption.
- `delta <= 0` → add nothing. A drop means FP5L.js reset the counter at a refuel (total fuel rose
  more than 0.5 L in one update) — never post a negative cost for that.
- Fold `L:NK_LG_FUEL_L_TOTAL × 2.30` (the Ramp tab's live default price, pinned at build time —
  see `fuel_source.strip_accumulation_rule.price`) into the running euro total. No separate fuel
  line in the strip's display: one running total stays the design (see "What the strip shows"),
  the same way the strip doesn't itemise any other category. Records/PM can ask for a visible
  fuel sub-line instead; this file's default is folded-in.
- `L:NK_LG_FUEL_BASELINE_L` advances to the current reading on every confirmed (non-reload) read —
  regardless of whether that read added litres or not. Skipping the update on a non-positive delta
  would leave the baseline stuck at its pre-refuel value and silently swallow every litre burned
  afterward, since the next delta would still come out negative against the stale baseline. A
  reload is the only thing that leaves the baseline untouched, exactly like
  `L:NK_LG_BASELINE_MXLOG`/`L:NK_LG_BASELINE_TACH`.

The laptop book's own fuel billing (Ramp tab, FP5L_USED-before-uplift with the typed uplift as
fallback) is unchanged by any of this — the strip keeps a separate running litres/euro figure.

## New persisted state this strip needs

The strip is a separate reader from the laptop ledger's browser `localStorage` — it needs its own
persisted cursor so it can post independently and never double-count against the laptop book.

**Collision check: done, none.** The PM verified against the package (16 Sep) that not one of the
seven `L:NK_LG_*` names below appears anywhere in it. So every one of them that must survive a
session needs its own `systems.cfg [LocalVars]` entry — package bytes, which means a Builder kit
and the PM's install, not this session's work. Proposed set, for Records and the Builder to accept
or adjust.

**What the strip reads already exists** — also PM-verified in the package, no new SimVar and no new
reader needed:

| Reads | LocalVar index | Used for |
|---|---|---|
| `NK_ST_FP5L_USED` | 10 | the fuel figure |
| `NK_ST_TACH` | 11 | cost per hour, and half the posting cursor |
| `NK_ST_MX_LOG` | 61 | the other half of the posting cursor |
| `NK_ST_WEAR` | 58 | the R in force, for normalisation |

That settles the three figures the strip shows: the running total, the cost per hour and the fuel
line are all derivable from variables in the package today. Only the persistence below is new.

| Proposed variable | Unit | Saved? | Meaning |
|---|---|---|---|
| `L:NK_LG_TOTAL_EUR_X100` | integer, euro × 100 | yes | Running total, fixed-point to avoid float drift on money. Divide by 100 to display. |
| `L:NK_LG_BASELINE_MXLOG` | integer | yes | The `NK_ST_MX_LOG` value the strip last posted against — the strip's own half of the posting cursor (see `posting_cursor` in `PRICES_AND_BILLING_RULES.json`), independent of the laptop ledger's cursor. |
| `L:NK_LG_BASELINE_TACH` | hours | yes | The `NK_ST_TACH` value paired with the above — same reload/new-install detection rules apply, evaluated independently by the strip. |
| `L:NK_LG_HOURS_BASELINE_TACH` | hours | yes | `NK_ST_TACH` value the strip is measuring "cost per hour" from. Reset only on an explicit pilot action (see below), never automatically — same principle as the new-install cursor rule. |
| `L:NK_LG_FLAG_MIDLEG_RATE` | boolean/bit | no (session only) | Set when a posting step's `wear_rate_normalisation.rule_r_changes_mid_leg` condition fires. Not shown on the strip itself in this design — the laptop ledger surfaces it for review. Listed here so the Builder doesn't silently drop the signal. |
| `L:NK_LG_FUEL_L_TOTAL` | litres | yes | Running fuel total accumulated from positive `L:NK_ST_FP5L_USED` deltas only. See "Fuel" above. Stored as litres, not euros, so a future price rebuild never needs to re-derive history. |
| `L:NK_LG_FUEL_BASELINE_L` | litres | yes | `L:NK_ST_FP5L_USED` as of the last confirmed (non-reload) read. Advances in lockstep with `L:NK_LG_BASELINE_MXLOG`/`L:NK_LG_BASELINE_TACH`, never independently. |

## Pro-rata pricing — DECIDED, and what it costs the strip

Board item, Claus, 16 Sep 2026 14:47:46 local, relayed word for word by the PM:
"Pro rata - bill only the wear that would have happened at REAL".

A service now posts `price_gross × (normalised_points / 100)` rather than the catalogue price.
Full rule in `PRICES_AND_BILLING_RULES.json` → `wear_rate_normalisation.DECIDED_how_a_normalised_loss_becomes_euro`.

That needs something the strip did not previously keep: how much of the part's life the service
restored. A service is detected by the condition reading 100, which is after the fact, so the
pre-service value has to have been persisted. Two designs, both fully specified in
`PRICES_AND_BILLING_RULES.json` → `wear_rate_normalisation.capture_requirement`:

| Design | New persisted LocalVars | Exact when |
|---|---|---|
| Accumulator — normalise each read as it happens | 22 | always, including across a rate change |
| Snapshot — keep last read's condition, divide at service | 11 | only while R holds since the last service |

Records added a third route in K7b, and it is better than either: **capture at the SERVICE action
itself**. `HANGAR.js` `act()` already calls `this.L()` in the same function, so immediately before
`this.set(item.key, 100)`, `this.L(item.key)` holds the exact pre-service condition and
`this.L("NK_ST_WEAR")` holds the R at that instant. That is the value *at* the service, where both
polling designs see only the value at the last confirmed read and miss whatever wore in between.
It writes `L:NK_LG_*` only and no `NK_ST_*`, so the "reads the wear model, never writes it"
boundary holds.

**Line numbers, against the installed V274** (K7c; they moved from V273): `act()` at **186**,
`this.set(item.key, 100)` at **192**, and the `MX_LOG` +1 at **202** — which is also the only place
in the package that writes `MX_LOG`, so the battery-pad finding in the rules file holds.

It replaces the snapshot, not the accumulator: a part whose life spanned more than one R still
needs per-read accumulation to divide correctly. This file recommends **the action capture
combined with accumulation** and does not choose — a rate change between two services is the
normal case, not an edge one, because R is a slider Claus moves. Records/Builder decide.

## Rounding — DECIDED

K7b F2, the PM, 16 Sep 2026. Round half-up on the exact decimal value, to the cent, once per
posting line. Never `toFixed`, never `Math.round` on a float: 31.185 is stored just below .185 as
a binary double, so `(31.185).toFixed(2)` gives 31.18 where the rule gives 31.19. Within a posting:
net to the cent first, VAT off the rounded net, gross as the sum of the two, so the invoice adds up
to its own printed lines. **A pro-rata service is no exception** (K7c C3): the normalised fraction
applies to the *net*, never to the gross — `price_gross × fraction` lands a cent high at some point
counts, 13.40 against 13.39 on a one-point brake service. Full rule in `PRICES_AND_BILLING_RULES.json` → `rounding`. **The strip
must use the same routine as the laptop book**, or the two disagree by a cent on every `.xx5`.

Either way the eleven variables to read — `NK_ST_C_ALT`, `NK_ST_C_BRK`, `NK_ST_C_CARB`,
`NK_ST_C_CYL`, `NK_ST_C_LTS`, `NK_ST_C_MAG`, `NK_ST_C_OIL`, `NK_ST_C_STR`, `NK_ST_C_TYR`,
`NK_ST_C_VAC`, `NK_ST_OIL_QT` — all exist in the package today, as does `NK_ST_WEAR` (58) for the
divisor. No new SimVar and no new reader. What is new is persistence, and persistence is
`systems.cfg [LocalVars]` bytes: a Builder kit and the PM's install, not this session's work.

The running total and cost per hour the strip shows are unaffected in shape — only the euro that
goes into them changes, and it changes downward at any R above 1.

`L:NK_LG_*` namespace chosen to match the project's existing `NK_` prefix convention (`SPEC.md`)
while staying visibly separate from the wear model's own `NK_ST_*`/`NK_W_*` variables — the strip
reads those, it must never write them.

## Pilot actions the strip needs (mirrors the laptop ledger's manual controls)

- **Confirm new baseline** — the strip-side equivalent of the laptop ledger's "pilot confirms it
  in the ledger" step for a new install/new machine (`PRICES_AND_BILLING_RULES.json` →
  `reset_to_delivery.new_install_case`). Re-baselines `L:NK_LG_BASELINE_MXLOG` /
  `L:NK_LG_BASELINE_TACH` to the current read. Never automatic.
- **Reset hours baseline** — re-baselines `L:NK_LG_HOURS_BASELINE_TACH` only, for starting a new
  "cost per hour" period without touching the money cursor above. Distinct button from the one
  above; conflating them would let a new-install reset also zero out an in-progress cost/hour
  figure the pilot didn't ask to clear.

## What this file does not decide

- The exact MSFS Coherent GT/JS mechanics of reading `L:` vars or laying out the strip visually —
  Builder's call, this is a data/behaviour spec only.
- Where on the HANGAR page the strip sits — Builder/PM, not scoped here.
- Whether fuel gets its own visible line instead of folding into the total (see "Fuel" above) —
  this file's default is folded-in; flag it if a separate line is wanted instead.

## The accrual clock — DECIDED, and the strip does not change

Added in revision 5, after the PM asked this session to confirm the reserve accrues on TACH.

**The strip accrues on `NK_ST_TACH` deltas, which is right for the strip** — it reads LocalVars in
the cockpit and TACH is the only hours variable that never goes backwards on purpose.

**The laptop book does not.** It accrues on `leg.hours`, which is airborne time, and so do the wear
model, the sinking funds and the 25 / 50 / 100-hour inspection counters. So the two halves of this
project bill the same aeroplane on different clocks.

Measured off page 86 of Claus's own Carnet de Route, not estimated: 7.9200 tacho hours against
6.6667 flown over the same eight legs — **0.1567 h per leg, 18.80 % more**. At the kit's own rates
of €20.79 engine plus €1.36 propeller, €22.15 per hour, that is **€27.76 over those eight legs,
€3.47 per leg, and €303.98 a year** on Claus's real 2025 flying of 73.00 hours. The inspection
counters carry the same difference: 100 flown hours is 118.80 tacho hours, so an inspection due on
tacho arrives **15.82 flown hours late** on the book's basis.

**Claus answered on the Annunciator at 21:05:42 local, 16 Sep 2026, word for word:**

> "Tacho time (the hour meter)"

**The strip does not change.** It was already reading `NK_ST_TACH` (LocalVar 11) under the
`(MX_LOG, TACH)` cursor it already has, and that was right all along. **The laptop book moved to
the meter in revision 6**, so there is now one clock across both halves.

One thing the book does that the strip does not need to: the book sees *records*, and under V275
only airborne legs become records, so it accrues the meter's **progression between records** rather
than a leg's own span — otherwise every minute of ground running between two legs would be lost.
The strip reads the LocalVar directly and continuously, so the question does not arise for it.

Airborne time now keeps the carnet's Heures de Vol and the airborne totals, and nothing else.
