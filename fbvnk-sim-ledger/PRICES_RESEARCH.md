# Working notes — the nine proposed prices

Claus, Annunciator, 16 Sep 2026 18:00:04, relayed word for word by the PM:
**"Cost chat proposes assumed prices, clearly labelled, for me to check"**.

This is working material for that round, not a kit file. Nothing here is in `KIT1_R4/`, and
nothing here posts: the book keeps *flag, bill nothing* for all nine until Claus confirms.

## The rule I am working to

The PM's instruction: a figure from Claus's own documents where one exists; else a public list or
catalogue price **from an HTML page with its URL**; **never recall**. Parts and labour separately
where possible.

## A constraint that changes half the job

**`WebFetch` is blocked by this environment's egress proxy** — `aircraftspruce.com` and every other
domain tried returns 403 at the CONNECT tunnel. `WebSearch` works, but it returns a *model summary
of snippets*, not a page read: I cannot open a catalogue page and quote its price verbatim, which
is what the instruction asks for.

So the nine split into two groups, and I will not blur them:

- **Sourceable from Claus's own records** — real figures, quoted below.
- **Not sourceable here** — carburettor, starter, oil pump/lines, and the four fault components
  (static port, pitot, COM wiring, NAV wiring). For these I can offer a search-summary figure with
  its URL, clearly labelled as *not a page read*, or nothing. Recall is not an option and I will
  not use it. **This needs the PM's or Claus's call**, and it is the one thing blocking a complete
  table.

## Sourced from Claus's own documents

All from `AIRWORTHINESS - DOCS/2024-JAN-DEC-2024-Aircraft-Hourly-Operating-Cost-Calculator-F-BVNK.xlsx`
unless stated. Figures quoted exactly as they appear.

| His record | Figure | VAT flag | Bears on |
|---|---|---|---|
| ACTUAL COSTS, `LED LIGHTS AERO-LITES` | € 750 | Y | **exterior lamps** |
| ACTUAL COSTS, `BATTERY GILL G35` | € 416 | y | the book's `batt` — see below |
| ACTUAL COSTS, `RH TOE BRAKE PARTS` | € 3,000 | n | the brakes entry — see below |
| PLAN sheet, `Oil Change Cost / 50H inspection` | $ 630.00 ex VAT, parts **and** labour | — | **oil top-up per quart** |

And from the 2024 lamps job, via `FBVNK_STAGING/nk_lights/ANSWER.md`, which verified each against
the signed scans:

- **Aero-Lites order 22473, 1 May 2024** — nav set (red/green/white) USD 226.00; taxi `AL-3615X-F`
  USD 155.00; landing `AL-3615X-S-PA` USD 165.00. Sub USD 546.00, shipping 54.99, **total USD 600.99**.
- **IAC `FAC 24/122`**, labour line, quoted exactly:
  `SUBST. LAMPADAS NAV, FAROIS ATERRAGEM E TAXI 3,00 H 48,00 EUR 144,00 EUR`

**That labour line gives IAC's own rate: € 48.00/h.** Worth having on its own — it prices the
labour half of any assumed job from a real invoice instead of an invented rate.

## Two things this turned up that are about figures already in the book

Neither is part of the nine. Both are raised because they concern real money.

1. **The battery.** The book carries `batt` at net 310.00 @ 23 % = **€ 381.30**. Claus's own 2024
   record for a Gill G35 is **€ 416**. The book is € 34.70 under his own figure, and `batt` is
   currently labelled an assumption when a real number exists.
2. **The brakes, again.** His 2024 sheet has `RH TOE BRAKE PARTS € 3,000`. That is a real
   brake-related spend, and it is not the € 1,340 the kit carries. `assumption:` a toe-brake
   assembly is not the pads/calipers/bleed overhaul the kit prices, so these are probably two
   different jobs — but after F1 found no invoice behind the € 1,340 at all, a € 3,000 brake line
   in his own records deserves a second look rather than being passed over.

## Closed, 17 Sep 2026 — the invoices themselves

The PM widened the acceptable basis to include **invoice analogy**, and that unblocked it. Rather
than analogise, I went to his actual IAC invoices in Dropbox and read them. Four were fetched in
full: **FAC 26/6** (50 h, 2026-01-23), **FAC 25/100** (annual, 2025-07-11), **FAC 24/122**
(annual, 2024-07-26), **FAC 25/64** (25 h, 2025-05-14).

WebFetch is still blocked — re-tested 17 Sep, `aircraftspruce.com` returns 403 at the CONNECT
tunnel — so no catalogue price is quoted anywhere. None is needed for six of the nine.

### What the invoices gave, quoted exactly

| Line | Figure | Bears on |
|---|---|---|
| `01010301 MAO-DE-OBRA - TECNICO 1ª CAT. - MANUTENCAO` | **52,00 EUR/h** (FAC 25/100, FAC 26/6) | every labour half below |
| same, earlier | 48,00 EUR/h (FAC 24/122, FAC 25/64) | the rate rose between May and July 2025 |
| `01010302 MAO-DE-OBRA - TECNICO AUXILIAR - MANUTENCAO` | 32,00 EUR/h (FAC 24/122) | — |
| `06030005 OLEO DE MOTOR - AERO DM 15W50` | **14,63 EUR**, 8,00 UNI a change (both) | **oil per quart — invoiced, not assumed** |
| `03320013 FILTRO DE OLEO - AA48110` | 56,50 EUR (both) | — |
| `09999998 MATERIAL DE SOLDADURA` | 15,00 EUR (FAC 24/122) | COM/NAV wiring parts |
| `05220054 TERMINAL FICHA FEMEA 2,5MM^2X6.3MM` | 0,85 EUR (FAC 24/122) | COM/NAV wiring parts |
| `01010301 ... SUBST. LAMPADAS NAV, FAROIS ATERRAGEM E TAXI` | `3,00 H 48,00 EUR 144,00 EUR` | the lamps job's labour |

The $ 630 / 50 h PLAN line is **not** used. It would have needed an FX rate, an assumed labour
time and an assumed filter price stacked to produce one number; the real 14,63 EUR line makes all
three unnecessary.

### Where the nine landed

Two invoiced outright (oil per unit, lamps). Two with real labour *and* real parts (COM, NAV
wiring). Two labour-only at his real rate (static port, pitot). **Three** — carburettor, starter,
oil pump/lines — still carry an unsourced figure, and only in the parts half.

Full table with every derivation: `KIT1_R10/PROPOSED_PRICES.json`, every row `approved: false`.

### Raised in passing

- **`insp25` is correct.** Its 168.00 cites FAC 25/64 whose filename reads `206,64`; the invoice
  is `3,50 H × 48,00 = 168,00` net, IVA 38,64, TOTAL 206,64. The filename is the gross. `insp50`
  786.88 and `insp100` 2 434,99 also reconcile to their invoices to the cent.
- **The oil quantity disagrees with the model.** `STRIP_SPEC` §2.2–2.4 has 7 qt; his oil change
  buys 8,00 UNI. `assumption:` that one UNI is one US quart — the invoice does not say so.
- The battery and brake points below still stand and are still not part of the nine.
