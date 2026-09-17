# NOTE — FBVNK_SIM_LEDGER_KIT1 · REVISION 9

**Contract text and vectors only. No code changed.** The Builder's B33 = V277 delta, relayed by the
PM at 22:5x: the aircraft decodes the **nearest airport within 3 NM** at engine start and at the
flown close and stores the code; an airport it could not decode arrives as **`----`**, not `????`;
and `depLat` / `depLon` / `arrLat` / `arrLon` are no longer written at all.

**The book already behaves this way,** which the PM expected and this session verified on the
published page rather than assuming:

| Record | Leg |
|---|---|
| `from: "LPCS", to: "LPCO"` | `LPCS` → `LPCO`, taken as sent |
| `from: "lfbt"` | `LFBT` — upper-cased, four letters checked |
| neither field | `----` → `----`, both boxes offered, typing `lfbt` sets it |
| `from: "LP"`, `to: "LPCO1"` | `----` — not four letters, so not a code |
| an old record still carrying the four positions | `----`, and **the positions are ignored entirely** — the leg does not even carry them |

No `????` is produced anywhere. That was the old resolver's marker and the resolver is gone.

**What changed in the kit:** the `from`/`to` row now states the aircraft's 3 NM decode rather than
"the airport the sim reports"; `----` is named in place of `????`; **V30's `expected_now` is
corrected**, because it still described `????` and the four position fields, both of which are gone;
**V39** is added for the ICAO path; and the Tarbes open item is rewritten — its ARP is not needed at
all now, only its fees.

**K7e L1, the stale `noTakeOff` row, was fixed in revision 8** and is not re-done here.

## What revision 8 changed

**K7e failed revision 6 on the reserve progression rule, and it was right to.** The numbers Records
put on it are the point: one reading from another machine and the next real record accrued the whole
difference between the two meters — **328.09 h, €7,267.19**. A mistyped `tachStart` gave
**€26,957.43**. Out-of-order filing double-accrued the very case `LEG_CONTRACT.md` called safe.

Revision 8 takes **Records' tested remedy as it stands** rather than designing something new, which
is what the leader asked for and the right call anyway — they had already run it.

| | Rule | Cursor |
|---|---|---|
| **P1a** | a forward gap more than **5 h** beyond the leg's own span is two meters, not one: accrue the own span, flag it | resumes |
| **P1a, extended** | an **own span** more than 5 h beyond the leg's own flying is not a leg either: accrue the airborne time, flag it. Same constant, same idea — this session's extension, because Records tested the gap path and a mistyped `tachStart` comes in on the other one | as above |
| **P2** | a record **older than the last one filed** accrues 0 when its reading closes at or below the cursor, else its own span | **never moves** |
| **P3** | a **negative gap** accrues nothing | **never moves** |

**Five hours** is this session's figure, set where Records left it to be set: longer than any ground
running between two of Claus's legs, far shorter than the smallest gap a second machine produces.

The chronology key is `date + engineStart`, captured in `normaliseLeg` **from the record's own
date** — before the ingest path pushes a back-dated leg forward to keep the carnet in order. Without
that it would compare rewritten dates and never fire. That is the sort of thing that only shows up
when you run it.

**Every sequence Records built, run through the real flow in a browser:**

| Case | Revision 6 | Revision 8 | True |
|---|---|---|---|
| V33 + V34, the good sequence | 6.03 | **6.03** | 6.03 |
| P1, another machine | 332.99 | **3.17** | Records expected 3.17 |
| P1, `tachStart` mistyped 12.30 for 1230.30 | 1220.72 | **2.92** | under-accrues, the safe side |
| P1, `tachStop` mistyped 123.10 | ~1110 | **3.68** | |
| P2, filed A, C, B, D | 5.80 | **4.00** | 4.00 |
| P3, `tachStop` below the cursor | over by 0.90 | **1.78**, sound | |
| A legitimate 3 h ground run between two records | 5.00 | **5.00** | 5.00 |

That last row is the one that matters as much as the failures: **the bound does not touch the real
case**, which is the whole point of putting it at five hours.

**Also fixed.** **A5** — V275 saves any airborne stretch of 10 s or more, so a hop can be 0.0028 h
and rounding to the minute left `0:00` against a leg that genuinely flew. The floor now goes on
after the rounding and reads the figure from before it, so a leg that flew shows at least `0:01`.
**L1** — the contract's `noTakeOff` row still said "an engine run that never got airborne … kept out
of the carnet", contradicting the same file two sections later; an air start carries that flag and
is a carnet entry. **L2** — `REJECTED_<stamp>.json` holds the rejected legs of a run whose valid
legs *were* written (exit 4); a run with nothing valid (exit 3) writes no file at all.

**Not changed, and said out loud.** **P4**, a carry larger than the next measured gap: the excess is
dropped rather than carried forward. Records call it bounded and small, the page says what it did,
and adding machinery for it would be exactly what Claus told this session to stop doing. **F1** is
closed by deletion — the positions are gone, so there are no coordinate labels left to argue about.
The key `reserves_per_flown_hour` keeps its name although the reserves run on the meter; the strip
reads that key, and renaming it to fix a label would break a reader.

**Why this is R8 and not R7.** The leader's list named R7, but R7 had already been reported and its
checksums published twenty minutes earlier. Amending reported bytes breaks the checksum contract, so
R7 stands as reported and this is R8.

## What revision 7 changed

Revision 7 is three orders from Claus in fifteen minutes, and one of them tells this session it
over-built. All three are done.

**21:54 — build it.** *"update the cost sheet artisfact to look like the latest proposal. Why I
need to ask for this"*. Everything the Job Card had as "proposed, not built" is built: the eleven
review rows before the signature, *Durée (en vol)* with every total airborne, the home-simulator
label at the head of the Logbook tab and every review card, and the neutral engine-time-between-legs
line under the carnet.

To make the money rows mean anything, **a leg now enters the book on arrival and posts at the
signature.** Invoices written before he touches the page cannot reflect his choices, and re-posting
afterwards would either double-count or burn the seeded squawk sequence. The carnet row and the
inbox entry still appear the moment the leg arrives.

**A correction of this session's own, found while building:** revision 5's prose said four rows move
money. It is **five**. Nuits stationnées, Handling and € par litre bill this leg; Comment elle a
volé and Surface bill later, through wear. Corrected on the page, in the rules file and in V36.

**21:57:47 — the inspection clock.** *"NO, the inspection times, sorry if I said otherwise, are
based on the journey log book airborne hours, NOT tacho."* So there are two clocks, and both are
his. The 25/50/100-hour inspections, the airframe total and the wear rates run on **airborne** hours
off the journey log. The reserves and the 2000-hour TBO counters run on the **hour meter** — his
21:05 pick — so the fund and the overhaul it pays for count the same hours. Verified: one leg feeds
the two counters different numbers, +5.317 h airborne and +6.03 h meter over the V33 sequence. V35.

**22:08 — stop over-engineering.** *"stop all this position shit, with lots and lats, u know what
airport we are at for take off and landing ICAO, use that. over engineered BS."* and *"FFS maintain
sanity check on what u come up with, don't over engineer, keep it simple and pick obvious."*

He is right, and the removal is larger than the build was. Out of the book: the aerodrome-positions
panel and the coordinate set, the 8 nm resolve radius, the `????` code, `resolveField`,
`nmBetween`, `knownCodes`, `fieldPos`, `posTxt`, `loadCoords`, `S.fieldPos`, the four position
fields on every record, the position line in the AUTO block, and the latitude and longitude in the
field table. **About 9 KB of machinery, replaced by taking the code the aeroplane sends.** A record
with no code shows `----` and he types it in the box that was already there.

The AUTO block is his layout of 22:08: départ and arrivée by ICAO with their engine times, then the
flying, the fuel and the tacho. No positions, no Zulu day, no prose, and the touchdown **positive**,
which is what the record carries and what the book bills on — so the sim-sign line revision 6 added
is gone too.

R17 and R19, the two coordinate studies, are not wasted work. They are what proved the resolver
worked before Claus decided the resolver was the wrong answer to the question.

## What revision 6 changed

From: travel-map-da cloud session (subagent to FBVNK PROJECT MANAGER)
Job: Step 7 GO, per PM's KIT1 review message, read 22:15, 15 Sep 2026
Revision 6: **Claus picked the clock**, and the laptop book moved to it. Plus the V275 logbook
layout on the review page, and `legNo`. Revision 5's carnet rule and its two defects stand and are
kept below; Records reviews R6 as K7e covering both. Revisions 1-5 stay frozen.

**KIT1 R6 REPORTED.** A new folder again, so the five before it stay frozen exactly as they were
reported and their md5s still point at the bytes they named.

## What revision 6 changes

Claus, on the F-BVNK Annunciator at **21:05:42 local, 16 Sep 2026**, answering
`ledger_accrual_clock`. Word for word:

> "Tacho time (the hour meter)"

R5 put that question up with the arithmetic: the two halves of this project were billing the same
aeroplane on different clocks, 18.80 % apart on page 86's own figures — €3.47 a leg, €303.98 a year
on his real 2025 flying. He picked the meter. **There is now one clock.**

### Which half moved, and which did not

| | |
|---|---|
| **The cockpit strip** | **Unchanged.** It was already reading `NK_ST_TACH` (LocalVar 11) under the `(MX_LOG, TACH)` cursor it has, and that was right all along. |
| **The laptop book** | **Moved.** It had been accruing on `leg.hours` — airborne. |
| **The test vectors** | **Unchanged, and worth saying plainly.** Every `hourly_reserve` figure in V1, V5, V6 and V15 was already computed from a TACH cursor delta, and every one is arithmetically correct — re-checked in R6, zero mismatches. The vectors were never on the wrong clock. The book was. |

**On the meter now:** the engine reserve at €20.79/h, the propeller reserve at €1.36/h,
`counters.hrs` and the 25 / 50 / 100-hour inspections it drives, `counters.ttaf`, and `engSMOH` /
`propSMOH` with their 2000-hour TBO postings.

**Still on airborne:** the carnet's Heures de Vol and the airborne totals — and the wear
coefficients, which is **this session's own call and is flagged rather than left silent**. Claus's
pick named the two reserves and the inspection counters. Those coefficients were rated against
airborne hours, so moving them would quietly re-rate every one by the 18.8 % the clocks differ by,
which nobody asked for; and the real wear model is the iPad's (`HANGAR.js`, `NK_*`), which is the
PM's. Say the word and they move.

### The progression rule, which is the part that needed care

The PM set it with the pick, and it is right: accrue the meter's **progression between records**,
not a leg's own `tachStop − tachStart`. Under V275 only airborne legs become records, so the taxi,
the run-up and any ground running between one record and the next lives in the gap — and that is
precisely the engine time Claus chose the meter to capture.

| Case | What accrues |
|---|---|
| Normal | this `tachStop` less the **previous record's `tachStop`**, to 0.01 h |
| First record the book sees | its own `tachStop − tachStart`; the cursor is picked up there |
| `tachStart` below the cursor | a reload or another machine. Flagged, **nothing negative posted**, the leg accrues its own span, the cursor resumes from the new reading |
| No `tachStop` at all | the leg's **airborne** time, said out loud, and **carried** — the next record with a real reading subtracts it from the measured gap, so nothing is billed twice |

That last row is the one that needed thinking about. Without the carry, a leg with no reading would
accrue its airborne hours and then the next real reading would measure a gap spanning that same leg
and accrue it again. Carrying the figure and netting it off the next measured gap is exact and
self-correcting, and it does not invent a meter reading the book never saw.

**Verified in the running book**, as a five-record sequence:

| Record | Meter | Accrued | Why |
|---|---|---|---|
| T-1 | 1228.32 → 1229.34 | **1.02 h** | first reading, own span |
| T-2 | 1230.00 → 1231.10 | **1.76 h** | gap from 1229.34. The leg ran 1.10; the other **0.66 h ran on the ground between the two records and is accrued** |
| T-3 | 900.00 → 901.25 | **1.25 h** | the meter went backwards. Flagged, nothing negative, own span, cursor resumes |
| T-4 | none | **1.50 h** | no reading: airborne, said out loud, carried |
| T-5 | 902.10 → 903.25 | **0.50 h** | gap 2.00 less the 1.50 carried |

Counters 18.40 → 24.43 h and the engine reserve €382.54 → €507.90. Recomputed independently in
Python `Decimal` with `ROUND_HALF_UP`: 6.03 h × €20.79 = **€125.36**, and 382.54 + 125.36 = **507.90**.
The book's own float arithmetic agrees to the cent.

**The seeded eight do not move.** All eight were hand-filed before the aeroplane wrote tacho, so
every one takes the airborne fallback: 18.4 h, engine €382.54, propeller €25.02, exactly as before
revision 6. That is the fallback doing its job on the only real data it has.

### The logbook layout, and `legNo`

Claus, 21:0x: *"update the cost ledger with what the lates layout of the lookbook looks like"*. The
review-and-sign page now reproduces the **V275 iPad logbook panel** line for line, built from the
installed `HANGAR.js` `lbLastText()` strings (md5 `91fa9dce`) rather than from the summary, so the
two cannot drift. Full detail and the three deliberate differences — always Zulu, the tacho with a
dot, the touchdown in the sim's own negative-down sign — are in `LEG_CONTRACT.md` → *What the review
page shows*.

`legNo` is added as an optional field so the panel's first line can read `Leg 12:` as the iPad
prints it. Without it the line reads `Leg —:` and shows the `legId`. **For the Builder:** it comes
straight off `NK_LB_N`, and it is the last thing between the two screens and "line for line".

His page-86 carnet columns are untouched, as instructed.

### Also in revision 6

V275's save rule and its known limit are in the contract now: ten seconds airborne in one stretch,
ground runs leave no entry, an air start still arrives with `noTakeOff: true` and still counts — and
a sim reload within about 20 s of touchdown **loses a whole flight, which the book cannot detect**.
Under the new clock those hours are at least still accrued, because the meter's progression counts
them whether or not a leg was written. The flight itself cannot be put back in the carnet.

## What revision 5 changed

Claus, 16 Sep 18:34, relayed by the PM, word for word:

> "I want to remark. If NO airborne event, no log entry at all. Eg, engine starts, taxi etc. if no
> airborne (flight even) NO log book entry. Only airborne (flight time) events goes into the
> logbook. So, from time to time there may be a tacho log entry "discrepency" between last leg and
> current (last) leg - just like real world, so just accept this."

Four things came out of working that through the book, and **two of them were defects in it**.

| | What | Where |
|---|---|---|
| **A1** | The carnet test was the `noTakeOff` flag, which is not the same thing as the airborne event. An **air start** — spawned airborne, no take-off — was being kept OUT of the carnet. It belongs in it. | the book's code |
| **A2** | A record stating `airborneHours: 0.0` was **refused outright** — not filed, not posted, thrown away with its fuel and its engine time. The Builder's first real read is exactly that shape. | the book's code |
| **A3** | No tacho continuity check exists and none ever did. Nothing to remove; the page now says the gap is expected. | confirmed, no change |
| **A4** | Two money questions the PM put: what happens to fuel and hours between legs, and whether the reserve basis is intended. **The kit and the book disagree on the basis**, by a figure that is measured, not guessed. | open — below |

### A1 — the air start, which the flag reading got backwards

`noTakeOff` and "not a flight" are not the same claim. An air start is spawned airborne: the flag is
true, the airborne time is real, and it is unambiguously a flight. Revision 4's book read the flag
and kept it off the carnet.

Fixed, and the fix is to test the thing Claus's rule is actually about. When the record carries an
airborne figure, that figure decides. When it carries none, the book falls back to whether a
take-off or a landing time exists, because an aeroplane that took off was airborne whether or not
the record put a number on it. A record that states zero airborne time **and** carries a take-off
time contradicts itself: the airborne figure is taken as the answer and the page says the converter
is worth checking.

Checked in the running book: a ground run stays off the carnet and the count under the page says so;
an air start of 0.85 h goes on it and the review page says *"No take-off on this record but 0:51
airborne — an air start. It goes in the carnet: the rule is the airborne event, not the take-off."*
Ordinary legs, legs with a take-off but no landing time, and the seeded eight are all unmoved.

### A2 — the refusal that would have lost the first real record

The book took the airborne figure as the leg's hours and only fell back to block time when the
airborne field was **missing**. A figure of **zero** is not missing, so `hours` came out 0 and the
record hit `return {ok:false, why:'The record carries no flight time…'}`. Refused. Nothing filed,
nothing posted, no trace.

The Builder's first real read off the aeroplane, 16 Sep 16:14:01Z:

    legId "2026-09-16/16:14:01Z/1250.07"
    airborneHours 0.0   blockHours 0.081   noTakeOff true   landings 0
    tach 1250.07 -> 1250.10   fp5l 231 -> 234   refuelInLeg true   burnSource null

That is the shape. It would have been thrown away. The fallback now fires on zero as well as on
absent, so the record files on its block time — **0:05**, the record's own 0.081 h rounded to the
minute as the book rounds everything — posts its costs, and stays off the carnet. Run through the
book as written above, it now files and prints: *"Zero airborne time, so the engine time for this
record is its block time, 0:05. It posts on that; it does not go in the carnet."*

Its aerodrome is **Tarbes LFBT**, which the book has never heard of. See *What Tarbes exposed*.

### A3 — the tacho gap is not flagged, and never was

Searched the book for any tacho continuity or discrepancy check between consecutive legs: **there is
none**. The TACHO column prints each leg's own pair and nothing compares one row's stop with the
next row's start. So there was nothing to make neutral. What did change is the line under the
Logbook page, which now reads: *"1 run with no airborne time is not on this page — only airborne
events go in the logbook. It is in the Ledger, and its fuel and fees posted as usual. The tacho gap
it leaves between legs is expected, not an error."*

### A4 — fuel and hours that fall between legs

Two separate questions. The first has a clean answer; the second is a decision for Claus and this
session will not take it.

**What the book does today, stated plainly.** Fuel is billed per leg off `burnL`, the tank pair, or
the uplift. Hours, reserves, wear and the 25/50/100-hour counters all accrue on `leg.hours`, which
is airborne time — or, for a record with zero airborne time, its block time. **Nothing reads the gap
between one leg and the next.** Once V275 stops the aeroplane closing ground-run legs at all, fuel
burnt and engine time run between legs are billed by nothing and accrued by nothing. They show only
as a break in the Carburant columns and a step in the TACHO column.

**The fuel half needs no assumed burn rate, because the FP-5L measures it.** The gap is
`previous leg's fp5lArr − this leg's fp5lStart`, in litres, off the instrument. Proposed rule, not
built: when that difference is positive and the pilot declares no uplift between the two, bill it at
the departure field's price and label it on the review page as fuel burnt between legs. At Cascais's
€2.30/L a 3 L gap is **€6.90** gross, a 10 L gap **€23.00**. This session has **no measured ground
burn rate for F-BVNK and does not assume one** — the one real ground run carries a refuel
(fp5l 231 → 234), so no burn is readable from it at all.

**The hours half is a correction, not a confirmation.** The PM wrote: *"The engine reserve accrues
on TACH, which includes ground running; confirm that is intended."* It does not, in the laptop book.

- `PRICES_AND_BILLING_RULES.json` → `hours_sources.reserves_accrue_on` says **`NK_ST_TACH` deltas**.
  That is the cockpit strip's rule and it is right for the strip.
- `FBVNK-LEDGER.html` accrues on `leg.hours` — **airborne time**. So do the wear model, the sinking
  funds and the 25 / 50 / 100-hour inspection counters.

**The two halves of the same project disagree on the basis, and the difference is measured off
Claus's own page 86**, not estimated:

| | |
|---|---|
| Tacho over page 86's eight legs | **7.9200 h** (1236,24 less 1228,32) |
| Flown over the same eight legs | **6.6667 h** |
| Extra tacho per leg | **0.1567 h** |
| Tacho as a ratio of flown | **1.18799 — 18.80 % more** |

At the kit's own reserve rates, engine €20.79 + propeller €1.36 = **€22.15 per hour**:

| Basis | Over page 86's eight legs | On Claus's real 2025 year, 73.00 flown hours |
|---|---|---|
| Airborne — what the book does | €147.67 | — |
| Tacho — what the kit says | €175.43 | — |
| **Not accrued** | **€27.76**, or **€3.47 per leg** | **€303.98** a year — €285.31 engine, €18.66 propeller |

The inspection counters carry the same difference: 100 flown hours is 118.80 tacho hours, so an
inspection that should fall on tacho arrives **15.82 flown hours late** on the book's basis.

Every figure above was computed in Python `Decimal` with `ROUND_HALF_UP`, from page 86's own tacho
and flown totals and the book's own rates. None is an estimate.

**This is a decision for Claus, and it is not this session's to take.** The case for airborne is
that Claus keeps his carnet on airborne time and said so; the case for tacho is that an engine wears
while it idles and the real reserve rates were set against a real aeroplane's real hours. What this
session will not do is quietly leave the book and the strip billing on different clocks. **For the
board, through the PM.** The book is unchanged pending his answer, and the divergence is now written
into both `PRICES_AND_BILLING_RULES.json` and `STRIP_SPEC.md` so neither can be read without it.

### What Tarbes exposed

The first real record departs and arrives at 43.185 N 0.002 W — **Tarbes LFBT**, which is in neither
the book's field table nor R17. It resolves to nothing, files as `????` and prices at the default
field, all of which is by design and labelled. But the review page's aerodrome picker offered
**only the eight codes already in the table**, so Claus could not have named it. On his first night
with the converter live, every leg from Tarbes would have filed as `????` with no way to fix it.

Fixed, and this one is this session's own initiative rather than an instruction — say so and reverse
it if it is wrong. The picker now has a box beside it that takes any four-letter code. Typing
`LFBT` learns the position against that code, the carnet row reads `LFBT`, and the next leg from
there resolves by itself — verified at **0.4 nm** and **0.0 nm**. It invents no fees: an aerodrome
with no entry in the field table is priced at the default and the card says so outright, because a
made-up landing fee would be worse than an honest default.

Two defects turned up in building it and are fixed: naming a field fired twice (the keystroke's
`change` and the native one on blur), so the note landed twice; and re-rendering the card from
inside a handler on an element inside that card threw a DOM error. Both are gone, verified.

**Still needed for LFBT, and not invented here:** its ARP from the French AIP, and its landing,
parking and fuel figures. The ARP is Records' kind of work — R19, if the PM wants it. The fees are
Claus's or the PM's.

### The two real-clock fields

`realEngineStart` and `realShutdown`, the PC clock in UTC at the logger rows that opened and closed
the leg. Added as optional. **The sim clock stays what the carnet prints** — the carnet is the
aeroplane's book and it is her clock that belongs in it — and the PM says the question of whether
Claus would rather have the real one has not been put to him. So nothing switches. The book carries
both and the review page says so only where they differ: *"The PC clock and the aeroplane's clock
differ on this leg: she says 09:00 → 10:00, the laptop says 21:03 → 22:04. The carnet takes her
clock."* Absent, and on `state.CFG` legs which have neither, nothing is said. An unparseable value
is dropped rather than half-read.

## What revision 4 changed

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

## The two questions only the book could answer

PM, 16 Sep 18:2x. Both were checked **in the running book in a browser**, not read off the source,
because the last time this session answered a contract question from the source it got C1 backwards
and Records had to catch it.

### 1. `notes[]` — accepted, and silently dropped. Now read and shown.

**What it did before this revision:** a record carrying `notes[]` filed without complaint and the
array went nowhere. `normaliseLeg` reads named fields off the record and nothing else, so an unknown
key raises no error and leaves no trace. Proved by ingesting a record whose note carried a unique
marker string and searching the whole rendered page for it: **not on the review card, not anywhere
in the document.** No page error either — which is the worst shape of all, because a converter
would have had no way of knowing its notes were being thrown away.

**The PM's proposal is the right one, and it is built.** `notes[]` is now an optional field of
`FBVNK_LEG/2` and the book shows it on the review page. Full rules in `LEG_CONTRACT.md` →
*`notes[]` — the converter's own notes*; in short:

- strings only, in order, **12 at most**, **160 characters each**;
- non-strings, empties and overflow are dropped **and counted out loud** in the book's own notes —
  *"4 of the 5 notes on the record are not shown — not a string, empty, or past the twelfth"*;
- a `notes` that is not an array is refused with a line saying so, rather than ignored;
- escaped on the way out — `<img src=x onerror=…>` renders as text and creates no element.
  Verified: zero elements created.

**They are kept in their own array, not merged into the book's notes.** The book's notes are what
the book decided (*"assumption: asphalt at LPCO"*); `notes[]` is what the laptop observed
(*"FP-5L gap of 3 L at 09:41Z, not counted"*). A pilot signing the page should be able to tell the
two apart, so the converter's notes print above the book's, under their own heading
**Du convertisseur**, in darker ink. Screenshot in the branch at `fbvnk-sim-ledger/srcnotes-card.png`.

**One thing for the Builder.** `note` (singular, string) and `notes[]` (array) are different fields
and go to different places: `note` is the carnet's **Incidents · Observations** column, `notes[]`
is review-page only and never enters the carnet. The adjacency of the two names is a trap; the
contract now says so in both places.

### 2. `burnSource: null` with `burnL` absent — accepted. No change needed.

`burnSource` is validated by membership in `['logger','aircraft','rem','uplift']`, so JSON `null`
is simply not a member and the book carries on with no declared rung. `burnL` absent gives `null`
from `numOrNull`. The chain then runs from the top and takes the first rung the record supports —
which is exactly V23's intent.

Tested through `JSON.parse`, so the `null` is a real JSON null and not a JavaScript literal. All
three V23-family shapes, each run twice — once with `burnSource: null` and once with the key
absent:

| Record | `burnSource: null` | key absent |
|---|---|---|
| FP-5L 200 → 158, no refuel | `rem`, 42.0 L | `rem`, 42.0 L |
| refuel, uplift 60 | `uplift`, 60.0 L | `uplift`, 60.0 L |
| refuel, no burn, no uplift (**V23**) | nothing billed | nothing billed |

Identical in every row. `burnL: null` written out explicitly behaves the same. **The Builder's
shape is fine as it stands — no change to the converter.**

Two things the Builder should know before leaning on it, both now in `LEG_CONTRACT.md`:

- **`null` with a `burnL` present is labelled `logger`.** With no rung named, the book takes a
  present `burnL` as the logger's, because that is the only rung that can produce one without the
  aeroplane's own figures. If the converter ever derives a `burnL` any other way, it must name the
  rung — otherwise the review page credits the logger for a number the logger never made.
- **`"none"` as a string would work today for the wrong reason.** It is ignored as any
  unrecognised string is, so it behaves like `null` and would break silently if `"none"` ever
  became meaningful. JSON `null` is the value.

### The import facts, recorded

Now in `LEG_CONTRACT.md` → *What the converter leaves on disk*: `LEG.json` is the latest conversion
only and is what the poller reads; the dated `<YYYY-MM-DD_HHMMSS>.json` files are the complete
record and are safe to give the book, all of them, more than once, because `legId` dedupes;
`REJECTED_<stamp>.json` is the converter's own reject pile and **never goes to the book**. `legId`
is `"YYYY-MM-DD/HH:MM:SSZ/tach"`, worked example `"2026-09-16/08:20:16Z/1228.32"`. The book treats
it as an opaque string and compares it whole.

## What's in this folder

| File | What it is |
|---|---|
| `PRICES_AND_BILLING_RULES.json` | The single price/rules source both the laptop ledger and the future HANGAR strip read. Prices transcribed verbatim from `FBVNK-LEDGER.html` commit `5e72853`, and in this revision checked against Claus's own 2025 accounts where an invoice exists for them. Wear-rate normalisation, posting-cursor, Reset-to-delivery, rounding and the hours/faults/battery rules cite `SPEC.md` (R5) and the K7/K7b reviews inline. |
| `TEST_VECTORS.json` | Thirty-five sequences (V1-V34, with V2 split into 2a/2b) of LocalVar reads → expected postings in euro. New this revision: **V18** the real Spanish invoice at 21 % and a `.xx5` rounding case, **V19-V23** the `burnSource` fallback chain in order including the two cases a mid-leg refuel breaks, **V24** the aircraft rung barred by a refuel, **V25-V26** pro rata rounding net-first, and — answering the PM's two questions — **V27** the `notes[]` field with its caps, its drop-counting and its escaping, and **V28** `burnSource` JSON `null` proved identical to the key being absent. New in revision 5: **V29** the carnet entry being the airborne event and not the take-off flag, **V30** the zero-airborne refusal that would have lost the first real record, **V31** the two real-clock fields, and **V32**, the accrual-basis disagreement, now rewritten as CLOSED with Claus's words. New in revision 6: **V33** the meter's progression across a ground run and the no-reading carry, and **V34** the meter going backwards. **V6** pins the rounding mode, **V14** the overhaul consuming its own step, **V16** the repriced brakes. |
| `LEG_CONTRACT.md` | Contract `FBVNK_LEG/2` in one file — every field, where V274 stores it, what the book does without it, the fuel chain, the touchdown clamp, the time base, and what the converter must derive. New in this revision: the optional **`notes[]`** field with its caps and its drop-counting, `burnSource` **`null`** stated as a value in its own right, and **what the converter leaves on disk** — which of `LEG.json`, the dated files and `REJECTED_*.json` may be given to the book, and the `legId` format that makes a re-drop harmless. New in revision 5: **what reaches the carnet** — the airborne-event rule, the air start, the contradictory record, and the refusal that is fixed — and the optional **`realEngineStart` / `realShutdown`** pair. New in revision 6: **the tacho is now the money** — the progression rule, its four cases and what it asks of the converter — **what the aeroplane saves under V275** with its known limit, **what the review page shows** and why it is not the carnet's format, and the optional **`legNo`**. The converter kit starts from this page and Records reviews against it. |
| `STRIP_SPEC.md` | What the HANGAR strip shows, its scope, the rounding rule it must share with the laptop book, what pro-rata pricing costs it in new persisted state, and the three capture routes including the one Records found at the SERVICE action. New in revision 5 and settled in revision 6: **the accrual clock** — Claus picked the hour meter, the strip was already on it and does not change, and the laptop book moved to it. No package code. |
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

## Review after each flight — what the page does today, and what it does not

Claus, 16 Sep 18:04:32, alongside his P1 answer: *"I assume it is possible to always after the
flight is ended (batt master off) to see the prefilled (auto by you) information, and if needed be
make other selections, so we'll need to figure out what those selections/choices should be … also
perhaps the pilot wants to enter manual remark, etc. Maybe u already set this up, I haven't checked
yet."*

**Design proposal only. Nothing below is built.**

### What is already there

He has not looked yet, so here is the page exactly as it stands with an automatic leg, read off
the running book rather than described from the source:

- It opens by itself the moment a record arrives, headed *"Carnet de route · filed by the aeroplane
  · read it and sign"*.
- It shows, prefilled: route with off/on times, **Date**, **Durée**, **Nature**, **Carburant**
  (litres and which rung produced them), and an **AUTO — as F-BVNK recorded it** block with
  **Moteur**, **En vol**, **FP-5L**, **Touché**, **Tacho** and the record's **Réf.**
- It lists every `assumption:` it had to make, in plain words.
- It lists **Entered this leg** — every invoice and tech-log line the leg posted.
- It has a signature line, and **Sign & reveal cost**, which is when the money appears.
- Where a position did not resolve it offers a picker, and naming the aerodrome teaches the book.

### What is missing, and it is exactly what he asked for

**The only thing on that page he can change is his own name.** Everything else is read-only. There
is **no free-text remark box** and **no selections at all**.

And the page is already telling us what the selections should be: the three `assumption:` lines it
prints on a typical leg are the fuel price, how she was flown, and the surface — each one a thing
the aeroplane cannot know and the pilot can. The design writes itself: **turn each stated
assumption into a choice, and add the remark.**

### Proposed selection set

Labels follow his carnet columns. Defaults are what the book already picks, so signing without
touching anything gives exactly today's behaviour.

| Label, as in his carnet | Options | Default the book pre-selects | Money? |
|---|---|---|---|
| **Nature du Vol** | P1 · P1 (école) · P2 · double commande · essai · convoyage | `P1` — every page 86 row is P1 | no |
| **Fonctions** | PIC · PICUS · dual · SPIC | `PIC` — as page 86 | no |
| **Équipage — Noms** | him alone, or a second name typed in | his name, ditto after the first row | no |
| **Incidents · Observations** | **free text**, his words, into the carnet column | `NIL`, or whatever the record carried | no |
| **Comment elle a volé** | gentle · by the book · pressed on | from `power` if sent, else *by the book*, **labelled** | **yes** — wear |
| **Surface** | asphalt · grass · gravel | asphalt, **labelled** | **yes** — tyre and prop wear |
| **Nuits stationnées** | 0, 1, 2 … | from the record, else 0 | **yes** — parking fee |
| **Handling** | no · yes | no | **yes** — €95.00 |
| **€ par litre** | the standing price for the field, or typed | the field's price, **labelled** | **yes** — fuel bill |

Three notes on that table. The four money rows are the only ones that can change what the leg
costs, and they are exactly the four the page currently prints as assumptions — so the change
removes three `assumption:` lines rather than adding new ways to be wrong. The remark is the one
Claus asked for by name and is free text, not a menu. And `Visa` stays empty: the column's own
printed sub-head on his page 86 reads *Douanes et Autorités Aéronautiques*, and nothing in a sim
fills it.

### R18 landed — what it changes in that table, and what it does not

Records' R18 (`FBVNK_STAGING/R18_easa_logbook/NOTE.md`, md5 `8d275080`) read AMC1 / GM1 FCL.050 off
the EASA Easy Access Rules HTML, Revision from August 2023, and FCL.050 off the EUR-Lex consolidated
text of 30.04.2026. Records states its own currency limit: **whether AMC1 / GM1 FCL.050 changed
after August 2023 is not verified** — the newer text is PDF/XML only and was not fetched under the
HTML-only rule. Everything below inherits that limit.

**One correction to the table above, and it is this session's error.** *Fonctions* was offered as
`PIC · PICUS · dual · SPIC`. AMC1 FCL.050 (i)(10) says **PIC, SPIC and PICUS are all entered as
PIC**, with SPIC/PICUS countersigned by the PIC or FI in the remarks column — so PICUS and SPIC are
not peers of PIC and should not be offered as if they were. The four names printed in column 10 of
the AMC's own format are:

| Row | Was | Is now | Why |
|---|---|---|---|
| **Fonctions** | PIC · PICUS · dual · SPIC | **PIC · co-pilot · dual · instructor** | the four printed in AMC1 col 10, *PILOT FUNCTION TIME*. Default stays `PIC`, as page 86. No money. |

**Nature du Vol is not widened, because R18 gives nothing to widen it with.** R18 finds **no direct
EASA column** for it: the nearest items are col 9 *OPERATIONAL CONDITION TIME (NIGHT, IFR)* and the
col 12 remarks, which GM1 allows for "the specific nature of a particular flight". So the options
stay as read off page 86.

**Two rows R18 makes available, and one reason to hesitate.** The EASA items with no carnet column
that a sim leg could actually fill are col 8 *LANDINGS DAY / NIGHT* and col 9 *OPERATIONAL CONDITION
TIME NIGHT / IFR*:

| Label | Options | Default | Money? |
|---|---|---|---|
| **Atterrissages — jour / nuit** | two counts | the record's `landings` in the day box, 0 at night | no |
| **Nuit / IFR** | two times, h:mm | 0 / 0 | no |

The hesitation: **those are pilot-logbook columns and this is an aircraft journey record.** Records
reaches the same reading, labelled as an assumption — Carburant, Huile, Incidents and Visa with no
aircraft type or registration column look like a *carnet de route*, not a pilot logbook. Claus's
document is titled *Carnet de Route* and carries F-BVNK's own hours, so the two are different
records and the EASA items are a second one, not a replacement. **Proposal: capture both on the
review page, and keep them off the carnet rows**, which stay the seventeen columns of page 86.

**Flight time: the definitions differ, and the page should say which it is using.** EASA flight time
runs "from the moment an aircraft first moves for the purpose of taking off until the moment it
finally comes to rest" — nearer block than airborne. Claus, 16 Sep: *"I am using airborne (flight
time) and not block time."* His carnet keeps airborne. The book already holds both and shows both on
the review card (*En vol* and *Moteur*), but **Durée** is printed with no statement of which it is.
Proposal: label it — *Durée (en vol)* — and leave the totals airborne, as he asked.

**Labelling, from Records' conclusion.** A home MSFS leg is **not flight time** (FCL.010 defines it
by a real aircraft moving) and **not an FSTD session** (col 11 needs a qualification number, and
Art. 10b makes qualification a condition). R18 states plainly that **how to log a non-qualified home
simulator is not stated in the rules.** Records' recommendation, which this session agrees with:
keep this book separate from Claus's EASA logbook, and label it. Proposal for the board — a single
line at the head of the Logbook tab and on the review card:

> **Home simulator record — not an EASA logbook.** MSFS 2024, non-qualified device. Not flight
> time, not FSTD time.

That line is **not built**. It changes what Claus's carnet says about itself, so it goes to the
board through the PM like the rest. It is the one row here this session would ship without waiting,
because it can only make the record more accurate about what it is.

**One thing back to Records.** R18 marks *Visa* "not obvious" with the assumption that it means a
countersignature, noting "I have not seen the carnet itself." The carnet settles it: the column's
printed sub-head on page 86 reads **Visa · *Douanes et Autorités Aéronautiques***. It is the customs
and aeronautical-authorities column, not a countersignature, and the book has carried that sub-head
since it was built from the scan.

### Ordering, if this is built

The selections must sit **before** the signature and before the cost is revealed, because four of
them change the cost. The flow stays: record arrives → page opens prefilled → he adjusts what the
aeroplane could not know → signs → cost. That is the order he described.

## Open — needs a person, not a guess

Three. Revision 6 closed the accrual clock — Claus picked the hour meter at 21:05:42 — leaving the
two long-standing ones and Tarbes. Revision 4 closed three of revision 3's five: the aerodrome coordinates and the `LEAP` /
Teruel naming, both under *Closed since revision 3* in `LEG_CONTRACT.md`, and the carnet's time
base — **Claus tapped UTC on 16 Sep at 18:00**, the book is set to it, and the control stays.
Revision 5 opens two new ones, 3 and 4 below, and neither is a gap this session could fill by
choosing.

1. **The nine prices — proposed, for Claus to check.** Carried out this revision, against his
   pick of 16 Sep 18:00:04: *"Cost chat proposes assumed prices, clearly labelled, for me to
   check"*. The full table with every derivation is in `PROPOSED_PRICES.json`; nothing is in
   `PRICES_AND_BILLING_RULES.json` and **nothing bills** until he has said yes.

   The PM asked for five. It is **nine** — the five `PRICES` keys (carburettor, starter, oil
   pump/lines, exterior lamps, oil top-up per quart) **and** four fault components (static port,
   pitot, COM wiring, NAV wiring), as this NOTE has said since revision 5.

   Going back to his own invoices in Dropbox moved most of them off guesswork:

   | | items | money comes from |
   |---|---|---|
   | invoiced outright | oil per unit, exterior lamps | his own invoice lines, quoted |
   | labour + parts both real | COM wiring, NAV wiring | real rate x hours, real parts lines |
   | labour real, no parts | static port, pitot | real rate x hours |
   | labour real, parts a guess | carburettor, starter, oil pump/lines | **the parts half is unsourced** |

   So **three** of the nine still carry a money figure with no source behind it, and only the
   parts half of those three. Everything else is his own paperwork.

   Rates and parts taken from the invoices, quoted exactly:

   - `01010301 MAO-DE-OBRA - TECNICO 1ª CAT. - MANUTENCAO` — **52,00 EUR/h** (FAC 25/100,
     2025-07-11; FAC 26/6, 2026-01-23). It was **48,00 EUR/h** in FAC 24/122 (2024-07-26) and
     still 48,00 in FAC 25/64 (2025-05-14), so the rate rose between May and July 2025.
   - `01010302 MAO-DE-OBRA - TECNICO AUXILIAR - MANUTENCAO` — 32,00 EUR/h (FAC 24/122).
   - `06030005 OLEO DE MOTOR - AERO DM 15W50` — **14,63 EUR**, 8,00 UNI a change (FAC 25/100 and
     FAC 26/6, identical on both).
   - `03320013 FILTRO DE OLEO - AA48110` — 56,50 EUR (both).
   - `09999998 MATERIAL DE SOLDADURA` 15,00 EUR and `05220054 TERMINAL FICHA FEMEA
     2,5MM^2X6.3MM` 0,85 EUR (FAC 24/122) — these price the wiring repairs' parts.
   - `01010301 ... SUBST. LAMPADAS NAV, FAROIS ATERRAGEM E TAXI 3,00 H 48,00 EUR 144,00 EUR`
     (FAC 24/122) — the lamps job's own labour line.

   **assumption:** that one `UNI` of `OLEO DE MOTOR` is one US quart. The invoice unit is `UNI`,
   not `QT`, and 8,00 a change is consistent with a full sump, but the invoice does not say it.
   Worth Claus's eye, because `STRIP_SPEC` §2.2–2.4 models the oil at **7 qt** and his own oil
   change buys **8**.

   **assumption:** the labour hours in the table are this session's, not his. The rate they are
   multiplied by is real.

   **Checked and found correct, not an error:** `insp25` carries net 168,00 citing FAC 25/64,
   whose filename reads `206,64`. The invoice is `3,50 H x 48,00 = 168,00` net, IVA 38,64,
   TOTAL 206,64 — the filename is the gross. The book is right.
2. **Which capture route the Builder takes** — the accumulator, the snapshot, or Records' capture
   at the SERVICE action. This session recommends the action capture *combined with* accumulation,
   because the action sees the exact value at the service while a mixed-R life still needs
   per-read division. Not chosen here.
3. **Tarbes LFBT.** Its ARP from the French AIP, and its landing, parking and fuel figures. The
   first real record off the aeroplane departs and arrives there. It files as `????` and prices at
   the default field until both are supplied, and Claus can now name it on the review page, which
   fixes the carnet row and the next resolution but not the fees. The ARP is Records' work — R19 if
   the PM wants it. The fees are Claus's or the PM's.

None of these were guessed at to fill a gap — each is flagged instead.

## Verification

**Where the bytes are.** All five files are committed at `fbvnk-sim-ledger/KIT1_R6/` on
`claude/fbvnk-cost-sheet-maintenance-svcsw2`. That is the authoritative copy. This session is a
cloud session: its Dropbox connector creates files from inline text and **cannot upload a local
file**, so retyping the folder into the staging area would risk exactly the kind of drift that put
`STRIP_SPEC.md` one byte out on the R3 upload. The house rule covers this case and says to hand
over the branch plus checksums instead of retyping bytes, which is what this is.

The staging folder therefore carries `MD5.txt` and a pointer, not the five files. `md5sum -c
MD5.txt` run against a checkout of the branch reports all five OK; this session ran it and it does.

**KIT1 R6 REPORTED** — frozen from here; Records reviews it as K7e against the branch, covering
revision 5's A1–A3 as well, per the PM.

Every euro figure in this revision was recomputed independently in Python `Decimal` with
`ROUND_HALF_UP`, the decided mode, not in float: V1 513.65, V6 31.19, V15 1.80, V16 4.02,
V18 80.33 VAT on a printed gross of 462.83, and the two new net-first services at 13.39 and 200.99.
The laptop book's own `eur2()` gives the same answers on all 23 of its test cases.

**Everything claimed about the book's behaviour in this note was taken from the running book in a
browser, not from the source** — including both defects in revision 5, which is how they were found
at all: A2 turned up only because the Builder's first real record was put through the book verbatim
rather than read against the contract. The regression set was re-run after every change and is
unmoved: C1 the touchdown clamp, C2 the refuel guard, the timezone conversion, page 86's rows, the
V274 flags, the rounding cases, and the `notes[]` set. The seeded book stands at **8 legs, 16
invoices, 18.4 hours** after all of it, and no page raised an error.

**The revision 4 answers were taken the same way.** V27's cases were checked by ingesting records and reading the rendered review card and
the whole document; V28's six runs went through `JSON.parse` first so the `null` was a real JSON
null. The seeded book was re-counted after every change and is unmoved at **8 legs, 16 invoices,
18.4 hours**, and the page raised no error on any of it. The reason for doing it this way is C1:
the last contract question this session answered from the source came out backwards, and Records
had to catch it.
