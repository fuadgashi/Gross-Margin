# Gross profit and margin

[← README](../README.md)

## Gross profit and gross margin

```
Gross Profit = Sales Net Price ex VAT − Sales Cost Price
Gross Margin = Gross Profit ÷ Sales Net Price ex VAT
```

- **Both sides are without VAT.** The net price on a sales line includes VAT, so the sales side
  is `Sales Net Price ex VAT`, the VAT of each line taken out; the cost (`UnitCost`) is without
  VAT as the source posts it. A margin read against sales with VAT would not reconcile
  ([measures.md](measures.md#two-sales-bases)).
- **Goods only.** Service and fixed-asset lines never reach the sales fact, so cost and price sit
  on one base.
- **The margin is a ratio**: on a total row it is the total profit over the total sales, never
  an average of the rows' margins.

## The plan margin: at the regular price

Beside the margin actually made runs the margin the price was built for, before any discount:

```
Gross Profit at Regular Price             = sum over sales lines: Quantity × (list price ex VAT − unit cost)
Gross Margin at Regular Price Actual Year = Gross Profit at Regular Price Actual Year ÷ Sales Selling Price ex VAT, this year to date
```

- **The same scope as the actual margin** — goods only, without VAT, sales-out documents — so
  the gap between the plan profit and `Gross Profit` is exactly the discounts given.
- The charts on both pages set the three side by side: this year, last year, and this year at
  the regular price.

## The extra discount against the marketing budget

The third chart sets `Extra Discount Actual Year` against last year, with
`Marketing Accumulated Actual Year` as a line: the extra discount is what the invoices give away
after the regular discount, and the marketing budget is what the price sets aside to pay for it
— see [The trade budget](#the-trade-budget).

## Year on year

Every figure of the profit tables is read this year to date against the prior year:

| Measure | Compares |
|---|---|
| `Index Sales Net Price ex VAT Actual/Prior` | sales without VAT |
| `Index Gross Profit Actual/Prior` | gross profit |
| `Index Gross Margin Actual/Prior` | gross margin |

An index is `IndexDif`: (this year − last year) ÷ last year, with an arrow, ▲ at or above 0 and
▼ below. Its colour is generated from an annotation: a rise is good — see
[dax-quality.md](dax-quality.md#colour-measures).

## The two pages

| Page | Rows of the table | Its tooltip |
|---|---|---|
| `Brand` | brand › product | `Brand Tooltip`: what is behind a brand's or product's margin |
| `Channel` | channel › region › city | `Channel Tooltip`: what is behind a channel's growth |

Both pages carry the same four charts above the table — gross profit, gross margin, the extra
discount, and the margin by week — so a slicer pick reads the same way on either.

## The brand tooltip

For the brand or product under the pointer:

| Figure | Measure |
|---|---|
| net sales, gross margin, gross profit | `Marketing Product Sales`, `Marketing Product Gross Margin`, `Marketing Product Gross Profit` |
| the trade budget: spent, its share of sales, accumulated, the difference, spent of accumulated | `Budget Spent | Tooltip`, `Marketing Budget Spent Rate`, `Budget Accumulated | Tooltip`, `Budget Difference | Tooltip`, `Marketing Budget Spent Of Accumulated` |
| extra discount and its rate | `Extra Discount | Tooltip`, `Extra Discount Rate | Tooltip` |
| why sales moved: volume, price, the verdict | `Marketing Quantity Effect`, `Marketing Price Effect`, `Marketing Verdict` |

### Price × volume

```
Sales  = Quantity × Price
Change = P0 × (Q1 − Q0)  +  Q1 × (P1 − P0)
         Marketing Quantity Effect   Marketing Price Effect
```

- **Price is net sales over quantity**, this year to date and last year to the same day.
- **The two effects add up to the change exactly.** Each factor is held at the opposite period
  from the one that moves; held at the same one, a remainder would belong to neither.
- **`Marketing Verdict`** names the pattern by the signs of the two effects: growth from price
  and volume, growth from price only, higher volume at a lower price, or a decline in both. A
  zero effect takes the other's sign; both zero is no change, and the verdict is blank.

## The trade budget

The brand tooltip shows whether the marketing budget the price sets aside covers what is spent.

```
Budget Accumulated = Marketing Accumulated + Credit Notes Received + Service Invoices
Budget Spent       = Trade Terms Spent + Extra Discount Without VAT + Credit Notes Issued
Trade Terms Spent  = Campaigns + Promotions + Seasonal Fixed Positions
Budget Difference  = Budget Accumulated − Budget Spent
```

- **`Marketing Accumulated`** is what the price leaves per unit after cost and four brand rates —
  distribution cost, distribution profit, annual bonus, secondary position — times the quantity
  sold. The rates come from the source, in `DimBrand`. A negative value is a real result: the
  price does not cover cost and the rates.
- **`Credit Notes Received`** and **`Service Invoices`** are what suppliers give back, booked on
  a brand; **`Credit Notes Issued`** are credit notes to customers. Consideration payable to a
  customer reduces revenue unless it pays for a distinct good or service
  [[ifrs-15]](sources.md#ifrs-15); the report shows sales before them, and the credit notes as
  spend.
- **The trade terms** come from one register, read once by the shared expression `TradeTerms`:
  one row per term, with its group (Bonus, Position, Campaign, Promotion), period and value.
  `Campaigns` and `Promotions` count the amount agreed, dated at the start;
  `Seasonal Fixed Positions` counts the fixed placements that do not run a whole year, in each
  month they have run up to today.
- **Everything here is without VAT**, so Accumulated and Spent add up on one basis.
- **Margin and budget are kept apart.** No margin measure takes budget or spend; the tooltip
  shows them side by side, never one inside the other.

## The channel tooltip

For the channel, region or city under the pointer, what is behind its growth:

| Figure | Measure |
|---|---|
| products bought, kept, new, dropped | `Customer Products Count`, `Customer Products Kept`, `Customer Products New`, `Customer Products Delisted` |
| outlets reached, kept, new, lost | `Brand Numeric Distribution`, `Brand Outlets Kept`, `Brand Outlets New`, `Brand Outlets Lost` |
| why sales moved: reach, sales per outlet, the verdict | `Brand Distribution Effect`, `Brand Sales per Outlet Effect`, `Brand Verdict` |

### Reach × sales per outlet

```
Sales  = Outlets × Sales per outlet
Change = R0 × (N1 − N0)  +  N1 × (R1 − R0)
         Brand Distribution Effect   Brand Sales per Outlet Effect
```

- **Outlets** are the outlets with a sale (`Numeric Distribution`), this year to date and last
  year to the same day; **sales per outlet** is net sales over them.
- The two effects add up to the change exactly, as price and volume do.
- **`Brand Verdict`** names the pattern: healthy growth (both up), growth from distribution
  (more outlets, less each), lower coverage with more per outlet, or a decline in both.

### Kept, new and lost

`ChurnKept`, `ChurnNew` and `ChurnLost` compare the products (or outlets) with a sale this year
to date with those with a sale over the same days a year earlier:

- **kept** — in both windows; **new** — now, not then; **lost** or **dropped** — then, not now.
  Kept + new is this year's count, kept + lost last year's.
- **The windows are aligned.** The current window follows the date slicers within this year; the
  prior one is the same window a year back (`YearToDaySet` over `PrevYearSet`), never the whole
  prior year, so nine months are not compared against twelve.
- **The set passed in is a column of the fact**, `VALUES( FactSalesLine[ProductId] )` or
  `VALUES( FactSalesLine[OutletId] )`. A column on the one side of a relationship is not filtered
  by the fact [[relationships]](sources.md#relationships), so it would return every product or
  outlet in the catalogue.
- **Counted by key with no minimum value**: a single small sale counts as much as a regular one.

## What the slicers reach

| Follows a channel, region or outlet pick | Shows the company figure under such a pick |
|---|---|
| everything on sales — profit, margin, discount, the effects, kept / new / lost — and every trade term | `Credit Notes Issued`, `Credit Notes Received`, `Service Invoices` |

The credit notes and the service invoices carry a customer or supplier and a brand, but no
outlet. A customer pick reaches them; a channel, region or outlet pick does not. So under such a
pick the tooltip's budget figures add an outlet group's figures to company figures.

## Colours

Only figures with a clear direction are coloured: the three indexes, the budget difference, the
four effects and the two verdicts. A verdict takes the same test as its wording — both effects up
is good, both down is bad, a mixed one is neutral — so colour and text never disagree. Margin
itself is left uncoloured: whether a margin is good depends on the brand and the channel, a
business call rather than a number.
