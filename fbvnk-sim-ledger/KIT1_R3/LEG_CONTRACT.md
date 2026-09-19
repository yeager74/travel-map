# LEG_CONTRACT — `FBVNK_LEG/2`

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
| `burnSource` | enum | converter | `logger` \| `aircraft` \| `rem` \| `uplift`. See the chain below. |
| `refuelInLeg` | bool | refuel-in-leg flag | Makes `rem` unusable. |
| `touchdownFpm` | number | last landing V/S, **negated** | **Positive is descent.** Clamped at 0; see below. |
| `noTakeOff` | bool | `noTakeOff` flag | An engine run that never got airborne. Filed, and it posts, but kept out of the carnet — a ground run is not a flight. The Logbook page says how many are being left out. |
| `stopInAir` | bool | `engineStoppedInFlight` | Written into the carnet's Observations as *Arrêt moteur en vol*. Flagged, never billed, per the standing rule. |
| `uplift`, `fuelPrice` | litres, €/L | not stored | Optional. Price missing → the standing price for the destination, labelled. |
| `power` | object | not stored | `maxRpm`, `minutesAboveCruise`, `hardBraking`. Missing → "by the book", labelled. **The thresholds in the book are this session's proposal, not measured off the aeroplane.** Set the real ones, or send `style` outright and skip the inference. |
| `surface`, `nights`, `handling` | string, int, bool | not stored | Optional. Surface missing → asphalt, labelled. |
| `conditionBefore`, `conditionAfter` | object | not stored | For a leg carrying a service: the eleven `NK_ST_C_*`/`NK_ST_OIL_QT` values plus the R in force, so a pro-rata posting is computable. Comes with the strip kit. |

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
| `aircraft` | no logger run, but V274 saved the leg's own burn | V20 |
| `rem` | no burn figure at all **and** `refuelInLeg` is false | V21 |
| `uplift` | last resort | V22 |
| — | a refuel, no burn figure and no uplift: **nothing is billed**, and the page says exactly that | V23 |

`burnL` inherits the FP-5L's own gaps whatever the source: a single-update drop of 2 L or more is
never counted, and nothing counts while the instrument is unpowered. REM and USED can therefore
disagree, which is the reason `burnSource` exists at all.

## Touchdown rate

Positive is descent. The converter negates the sim's own negative vertical speed.

- **0 or positive after negating** → 0, labelled "a float, or a very soft arrival". Not a hard
  landing by any reading.
- **Negative** → clamped to 0 and said out loud, because it means one of two things: a converter
  that forgot to negate, in which case every landing would come through soft and the hard-landing
  check would silently never fire again; or an arrival with no descent at all. Worth checking the
  converter either way.
- **Missing** → 150 fpm, labelled, and the page states plainly that no hard-landing check was made.

## Times

UTC. The aeroplane's clock is Zulu, so every time field is Zulu and a leg carries `tz: "Z"`.

The carnet has a UTC / local control, **UTC by default**, so whichever way Claus answers on the
board it works with no rebuild. A leg typed into the book's fallback form carries no zone and is
never converted — a conversion needs a base, and guessing one would put a wrong time in the
carnet. The column head says which base is showing. Checked in Europe/Lisbon: 0800/0918 Z shows as
0900/1018 local, and the hand-filed rows do not move.

## What the converter has to derive

Per K17b §7, and the book assumes all of it is done before the record reaches it:

- the **year** — V274 stores the Zulu day of year only;
- the **next day** when arr, take-off or landing seconds fall below dep's (midnight wrap);
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

## Open against this contract

1. **Aerodrome coordinates.** No latitude or longitude is on file for any aerodrome, and this
   session will not put a position on a real one from recall. Until they are supplied every
   position resolves to `????`. Eight pairs wanted: LPCS, LPCO, LPVL, LEVD, LEAP, LEST, LEBG,
   LEMP.
2. **UTC or local in the carnet.** On the board. The control makes either answer cheap.
3. **`LEAP` / Teruel.** The book's field table calls `LEAP` "Teruel", but Claus's own flight
   planning has both `LPCS - LETL` and `LETL - LEAP` for 24 Oct 2025, which are two different
   aerodromes. At least one name is wrong. Flagged, not corrected.
4. **Whole litres.** The REM figures are whole litres, the PM's decision; the burn is to 0.1 L.
   The book prints what it is given. Claus can ask for decimals.
