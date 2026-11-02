# Core measure rules

[← README](../README.md)

## Sales is one measure

Sales anywhere in this model means **`Sales Net Price`**: quantity × net price across
`FactSalesLine` under one filter, `DimDocumentType[IsSalesOut] = TRUE`. The sales rule — goods
only, the lines the source counts as sales — is applied in the source query, so no measure
repeats it. **Never rebuild this measure inline; call it.**

## Two sales bases

The prices on `FactSalesLine` include VAT, as the source stores them: `ListPrice` (before any
discount), `GrossPrice` (after the regular discount) and `NetPrice` (after the extra discount
too).

Beside `Sales Net Price` runs a family named **`ex VAT`**, which takes the VAT out line by line
with `FactSalesLine[ExVatFactor]`, `1 / (1 + VAT rate / 100)` from the source query.

- **`Gross Margin` divides by the ex-VAT figure**, and so does the plan margin — see
  [gross-margin.md](gross-margin.md#gross-profit-and-gross-margin).
- **The trade budget is without VAT on every part**, and its rates are charged on
  `Sales Net Price ex VAT` — see [gross-margin.md](gross-margin.md#the-trade-budget).
- **The effects in the tooltips read `Sales Net Price`**, with VAT, the one sales figure.
- **`Gross Margin`** A margin read against sales with VAT would
  not reconcile.
- Putting both bases in one visual compares two things silently; nothing errors.

## Cost and prices

- **Cost is without VAT.** `FactSalesLine[UnitCost]` is the average cost the source posts to
  the cost of sales; `Sales Cost Price` is quantity × it. It is never grossed up.
- **Every per-unit figure is value over quantity**, never an average over lines: unit × quantity
  must give back the money. An average over lines counts a one-piece line as much as a
  thousand-piece one. Sales per outlet in the channel tooltip follows the same rule: net sales
  over outlets, never an average of outlet figures.

## Time windows are functions

A time window is a call to a user-defined function, never written out in a measure.

| Function | Window |
|---|---|
| `YearToDay` | the date slicers' window within the current year, cut at today |
| `PrevYear` | the window one year back, inside last calendar year |

A prior year **to date** is `YearToDay` over `PrevYear`: last year, cut at today's date one year
back. The tooltips' effects and the kept / new / lost counts compare like for like on it.

- **Actual Year** in a measure name is `YearToDay`; **Prior Year** is `PrevYear`;
  **Prior Year YTD** is both.
- **Table-valued windows are functions too.** `YearToDaySet` and `PrevYearSet` carry the same
  filters through `CALCULATETABLE`, for the sets the kept / new / lost counts compare — see
  [gross-margin.md](gross-margin.md#kept-new-and-lost).
- `IndexDif` is the change against a prior value, (current − prior) ÷ prior, with guards for a
  blank or negative prior.

## Where a measure lives

Measures are foldered by job, not by page, so a new page adds no folder:

| Folder | Holds |
|---|---|
| `Calculation Roots\…` | the building blocks: sales, prices, the price calculation, profit |
| `Sales\…` | sales in a time window: `Actual Year`, `Prior Year` |
| `Margin\…` | gross profit and margin, actual and plan, with their indexes |
| `KPI\…` | outlets and products with a sale |
| `Trade Marketing\…` | the trade budget: accumulated, spent, credit notes, discounts, trade terms |
| `Lookup\…` | thin views of a root for the tooltip ([below](#tooltip-measures)) |
| `Formatting\…` | the colour measures and the slicer filter, hidden |

## Tooltip measures

The tooltips read their own measures, in `Lookup\Marketing`, `Lookup\Brand` and
`Lookup\Customer`: each is a root or a small calculation over roots (`Marketing Product Sales`
is `Sales Net Price Actual Year`, `Brand Numeric Distribution` is
`Numeric Distribution Actual Year`). A view carries its own caption and its
unit in the format — "of sales", "outlets", "products", "M" — without moving a measure the pages
depend on. **Never point a page visual at a `Lookup` view, and never move a root into
`Lookup`.** The value stays a number, so sorting, totals and colour still work.

## Colours

`RagColor` turns a value into a theme colour name — `good`, `bad` or `neutral` — so colours
follow the theme. An index's colour measure is generated from an annotation — see
[dax-quality.md](dax-quality.md#colour-measures).

## Formats

- **`NumberFormat`** shows at most three significant digits with K, M or B, as the measure's
  whole dynamic format string; it returns a quoted literal, so the measure stays a number
  [[dynamic-format-strings]](sources.md#dynamic-format-strings).
- **`IndexFormat`** shows an arrow and a percentage with one decimal, ▲ at or above its centre
  and ▼ below.
- **No currency symbol.** Values are plain numbers; the report shows no currency.
- **Text measures carry no format**: the verdicts, the brand name and the colour measures.
