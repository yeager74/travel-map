# LEG_CONTRACT — `FBVNK_LEG/2`

> **Revision 4 of the kit corrects one rule in this file and adds one optional field.** The
> corrected rule is the touchdown clamp, which was stated backwards here. The added field is
> `notes[]`, the converter's own notes, which the book now reads and shows on the review page —
> it was being accepted and silently dropped. **No existing field name has changed**: every other
> name is exactly that of `90020d89`, the copy the Builder is working to.

The record the laptop book reads, one per leg. Revision 2 of the contract, rewritten after
Records read revision 1 against what V274 actually stores (K7b §6) and after the PM's V274
install note of 16 Sep 16:4x.

**Who writes it.** A converter on the laptop, from a logger run folder to a `LEG.json` in these
field names. Builder's kit, after V274. The aeroplane itself writes LocalVars and logger datums,
not this JSON.

**Who reads it.** `FBVNK-LEDGER.html`, the laptop book. It is implemented and tested against
every case below; nothing here is aspirational.

**How it arrives.** Dropped on the Ramp tab, opened, taken off the clipboard, pasted, or polled
from `./LEG.json` when the book is served out of the folder the converter writes into. A batch is
`{"legs":[…]}` or a bare array, filed in order. `legId` deduplicates: the same record twice posts
once.

**What the converter leaves on disk, and what the book may be given.** The Builder's converter
writes into `FBVNK_LEGS`:

| File | What it is | Safe to give the book |
|---|---|---|
| `LEG.json` | the **latest conversion only**, overwritten each run | Yes. This is what the poller reads. |
| `<YYYY-MM-DD_HHMMSS>.json` | the **complete record** — one dated file per conversion, never overwritten | Yes, including all of them at once and more than once. `legId` dedupes, so a re-drop posts nothing twice. |
| `REJECTED_<stamp>.json` | a run the converter itself refused | **Never.** It is the converter's own reject pile, kept for diagnosis. Anything in it failed the converter's checks, so nothing in it should reach the book's accounts. |

`legId` is `"YYYY-MM-DD/HH:MM:SSZ/tach"` — date, engine-start Zulu, tacho at engine start, slash
separated. Worked example from the converter: `"2026-09-16/08:20:16Z/1228.32"`. The book treats it
as an opaque string and compares it whole; the format is the converter's to keep stable, and
keeping it stable is what makes re-dropping the dated files harmless.

---

## The fields

Only a departure and a destination are required, and a position counts as either. Everything else
missing is derived where it can be and printed on the pilot's review page where it cannot, each
line labelled `assumption:` where it is one. Nothing is filled in silently.

| Field | Type | Source in V274 | What the book does |
|---|---|---|---|
| `schema` | string | converter | `"FBVNK_LEG/2"`. A different prefix is refused. `FBVNK_LEG/1` is still accepted, with a note. |
| `legId` | string | converter: `date + engineStart + tachStart` | The dedupe key. Missing → derived from date, route and time, and the page says so. |
| `date` | `YYYY-MM-DD` | converter, from the run folder | V274 stores the Zulu **day of year only, with no year**. The year is the converter's. Missing → today, labelled. |
| `from`, `to` | ICAO | **not stored** | Optional when positions are present — see below. |
| `depLat`, `depLon`, `arrLat`, `arrLon` | degrees | dep/arr lat/lon | The book resolves the nearest aerodrome within **8 nm**. Unresolved → `????`, priced at the default field, labelled, and the pilot naming it once teaches the book that position for good. |
| `engineStart`, `shutdown` | ISO with `Z`, or `HH:MM` / `HHMM` **UTC** | dep/arr Zulu seconds of day | Block time. Shown on the review page; not what the carnet totals. |
| `takeOff`, `landing` | same | take-off seconds; **last** landing seconds | Airborne. **This is Heures de Vol** — Claus flies the book on flight time, not block. |
| `landings` | int | landings count | More than one → the page says the arrival time is the last one, which is all she keeps. |
| `airborneHours`, `blockHours` | number | airborne seconds; block = arr − dep | Optional. Preferred over the clock times when sent; otherwise derived, wrapping once past midnight. |
| `tachStart`, `tachStop` | hours | `NK_ST_TACH` at dep/arr | Fills the TACHO column — Claus's own, ruled into page 86 by hand. Missing → the column stays empty and the page says so. |
| `fp5lStart`, `fp5lShutdown` | litres | FP-5L **REM** at dep/arr, whole litres | The tank total. These are the Carburant Départ/Arrivée columns. **Not** the burn — see below. |
| `burnL` | litres | USED burn, to 0.1 L | What the book bills when present. |
| `burnSource` | enum or `null` | converter | `logger` \| `aircraft` \| `rem` \| `uplift`, or JSON `null` for "no figure — let the book choose". `null` and the field being absent are the same thing to the book. See the chain below. |
| `refuelInLeg` | bool | refuel-in-leg flag | Makes `rem` unusable. |
| `touchdownFpm` | number | last landing V/S, **negated** | **Positive is descent.** Clamped at 0; see below. |
| `noTakeOff` | bool | `noTakeOff` flag | An engine run that never got airborne. Filed, and it posts, but kept out of the carnet — a ground run is not a flight. The Logbook page says how many are being left out. |
| `stopInAir` | bool | `engineStoppedInFlight` | Written into the carnet's Observations as *Arrêt moteur en vol*. Flagged, never billed, per the standing rule. |
| `uplift`, `fuelPrice` | litres, €/L | not stored | Optional. Price missing → the standing price for the destination, labelled. |
| `power` | object | not stored | `maxRpm`, `minutesAboveCruise`, `hardBraking`. Missing → "by the book", labelled. **The thresholds in the book are this session's proposal, not measured off the aeroplane.** Set the real ones, or send `style` outright and skip the inference. |
| `surface`, `nights`, `handling` | string, int, bool | not stored | Optional. Surface missing → asphalt, labelled. |
| `conditionBefore`, `conditionAfter` | object | not stored | For a leg carrying a service: the eleven `NK_ST_C_*`/`NK_ST_OIL_QT` values plus the R in force, so a pro-rata posting is computable. Comes with the strip kit. |
| `note` | string | not stored | One line for the carnet's **Observations** column, where Claus's own hand-written remarks go. Truncated at 160 characters. Not the same field as `notes[]`. |
| `notes[]` | array of strings | converter | **Added in revision 4.** The converter's own notes — what the laptop saw while it read the logger. Shown on the pilot's review page under *Du convertisseur*, kept apart from the book's own notes. Never enters the carnet. Rules below. |

---

## Fuel: why there are two fuel numbers

`fp5lStart` and `fp5lShutdown` are REM, the tank total, because that is what Claus writes in his
own book. The **burn** is a separate figure because a refuel inside the leg makes
start-minus-shutdown wrong: the tank goes up mid-leg and the subtraction loses everything burnt
before it. On a 200 → 190 leg with an uplift in the middle, the subtraction says 10 L against a
real burn nearer 70.

The book bills in this order, and labels anything but the first:

| `burnSource` | Use it when | Vector |
|---|---|---|
| `logger` | the laptop logger ran the whole leg, so `burnL` is the sum of the per-row `fp5l_used` increases | V19 |
| `aircraft` | no logger run, V274 saved the leg's own burn, **and `refuelInLeg` is false** | V20, V24 |
| `rem` | no burn figure at all **and** `refuelInLeg` is false | V21 |
| `uplift` | last resort | V22 |
| — | a refuel, no burn figure and no uplift: **nothing is billed**, and the page says exactly that | V23 |

`burnSource` names which rung the converter believes applies; the book still walks the chain and
can end somewhere else, as V24 does. **`null` is a legitimate value** and means the converter is
not naming a rung: the book then starts at the top and takes the first rung the record can
support. Verified on the branch, through `JSON.parse`, against all three V23-family shapes — `null`
and the field being absent give the identical result in every one:

| Record | `burnSource: null` | field absent |
|---|---|---|
| FP-5L pair 200 → 158, no refuel | `rem`, 42.0 L | `rem`, 42.0 L |
| refuel, uplift 60 | `uplift`, 60.0 L | `uplift`, 60.0 L |
| refuel, no burn, no uplift (**V23**) | nothing billed | nothing billed |

Two things to know about `null` before the converter leans on it:

- **`null` with a `burnL` present is labelled `logger`.** With no named rung the book takes a
  present `burnL` as the logger's, because that is the only rung that can produce one without the
  aeroplane's own figures. If the converter ever derives a `burnL` some other way, it must name
  the rung — otherwise the review page will credit the logger for a number the logger never made.
- **The string `"none"` is not a value.** It is ignored exactly as any unrecognised string is, so
  it behaves like `null` today and would break silently if `"none"` ever became meaningful. Send
  JSON `null`.

`burnL` inherits the FP-5L's own gaps whatever the source: a single-update drop of 2 L or more is
never counted, and nothing counts while the instrument is unpowered. REM and USED can therefore
disagree, which is the reason `burnSource` exists at all.

**There is no stored `burnL` variable.** Each rung derives it:

- **`logger`** — the sum of the `fp5l_used` increases across the leg's rows, from the `lb_open`
  0 → 1 transition to the `lb_n` increment. Summing increases is why this rung survives a refuel.
- **`aircraft`** — `NK_LB_L_FUEL_ARR − NK_LB_L_FUEL_DEP`. Those two are FP-5L **USED** readings,
  and a refuel resets USED, so the subtraction comes out negative or tiny and V274 itself reports
  "no leg burn". **Hence the `refuelInLeg` guard, added in revision 4 (K7c C2)** — without it a leg
  that burnt ~70 L would have been billed as 9. The book was fixed and V24 pins it.
- **`rem`** — `fp5lStart − fp5lShutdown`, the REM tank totals, guarded the same way for the mirror
  reason: a refuel raises the tank mid-leg and the subtraction loses everything burnt before it.

## Touchdown rate

Positive is descent. The converter negates the sim's own negative vertical speed, so a normal
landing recorded as −180 fpm reaches the book as **+180 and stays 180**.

> **Corrected in revision 4 (K7c C1).** Revision 3 of this file said "0 or positive after negating
> → 0", which is backwards: a normal landing *is* positive after negating, so read literally it
> would have clamped every landing to 0 and the hard-landing check would never have fired. The rule
> below is the right one. **The book's code was never wrong** — it clamps on negative, keeps
> positive, and that was verified on the branch before this text was corrected: +180 → 180,
> +520 → 520, +560 raises "Hard landing at 560 fpm — inspection required", −180 → 0 with a note.
> Only this document was wrong, and it is the document the converter is built from, which is what
> made it worth catching.

- **Positive** → kept as it is. This is the normal case and it is what the hard-landing check reads.
- **0** → 0, labelled "a float, or a very soft arrival". No hard-landing check made on that leg.
- **Negative after negating** → clamped to 0 and said out loud, because it means one of two things:
  a converter that forgot to negate, in which case every landing would come through soft and the
  check would silently never fire again; or an arrival with no descent at all. Worth checking the
  converter either way.
- **Missing** → 150 fpm, labelled, and the page states plainly that no hard-landing check was made.

## `notes[]` — the converter's own notes

Added in revision 4, on the PM's proposal of 16 Sep 18:2x. Before it, a `notes[]` array on a
record was accepted without complaint and **silently dropped** — the book read only the field names
it knew. That was the wrong answer to give a converter that had something to say: a line like
*"fp5l gap 3 L at 09:41Z, not counted"* is the only thing that can explain a fuel figure the book
has to take on trust, and losing it left the pilot signing a number with no account of it.

**What the book does with them, as implemented and verified on the branch:**

| Sent | Result |
|---|---|
| array of strings | kept in order, shown on the review page |
| absent, `null`, or `[]` | nothing shown, nothing said |
| a string instead of an array | nothing kept, and the book says *"The record carries a `notes` field that is not an array. Ignored — the contract has notes[] as an array of strings."* |
| a member that is not a string, or empty after trimming | that member dropped |
| more than **12** | the first 12 kept |
| longer than **160 characters** | truncated to 160, the 160th an ellipsis |
| anything dropped for any of the three reasons above | counted out loud in the book's own notes: *"4 of the 5 notes on the record are not shown — not a string, empty, or past the twelfth."* |
| HTML or script in a note | escaped, never parsed. Verified: `<img src=x onerror=…>` renders as text and creates no element. |

The caps exist so a converter that starts shouting cannot bury the review page, and the count of
what was dropped exists because a note silently discarded is worse than no note at all — which is
the whole reason this field was added.

**Where they appear.** On the pilot's review page, inside the AUTO block, under their own heading
**Du convertisseur**, above the book's own notes and in darker ink. They are kept in a separate
array from the book's notes on purpose: the book's notes are what the book decided, `notes[]` is
what the laptop observed, and a pilot signing the page should be able to tell the two apart.

**Where they do not appear.** Not in the carnet, not on the chit, not in the ledger. The carnet's
Observations column is `note` — singular, a string, Claus's own remark — and that is a different
field. A converter with something for Observations sends `note`; a converter with something about
the conversion sends `notes[]`.

## Times

UTC. The aeroplane's clock is Zulu, so every time field is Zulu and a leg carries `tz: "Z"`.

The carnet has a UTC / local control and **Claus tapped UTC on 16 Sep at 18:00**, which is where
the book is set. The control stays, so local costs nothing if he changes his mind. A leg typed into
the book's fallback form carries no zone and is never converted — a conversion needs a base, and guessing one would put a wrong time in the
carnet. The column head says which base is showing. Checked in Europe/Lisbon: 0800/0918 Z shows as
0900/1018 local, and the hand-filed rows do not move.

## The installed variable names

V274's saved set, for the converter to read. Names as installed; this file does not rename anything.

| Contract field | V274 variable |
|---|---|
| `engineStart`, `shutdown` | `NK_LB_L_DEP_S`, `NK_LB_L_ARR_S` (Zulu seconds of day) |
| `date` | `NK_LB_L_DEP_DOY` (Zulu day of year, **no year**) |
| `takeOff` | `NK_LB_L_TO_S` (−1 = none seen) |
| `landing` | `NK_LB_L_LDG_S` (−1 = none; the **last** landing) |
| `landings` | `NK_LB_L_LDG_N` |
| `touchdownFpm` | `NK_LB_L_LDG_VS`, negated — sim sign is negative for descent |
| `airborneHours` | `NK_LB_L_AIR_S` |
| `depLat`, `depLon`, `arrLat`, `arrLon` | `NK_LB_L_LAT_DEP`, `LON_DEP`, `LAT_ARR`, `LON_ARR` |
| `fp5lStart`, `fp5lShutdown` | `NK_LB_L_REM_DEP`, `NK_LB_L_REM_ARR` (REM, whole litres) |
| `tachStart`, `tachStop` | `NK_LB_L_TACH_DEP`, `NK_LB_L_TACH_ARR` |
| `burnL` (`aircraft` rung) | **derived**: `NK_LB_L_FUEL_ARR − NK_LB_L_FUEL_DEP`, USED readings, no-refuel only |
| `burnL` (`logger` rung) | **derived**: the sum of `fp5l_used` increases across the leg's logger rows |
| `refuelInLeg` | `NK_LB_L_F_REFUEL` |
| `noTakeOff` | `NK_LB_L_F_NOTO` |
| `stopInAir` | `NK_LB_L_F_STOPAIR` |

Leg boundaries in a logger run come from the `lb_*` datums: `lb_open` going 0 → 1 opens a leg and
the `lb_n` increment closes it.

## What the converter has to derive

Per K17b §7, and the book assumes all of it is done before the record reaches it:

- the **year** — V274 stores the Zulu day of year only, with no year;
- the **next day** when arr, take-off or landing seconds fall below dep's (midnight wrap);
- **the next year** when that wrap happens on day 365 or 366: a departure on 31 December whose
  arrival, take-off or landing crosses midnight belongs to `year + 1`. Added in revision 4 (K7c);
- `-1` → absent;
- `touchdownFpm` = the negated V/S, clamped as above;
- block = arr − dep, with the wrap;
- `legId`, `date`, `burnSource`.

## The data-path limit the converter must respect

**Only the last closed leg is in V274's saved set.** Several legs in one session reach the laptop
individually only if the logger ran — each close shows as an `lb_n` increment in the rows. Without
the logger, the earlier legs of that session survive only inside the totals, and no record can be
written for them. The book has no way to detect a leg that was never converted, so this is the
converter's to get right, not the book's.

## Not supplied by the aeroplane

`from`/`to` as codes, `power`/`style`, `surface`, `nights`, `handling`, `uplift`, `fuelPrice`,
`conditionBefore`/`conditionAfter`. The book's labelled defaults stand for all of them.

## Closed since revision 3

1. **Aerodrome coordinates — done.** R17 landed the ARP for all eight from the national AIPs, each
   with its chapter and whether a second read confirmed it. They are in the book's field table and
   in `PRICES_AND_BILLING_RULES.json` → `fields`. The closest pair is LPCO–LPVL at 67.5 nm against
   the 8 nm resolve radius, so nothing can be confused. A position that still does not resolve
   files as `????` and says so, and naming it once on the review page teaches the book.
2. **`LEAP` / Teruel — done, and the book was the one that was wrong.** `LEAP` is Ampuriabrava;
   Teruel is `LETL`. The two codes had been swapped since the first build. Settled not by the AIP
   alone but by an invoice of Claus's: BP Energía España `2002864264` prints *"Location of
   Delivery: TEV - LETL - TERUEL"* and bills F-BVNK for 155.000 L on 24 OCT 25 at €366.14, which
   is €2.3622/L against the 2.36 that row has always carried. `LEMP` is Los Martínez del Puerto in
   Murcia and nothing of Claus's uses it, so it is dropped. His planning — `LPCS - LETL` then
   `LETL - LEAP` — reads Cascais → Teruel → Ampuriabrava and was always consistent.
3. **The carnet's time base — done.** Claus tapped **UTC**, 16 Sep 18:00. The book is set to it
   and the UTC / local control stays, so the other answer remains cheap. Checked in Europe/Lisbon:
   0800/0918 Z shows as 0900/1018 local and the hand-filed rows do not move.

## Open against this contract

1. **Whole litres.** The REM figures are whole litres, the PM's decision; the burn is to 0.1 L.
   The book prints what it is given. Claus can ask for decimals.
