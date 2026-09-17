# LEG_CONTRACT — `FBVNK_LEG/2`

> **Revision 9: the ICAO rows, against V277 as built.** The aircraft decodes the nearest airport
> within **3 NM** at engine start and at the flown close and stores the code; an airport it could
> not decode arrives as **`----`**, not `????`; `depLat` / `depLon` / `arrLat` / `arrLon` are no
> longer written at all. The book already behaves this way — `icao(rec.from) || '----'`, verified
> again on the published page — so this revision is contract text and vectors, not code.
>
> **Revision 8: the reserve cursor is bounded.** K7e failed revision 6's progression rule, and the
> numbers were not small — one reading from another machine accrued €7,267.19, a mistyped
> `tachStart` €26,957.43, and out-of-order filing double-accrued the case this very file called
> safe. Records' tested remedy is taken as it stands. See *The tacho is now the money*.
>
> **Revision 7: the aeroplane sends the ICAO, and the position machinery is gone.** Claus,
> 16 Sep 22:08: *"stop all this position shit, with lots and lats, u know what airport we are at
> for take off and landing ICAO, use that. over engineered BS."* He is right. V277 asks the sim
> which airport she is at, at engine start and at shutdown, and sends the code. `depLat`,
> `depLon`, `arrLat`, `arrLon` are **dropped from the contract**, and everything the book had for
> resolving them — the radius, the coordinate set, the old `????` code, the aerodrome-positions panel,
> the position lines — is **removed**. A record with no code shows `----` and he types it.
>
> Revision 7 also builds the review page he ordered at 21:54 and splits the clock he corrected at
> 21:57. See *What the review page shows*.
>
> **Revision 6: the hour meter is the clock, and `legNo` is added.** Claus answered the accrual
> question on the Annunciator at 21:05:42 on 16 Sep — *"Tacho time (the hour meter)"* — so
> `tachStart` and `tachStop` stopped being decoration on a carnet column and became the figures the
> money runs on. What that means for the converter is under *The tacho is now the money*. The other
> addition is `legNo`, the aeroplane's own leg counter, so the review page can print the iPad's
> first line as the iPad prints it.
>
> **Revision 5 changed what reaches the carnet, and added two optional fields.** Claus's rule of
> 16 Sep 18:34 makes the **airborne event** the test for a logbook entry — not the take-off flag,
> which is a different thing and gets an air start exactly backwards. The added fields are
> `realEngineStart` / `realShutdown`. Revision 4's own additions (`notes[]`, `burnSource: null`,
> and what the converter leaves on disk) stand unchanged. **No existing field name has changed**
> in either revision: every name is that of `90020d89`, the copy the Builder is working to.
>
> Revision 5 also fixes a refusal in the book that would have thrown away the first real record
> off the aeroplane. See *What reaches the carnet*.

The record the laptop book reads, one per leg. Revision 2 of the contract, rewritten after
Records read revision 1 against what V274 actually stores (K7b §6) and after the PM's V274
install note of 16 Sep 16:4x.

**Who writes it.** `nk_legs.py` on the laptop, from a logger run folder to a `LEG.json` in these
field names. Builder's kit, reviews K19 / K19b / K19c, **installed 16 Sep 18:3x**. The aeroplane
itself writes LocalVars and logger datums, not this JSON.

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
| `<YYYY-MM-DD_HHMMSS>.json` | the **complete record** — one dated file per conversion, never overwritten | Yes, including all of them at once and more than once, and **in any order**. `legId` dedupes a repeat, and from revision 8 the cursor rule handles a leg that arrives after newer ones. Revision 6 called this safe when it was not: K7e P2 showed a late-dropped file double-accruing. |
| `REJECTED_<stamp>.json` | the **rejected legs of a run whose valid legs were written** (converter exit 4). A run with nothing valid (exit 3) writes no file at all. | **Never.** Every leg in it failed the converter's own checks, so none of it should reach the book's accounts — and the run's good legs are already in the dated file beside it. (Corrected in revision 8, K7e L2.) |

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
| `from`, `to` | ICAO, or `----` | **V277** (B33): the aircraft decodes the **nearest airport within 3 NM**, at engine start and at the flown close, and stores the code | Taken as sent — the book does no resolving of its own. An airport the aircraft could not decode arrives as **`----`**, is priced at the default field, and the pilot types the code on the review page. |
| `engineStart`, `shutdown` | ISO with `Z`, or `HH:MM` / `HHMM` **UTC** | dep/arr Zulu seconds of day | Block time. Shown on the review page; not what the carnet totals. |
| `takeOff`, `landing` | same | take-off seconds; **last** landing seconds | Airborne. **This is Heures de Vol** — Claus flies the book on flight time, not block. |
| `landings` | int | landings count | More than one → the page says the arrival time is the last one, which is all she keeps. |
| `airborneHours`, `blockHours` | number | airborne seconds; block = arr − dep | Optional. Preferred over the clock times when sent; otherwise derived, wrapping once past midnight. |
| `tachStart`, `tachStop` | hours | `NK_ST_TACH` at dep/arr | Fills the TACHO column — Claus's own, ruled into page 86 by hand — and **from revision 6 it is also what the money runs on**. Missing → the column stays empty, the leg accrues its airborne time instead, and both are said on the page. See *The tacho is now the money*. |
| `legNo` | int | `NK_LB_N` | **Added in revision 6.** The aeroplane's own leg counter, the number the iPad prints as `Leg 12:`. It is Claus's cross-reference between the cockpit panel and the laptop page. Optional: without it the panel's first line reads `Leg —:` and shows the `legId` instead. |
| `fp5lStart`, `fp5lShutdown` | litres | FP-5L **REM** at dep/arr, whole litres | The tank total. These are the Carburant Départ/Arrivée columns. **Not** the burn — see below. |
| `burnL` | litres | USED burn, to 0.1 L | What the book bills when present. |
| `burnSource` | enum or `null` | converter | `logger` \| `aircraft` \| `rem` \| `uplift`, or JSON `null` for "no figure — let the book choose". `null` and the field being absent are the same thing to the book. See the chain below. |
| `refuelInLeg` | bool | refuel-in-leg flag | Makes `rem` unusable. |
| `touchdownFpm` | number | last landing V/S, **negated** | **Positive is descent.** Clamped at 0; see below. |
| `noTakeOff` | bool | `noTakeOff` flag | No take-off was stamped. **It does not mean "not a flight"** — an air start is spawned airborne and carries this flag, and it IS a carnet entry. What keeps a record off the carnet is zero airborne time, not this flag. See *What reaches the carnet*. (Corrected in revision 8, K7e L1.) |
| `stopInAir` | bool | `engineStoppedInFlight` | Written into the carnet's Observations as *Arrêt moteur en vol*. Flagged, never billed, per the standing rule. |
| `uplift`, `fuelPrice` | litres, €/L | not stored | Optional. Price missing → the standing price for the destination, labelled. |
| `power` | object | not stored | `maxRpm`, `minutesAboveCruise`, `hardBraking`. Missing → "by the book", labelled. **The thresholds in the book are this session's proposal, not measured off the aeroplane.** Set the real ones, or send `style` outright and skip the inference. |
| `surface`, `nights`, `handling` | string, int, bool | not stored | Optional. Surface missing → asphalt, labelled. |
| `conditionBefore`, `conditionAfter` | object | not stored | For a leg carrying a service: the eleven `NK_ST_C_*`/`NK_ST_OIL_QT` values plus the R in force, so a pro-rata posting is computable. Comes with the strip kit. |
| `realEngineStart`, `realShutdown` | ISO with `Z` | converter, PC clock | **Added in revision 5.** The laptop's own UTC at the logger rows that opened and closed the leg. Logger legs only — a `state.CFG` leg has neither. `engineStart` / `shutdown` stay the **sim** clock and stay what the carnet prints. Carried and shown on the review page only where the two differ. |
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

## What reaches the carnet

Claus, 16 Sep 18:34, word for word:

> "I want to remark. If NO airborne event, no log entry at all. Eg, engine starts, taxi etc. if no
> airborne (flight even) NO log book entry. Only airborne (flight time) events goes into the
> logbook. So, from time to time there may be a tacho log entry "discrepency" between last leg and
> current (last) leg - just like real world, so just accept this."

**The test is the airborne event, and it is not the `noTakeOff` flag.** Those two come apart in one
case that matters: an **air start** is spawned airborne, so it has no take-off at all —
`noTakeOff` is true and the airborne time is real. Reading `noTakeOff` as "not a flight" would have
kept every air start out of Claus's logbook. Revision 4 of the book did exactly that. Fixed:

| Record | In the carnet? | Posts? |
|---|---|---|
| airborne time > 0 | **yes** | yes |
| `noTakeOff: true` **and** airborne time > 0 — an **air start** | **yes**, and the page says it is one | yes |
| airborne time 0 or absent, no take-off and no landing — a ground run, taxi, false start | **no** | **yes** — it burnt fuel and ran the engine |
| no airborne figure, but a take-off or a landing time | yes | yes |
| airborne 0 **and** a take-off or landing time — the record contradicting itself | no, and the page says the converter is worth checking | yes |

When the record carries an airborne figure, that figure decides. Only when it carries none does the
book fall back to the evidence of a take-off or landing time, because an aeroplane that took off was
airborne whether or not the record put a number on it.

**A refusal that would have lost the first real record.** Before revision 5 the book took the
airborne figure as the leg's hours, and a record stating `airborneHours: 0.0` came out as zero hours
and was **refused outright** — not filed, not posted, gone. The block-time fallback only fired when
the airborne field was *missing*, not when it was *zero*. The Builder's first real read off the
aeroplane, `2026-09-16/16:14:01Z/1250.07`, is exactly that shape: `airborneHours 0.0` with
`blockHours 0.081`. It would have been thrown away with its fuel and its engine time. The fallback
now fires on zero as well as on absent, so a ground run files on its block time, posts, and stays
off the carnet.

**The tacho gap between legs is not an error.** Nothing in the book checks tacho continuity between
consecutive legs and nothing ever did — there is no check to remove. What the Logbook page says
about a run it left out now states the gap is expected. Claus: *"just accept this."*

The same is true of the **fuel** gap, and for the same reason: a run that leaves no leg takes litres
out of the tank between one leg's Carburant Arrivée and the next leg's Carburant Départ. Page 86's
own rows carry forward in seven cases of eight, so a break is visible on the page. What the book
does about it is a money question and is open — `NOTE.md` § *Fuel and hours that fall between legs*.

## The tacho is now the money

Claus, on the Annunciator at 21:05:42 local, 16 Sep 2026, asked which clock the reserves and the
inspection counters run on. Word for word:

> "Tacho time (the hour meter)"

So `tachStart` and `tachStop` are no longer a column to fill. They carry the engine reserve at
€20.79/h, the propeller reserve at €1.36/h, the 25 / 50 / 100-hour inspections, the airframe total
and the 2000-hour TBO counters. **Airborne time keeps the carnet's Heures de Vol and the airborne
totals, and nothing else.**

**It is the meter's progression that accrues, not the leg's own span.** Under V275 only airborne
legs become records at all, so the taxi, the run-up and any ground running between one record and
the next sits in the gap between the previous `tachStop` and this one. A real hour meter counts
every second of it, and so does the book:

| Case | What accrues | Cursor |
|---|---|---|
| Normal | this `tachStop` less the **previous record's `tachStop`**, to 0.01 h | moves |
| The first record the book sees | its own `tachStop − tachStart` | picked up |
| `tachStart` below the cursor | a reload or another machine. Flagged, **nothing negative posted**, the leg accrues its own span | resumes from the new reading |
| No `tachStop` at all | the leg's **airborne** time, said out loud, and **carried** — the next record with a real reading subtracts it from the measured gap | stays |
| **Gap more than 5 h beyond the leg's own span** | the own span, flagged. More than five hours cannot be ground running between two legs, so it is two meters, not one. *(K7e P1)* | resumes |
| **An own span more than 5 h beyond the leg's flying** | the **airborne** time, flagged. That is what a mistyped reading looks like, not a leg. | as above |
| **A record older than the last one filed** | 0 if its reading closes **at or below** the cursor, because those hours are already inside a measured gap; otherwise its own span. *(K7e P2 — the late-dropped dated file)* | **never moves** |
| **A gap that comes out negative** | nothing. A reading that goes down is not flying. *(K7e P3)* | **never moves** |

Verified against every sequence Records built: their another-machine case **3.17 h** where revision 6
gave 332.99; out-of-order filing **4.00 h** where revision 6 gave 5.80; a legitimate three-hour
ground run between two legs still **5.00 h**, so the bound does not touch the real case. V33 and V34
are unchanged at 6.03 h.

**What this asks of the converter.** Nothing new in the field names — but a missing or wrong tacho
now costs money rather than a blank column, and a `tachStop` that does not carry forward to the next
record's cursor is the one error the book cannot silently absorb. `NK_ST_TACH` never goes backwards
in normal running: no reset touches it, and it is seeded to 1246.58 only on a first install.

Worked in `TEST_VECTORS.json` → V33 and V34, and verified in the running book.

## What the review page shows

Claus ordered the page at 21:54 (*"update the cost sheet artisfact to look like the latest
proposal"*) and its layout at 22:08. Both are built.

**The AUTO block — what she recorded.** Departure and arrival by ICAO with their engine times,
then the flying, then the fuel, then the tacho. **No positions, no Zulu day, no prose.** Touchdown
**positive**, which is what the record carries and what the book bills on:

| | |
|---|---|
| Départ | `LPCS   08:20Z` |
| Arrivée | `LPCO   09:41Z` |
| Décollage | `08:32Z` |
| Atterrissage | `09:35Z` |
| En vol | `1:03` |
| Bloc | `1:21` |
| Carburant | `220 → 194 L` |
| Consommé | `26.4 L`, or `refuel in leg` |
| Atterrissages | `1` |
| Touché | `180 fpm` |
| Tacho | `1228.32 → 1229.34` |
| Réf. | the `legId` |

A figure the record does not carry shows a dash. Nothing on this block is derived.

**Eleven rows before the signature**, each defaulting to what the book would have picked by
itself, so a page signed untouched posts exactly what the same record posted before the rows
existed: Nature du Vol, Fonctions, Équipage — Noms, Atterrissages jour/nuit, Nuit/IFR heures,
Incidents · Observations, Comment elle a volé, Surface, Nuits stationnées, Handling, € par litre.

**Five of them cost money, not four** as revision 5's prose said. Nuits stationnées, Handling and
€ par litre bill this leg; Comment elle a volé and Surface bill later, through wear. All five are
marked on the page.

**So a leg now enters the book on arrival and posts at the signature.** Nothing else could honour
those rows: invoices written before he touches the page cannot reflect his choices, and re-posting
afterwards would either double-count or burn the seeded squawk sequence. The carnet row and the
inbox entry still appear the moment the leg arrives.

**The two EASA rows are captured and kept off the carnet.** Landings by day and night, and night
and IFR time, are AMC1 FCL.050 columns 8 and 9 — a pilot's logbook, not an aircraft journey
record. Claus's document is a *Carnet de Route*, so they live on the review page only.

**Durée (en vol)**, and every total airborne. EASA flight time is a different thing — first
movement for take-off to coming to rest — so the label says which one this is.

**The head of the Logbook tab and of every review card** carries: *Home simulator record — not an
EASA logbook. MSFS 2024, non-qualified device. Not flight time, not FSTD time.*

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

## What the aeroplane saves, V275

Installed 20:56, 16 Sep 2026 (the PM's report; the package is his).

- A leg runs from engine start to engine shutdown on the ground, and is **saved only if the
  aircraft was airborne at least 10 s in one continuous stretch** — the same 10 s that stamps a
  take-off and a landing.
- Ground runs, taxi bumps and hops under 10 s leave **no entry at all**: no leg number, no totals,
  no record. The zero-airborne record of revision 5 is therefore no longer produced. The book still
  handles it, because it costs nothing and covers anything written before tonight.
- **An air start still arrives with `noTakeOff: true`** and still clears the 10 s test, so it is
  still a record and still a carnet entry. That is the case revision 5 fixed.
- A bounce is not a landing; a touch-and-go counts two.
- An engine stop in the air closes the leg 60 s after it is on the ground with the engine off.
- Tacho gaps between consecutive legs are expected. Claus: *"just accept this."*

**Known limit, and the book cannot detect it.** A sim reload within about 20 s of touchdown, or
while taxiing in, leaves **no entry for that flight**. A real flight — airborne, landed, fuel burnt
— produces no record, and nothing in the record set says one is missing. The only visible trace is
the tacho gap, and under revision 6's clock those hours are still accrued the moment the next record
arrives, because the meter's progression counts them whether or not a leg was written for them. The
flight itself cannot be put back in the carnet and nothing can.

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

1. **Aerodrome coordinates — moot from revision 7.** R17 landed the ARP for all eight and R19
   added LFBT, and none of it is needed any more: the aeroplane sends the ICAO. The coordinates
   were removed from the book with the rest of the position machinery. R17 and R19 are not wasted
   work — they are what proved the resolver worked before Claus decided the resolver was the wrong
   answer to the question.
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
