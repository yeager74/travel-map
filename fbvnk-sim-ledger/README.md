# F-BVNK Sim Ledger

A live operating-cost and maintenance book for the MSFS 2024 Socata Rallye MS-893E
`ah-aircraft-socms893-F-BVNK`, built to the shape of F-BVNK's real 2025 accounts.

One file: **`FBVNK-LEDGER.html`**. No install, no dependencies, no network.

---

## Install

Drop `FBVNK-LEDGER.html` anywhere inside the package folder and open it from there
(double-click, or right-click → Open with → your browser). Suggested home:

```
ah-aircraft-socms893-F-BVNK\
├── manifest.json
├── layout.json
├── APPLY_COWL_REPAINT.bat
├── FBVNK-LEDGER.html      <-- here
└── SimObjects\Airplanes\ah-aircraft-socms893-F-BVNK\
```

It is inert as far as the sim is concerned: no `.json`, `.cfg`, `.ktx2` or `layout.json`
entry is touched, so it cannot affect loading. If it ends up listed in `layout.json`,
that is harmless, but it does not need to be there.

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
| AVGAS at Cascais | €2.29 / L average | 2025 cost calculator |

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

*assumption:* an automatic feed is straightforward — the ledger would read the `NK_*`
LocalVars instead of its own health figures — but the exact variable names, units and
scaling have not been confirmed, and nothing should be wired until they are. That belongs
to the package owner, not to this file.

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
