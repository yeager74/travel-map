# F-BVNK Sim Ledger

A live operating-cost and maintenance book for the MSFS 2024 Socata Rallye MS-893E — the
mod's aircraft folder `socata-ms893-mod`, painted by the separate `ah-aircraft-socms893-F-BVNK`
Community package — built to the shape of F-BVNK's real 2025 accounts.

One file: **`FBVNK-LEDGER.html`**. No install, no dependencies, no network.

---

## Install

**Keep this file outside every package folder.** `ah-aircraft-socms893-mod` (the aircraft)
and `ah-aircraft-socms893-F-BVNK` (the livery, a separate Community package that points
back at it) are git-owned by the project manager — installs, `layout.json`, commits, tags.
This ledger is never a package byte, so it does not live inside the sim's Community folder
at all: put it on the Desktop, in Documents, wherever is convenient, and open it from
there (double-click, or right-click → Open with → your browser).

The one exception: if the wear-model integration is ever built as a genuine in-sim page
(the "in-sim page" route on the [Job Card](https://claude.ai/artifact/UKVvwBMiX6GUhiKD6HMNa3)),
that becomes real package bytes built and installed by the project manager — not this
file, and not something you place by hand.

The book is kept in that browser's local storage, keyed to the file's location. Move the
file and you start a fresh book — use **Year Book → Copy book as JSON** first, then
**Load book from JSON** at the new location.

## The loop

1. Fly.
2. Open the ledger, drop six numbers into **Ramp → Close out the leg**: block time,
   uplift in litres, price per litre, touchdown rate, nights parked, how you flew it.
3. Press **File the leg**. Fuel, landing and parking fees, hangar rent, inspections and
   wear are all posted for you, as invoices from the real suppliers.
4. Squawks appear in the **Tech Log**. Broken items ground her. Worn items can be carried,
   but the bill climbs 6% every leg you fly on them.
5. Condition falls. When it crosses a band, **Paint Shop** tells you which texture set the
   aeroplane should be wearing and hands you the `.bat` to swap it.

## Where the numbers come from

Everything marked <kbd>invoiced</kbd> in **Rule Book → What things cost** is quoted from
F-BVNK's own 2025 supplier invoices:

| Item | Figure | Source |
|---|---|---|
| Hangar & maintenance support, monthly | €1,000.00 net + 23% = €1,230.00 | Sevenair Group, SA — 13 invoices in 2025 |
| Airworthiness management | €720.00 net | IAC — `FAC 25/135` |
| 25 h check | €168.00 net | IAC — `FAC 25/64` |
| 50 h check | €786.88 net | IAC — `FAC 26/6` |
| 100 h / annual | €2,434.99 net | IAC — `FAC 25/100` |
| Brake overhaul | €1,340.00 gross | Nicolau, Valladolid |
| Insurance | €883.00 / year | 2025 cost calculator |
| Engine overhaul at TBO | €35,000.00 | 2025 cost calculator |
| Propeller overhaul at TBO | €2,000.00 | 2025 cost calculator |
| Engine reserve | €20.79 / h | 2025 cost calculator |
| Propeller reserve | €1.36 / h | 2025 cost calculator |
| Cascais landing | €22.24 | Cascais Dinâmica E.M., S.A. |
| AVGAS at Cascais, 2025 average | €2.29 / L | 2025 cost calculator |
| AVGAS at Cascais, current reference | €2.83 / L (€2.30 net + 23% VAT) | Claus, 15 Sep 2026 — this is the Ramp tab's live default; every leg's price stays editable |

Opening counters are the real ones as recorded for 2025: TTAF 1265, engine 317 h SMOH
(overhauled 17-Apr-18), propeller 528 h SMOH (overhauled 16-Nov-11).

The 2025 benchmark the Year Book measures you against, quoted exactly:

- **€25,716.30** gross across **42** invoices in the file — €352.28 per hour over 73.00 hours
- **€23,473.96** gross across the **39** invoices dated inside calendar year 2025 — €321.56 per hour
- Wet hourly cost of operation, per the aeroplane's own calculator: **€378.48**

Anything marked <kbd>assumption</kbd> has **no 2025 invoice behind it** and was set for the
sim — tyres, oleos, magnetos, alternator, vacuum pump, wash, repaint and the away-field
fees at Burgos and Coimbra. Change them in `PRICES` and `FIELDS` near the top of the
`<script>` block if you want a harder or a softer aeroplane.

## Sim integration

The ledger is the **money layer**. The wear model that lives in the aeroplane is the iPad
maintenance page (`HANGAR.js`, LocalVars prefixed `NK_`) and stays the single source of
truth for condition.

Today the two are joined by hand: you read the iPad page, you type the numbers into the
ledger. **Paste sim summary** will pull block time, litres and touchdown rate out of
pasted text to save typing.

The `NK_*` LocalVars are now fully specified (`SPEC.md`, project manager's Records &
Checks helper, md5 `c06f7305`) — names, units, ranges, writers and Reset-to-delivery
values, all confirmed against the package source, nothing guessed. An integration design
exists mapping every ledger figure to an `NK_` variable or marking it as having none, with
a wiring plan (an idempotent cursor on `NK_ST_MX_LOG` paired with `NK_ST_TACH`, a named
data route, one source of truth for prices). Full detail and current status:
[F-BVNK Ledger Job Card](https://claude.ai/artifact/UKVvwBMiX6GUhiKD6HMNa3). Nothing is
wired yet — the project manager reviews the design and Claus picks the route on the
[F-BVNK Annunciator](https://claude.ai/artifact/1aDxiL7JRjFx9igTtC6PvF) before any build.

## Paint states

| Condition | Band | Texture set |
|---|---|---|
| 80–100 | Fresh | new cowling, before bugs and wear |
| 55–79 | Flown | bugs and wear |
| 30–54 | Tired | weathered |
| 0–29 | Scruffy | heavy wear |

The swap script follows the existing `APPLY_COWL_REPAINT.bat` convention and expects the
sets in `_wear_sets\<band>\`. Those texture sets are not supplied here.

## NOT INSTALLED
