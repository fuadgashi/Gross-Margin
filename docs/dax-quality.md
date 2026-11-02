# DAX quality

[← README](../README.md)

How the model's DAX is kept to its standards: nothing unused, no duplicate, one folder per
subject, every measure described and captioned, and every change proven not to move a number.

**Everything here is measured from the files.** `scripts/dax_audit.py` reads the model and the
report: every measure and function, and every place a visual, page or filter binds a measure.
It sees this report only: it cannot see Analyze in Excel or another report built on the
model.

| Command | Does |
|---|---|
| `python scripts/dax_audit.py` | prints every count below |
| `python scripts/dax_audit.py --strict` | the same, and exits 1 if a count that must be zero is not; runs on every pull request |
| `python scripts/dax_audit.py --register` | regenerates the [measure register](#measure-register) in this document |
| `python scripts/dax_audit.py --baseline baseline.dax` | writes the regression query of [How a DAX change is tested](#how-a-dax-change-is-tested) |
| `powershell -File scripts/dax_baseline.ps1 -QueryFile baseline.dax -OutCsv before.csv` | runs it against the model open in Desktop |
| `python scripts/dax_audit.py --compare before.csv after.csv` | compares two runs, measure by measure |
| `python scripts/dax_audit.py --json out.json` | writes every name behind every number |
| `python scripts/rag_colors.py` | writes the colour measure of every index that asks for one — see [Colour measures](#colour-measures) |

## What counts as used

A measure is **used** when a visual, page or filter of the report binds it, when a used
measure references it, or when a function a used measure calls reads it (`User Culture`, read
by `LabelText`). A measure no visual binds is still **kept when the docs name it** in
backticks, or name its folder. Everything a kept measure calls is kept with it. Today every
analytical measure is bound by a visual or reached through one.

## Counts that must be zero

`--strict` fails the pull request when any of these is not zero.

| Count | What it catches |
|---|---|
| unresolved report references | a visual bound to a measure that no longer exists |
| unused measures, unused label measures, unused functions | DAX nothing reaches: removed, not kept "in case" |
| exact duplicates | two measures with the same expression, ignoring spaces and comments; a measure that differs only in the group it filters (`"Campaign"`, `"Promotion"`) is not a duplicate |
| text typed into visuals | a field renamed inside a visual instead of captioned in the model — see [report.md](report.md#labels) |
| fields sharing one caption in a visual | two columns of one table or matrix under the same heading |
| colour measures out of date | a colour measure that no longer matches its index's annotation |
| measures without a description, format or folder | the standards in [maintenance.md](maintenance.md#engineering-standards); a colour measure is text and carries the text format hint instead of a format |
| variables without the `_` prefix | a `VAR` not in English snake_case with the `_` prefix — see [maintenance.md](maintenance.md#naming) |
| colours not from `RagColor` | a theme colour name typed into a measure instead of returned by `RagColor` |
| `IFERROR` | an error hidden instead of prevented: the model divides with `DIVIDE` and tests with `ISBLANK` |
| `FILTER` over a whole table | a filter argument built on a whole table: the model's filter arguments name columns |
| sales rebuilt inline | quantity × net price summed outside `Sales Net Price` and its ex-VAT twin — see [measures.md](measures.md#sales-is-one-measure) |
| visible measures without a caption | a field that would show its model name in a visual |

The audit also prints, without failing, the **shapes** that three or more measures share: the
expression with each measure reference replaced by a placeholder.

| Shape | Measures | Why they stay named measures |
|---|---|---|
| `YearToDay( [x] )` | 18 | the year-to-date window is already one function; each measure is a figure a chart, table or tooltip shows under its own caption |
| `[x]` | 10 | the tooltips' views of a root (`Marketing Product Sales` …): each carries the tooltip's own caption and unit, while the root keeps the page's |
| `RagColor( [x], 0 )` | 8 | the colour measures `rag_colors.py` writes — see [Colour measures](#colour-measures) |
| `PrevYear( [x] )` | 7 | the same window, one year back |
| `IndexDif( [x], [y] )` | 3 | the change is already one function; each index pairs two named figures and carries its own format and colour |

## Colour measures

An index asks for a colour with one annotation on the measure:

- `annotation RagColor = UpIsGood`: a rise is good.
- `annotation RagColor = UpIsBad`: a rise is bad.

`python scripts/rag_colors.py` then writes its colour measure, hidden, in
`Formatting\RAG Color`: `RagColor( [index], centre )`, or with the sign turned for `UpIsBad`.
Edit the annotation, never the colour measure; `--check` runs on every pull request and fails
when a colour measure is out of step. A colour measure with its own rule — anything but one
`RagColor` call on one index — is written by hand and left alone by the script: today those are
`Marketing Verdict Color` and `Brand Verdict Color`, which colour each verdict by the signs of
both its effects.

**What is coloured** is in [gross-margin.md](gross-margin.md#colours).

## How a DAX change is tested

Run before the change and after it, on the same refresh.

| Test | How | Passes when |
|---|---|---|
| **Numbers unchanged** | `--baseline`, then `dax_baseline.ps1` on the model open in Desktop, before and after; `--compare` | every kept numeric measure, every year: no value appears or vanishes, and a number differs by less than one part in 10¹² |
| **No broken binding** | `python scripts/model_check.py` and `python scripts/dax_audit.py` | 0 findings; `unresolved report references: 0` |
| **The audit's counts** | `python scripts/dax_audit.py --strict` | every count that must be zero is zero |
| **Visuals render** | open the pages listed under *Used on* in the register | no visual shows an error or a blank |
| **Documents** | `python scripts/docs_check.py` | 0 findings |

**The baseline needs data.** `dax_baseline.ps1` reads the model open in Power BI Desktop, so
it runs only once the model is connected to a source and refreshed — see
[data-source.md](data-source.md#running-it-on-your-own-data). The baseline query itself is generated from the files and
needs nothing: `--baseline` writes one query per folder, every measure by `DimDate[Year]`, and
the PowerShell script runs them as one query and saves one row per year. The comparison allows
a relative difference of 10⁻¹², so two runs may differ only in their last digits; a real change
moves a figure by far more.

## Measure register

Every analytical measure in its place: its folder, whether it is visible, and the visuals
that use it. Regenerated by `python scripts/dax_audit.py --register`; edit the model, not
this table.

<!-- register:start -->
90 analytical measures.

### Calculation Roots

| Measure | Folder | Visible | Used on |
|---|---|---|---|
| Numeric Distribution | KPI | yes | inside 2 measures (Numeric Distribution Actual Year, …) |
| Products Count | KPI | yes | inside 1 measure (Products Count Actual Year) |
| Gross Margin | Profit | yes | inside 2 measures (Gross Margin Actual Year, …) |
| Gross Profit | Profit | yes | inside 3 measures (Gross Margin, …) |
| Gross Profit at Regular Price | Profit | yes | inside 1 measure (Gross Profit at Regular Price Actual Year) |
| Sales Cost Price | Sales | yes | inside 1 measure (Gross Profit) |
| Sales Net Price | Sales | yes | inside 3 measures (Sales Net Price Actual Year, …) |
| Sales Net Price ex VAT | Sales | yes | inside 4 measures (Gross Margin, …) |
| Sales Quantity | Sales | yes | inside 2 measures (Sales Quantity Actual Year, …) |
| Sales Selling Price | Sales | yes | inside 1 measure (Sales Selling Price Actual Year) |
| Sales Selling Price ex VAT | Sales | yes | inside 1 measure (Gross Margin at Regular Price Actual Year) |

### Formatting

| Measure | Folder | Visible | Used on |
|---|---|---|---|
| Brand Distribution Effect Color | RAG Color | no | Channel Tooltip matrix |
| Brand Sales per Outlet Effect Color | RAG Color | no | Channel Tooltip matrix |
| Brand Verdict Color | RAG Color | no | Channel Tooltip matrix |
| Budget Difference Color \| Tooltip | RAG Color | no | Brand Tooltip matrix |
| Index Gross Margin Actual/Prior Color | RAG Color | no | Brand matrix, Channel matrix |
| Index Gross Profit Actual/Prior Color | RAG Color | no | Brand matrix, Channel matrix |
| Index Sales Net Price ex VAT Actual/Prior Color | RAG Color | no | Brand matrix, Channel matrix |
| Marketing Price Effect Color | RAG Color | no | Brand Tooltip matrix |
| Marketing Quantity Effect Color | RAG Color | no | Brand Tooltip matrix |
| Marketing Verdict Color | RAG Color | no | Brand Tooltip matrix |
| Slicer Has Data | Slicer Filter | no | Brand slicer, Channel slicer |

### KPI

| Measure | Folder | Visible | Used on |
|---|---|---|---|
| Numeric Distribution Actual Year | Numeric Distribution | yes | inside 3 measures (Brand Distribution Effect, …) |
| Numeric Distribution Prior Year | Numeric Distribution | yes | inside 1 measure (Numeric Distribution Prior Year YTD) |
| Numeric Distribution Prior Year YTD | Numeric Distribution | yes | inside 2 measures (Brand Distribution Effect, …) |
| Products Count Actual Year | Products Count | yes | inside 1 measure (Customer Products Count) |

### Lookup

| Measure | Folder | Visible | Used on |
|---|---|---|---|
| Brand Distribution Effect | Brand | yes | Channel Tooltip matrix |
| Brand Numeric Distribution | Brand | yes | Channel Tooltip matrix |
| Brand Outlets Kept | Brand | yes | Channel Tooltip matrix |
| Brand Outlets Lost | Brand | yes | Channel Tooltip matrix |
| Brand Outlets New | Brand | yes | Channel Tooltip matrix |
| Brand Sales per Outlet Effect | Brand | yes | Channel Tooltip matrix |
| Brand Supplier Name | Brand | yes | Brand Tooltip matrix |
| Brand Verdict | Brand | yes | Channel Tooltip matrix |
| Customer Products Count | Customer | yes | Channel Tooltip matrix |
| Customer Products Delisted | Customer | yes | Channel Tooltip matrix |
| Customer Products Kept | Customer | yes | Channel Tooltip matrix |
| Customer Products New | Customer | yes | Channel Tooltip matrix |
| Budget Accumulated \| Tooltip | Marketing | yes | Brand Tooltip matrix |
| Budget Difference \| Tooltip | Marketing | yes | Brand Tooltip matrix |
| Budget Spent \| Tooltip | Marketing | yes | Brand Tooltip matrix |
| Extra Discount Rate \| Tooltip | Marketing | yes | Brand Tooltip matrix |
| Extra Discount \| Tooltip | Marketing | yes | Brand Tooltip matrix |
| Marketing Budget Spent Of Accumulated | Marketing | yes | Brand Tooltip matrix |
| Marketing Budget Spent Rate | Marketing | yes | Brand Tooltip matrix |
| Marketing Price Effect | Marketing | yes | Brand Tooltip matrix |
| Marketing Product Gross Margin | Marketing | yes | Brand Tooltip matrix |
| Marketing Product Gross Profit | Marketing | yes | Brand Tooltip matrix |
| Marketing Product Sales | Marketing | yes | Brand Tooltip matrix |
| Marketing Quantity Effect | Marketing | yes | Brand Tooltip matrix |
| Marketing Verdict | Marketing | yes | Brand Tooltip matrix |

### Margin

| Measure | Folder | Visible | Used on |
|---|---|---|---|
| Gross Margin at Regular Price Actual Year | Plan | yes | Brand areaChart, Brand combo chart, Channel areaChart +1 |
| Gross Profit at Regular Price Actual Year | Plan | yes | Brand combo chart, Channel combo chart |
| Gross Profit Actual Year | Profit | yes | Brand combo chart, Brand matrix, Channel matrix +1 |
| Gross Profit Prior Year | Profit | yes | Brand combo chart, Brand matrix, Channel matrix +1 |
| Index Gross Profit Actual/Prior | Profit | yes | Brand matrix, Channel matrix |
| Gross Margin Actual Year | Ratios | yes | Brand areaChart, Brand combo chart, Brand matrix +3 |
| Gross Margin Prior Year | Ratios | yes | Brand areaChart, Brand combo chart, Brand matrix +3 |
| Index Gross Margin Actual/Prior | Ratios | yes | Brand matrix, Channel matrix |

### Sales

| Measure | Folder | Visible | Used on |
|---|---|---|---|
| Sales Net Price Actual Year | Net Price | yes | inside 3 measures (Brand Sales per Outlet Effect, …) |
| Sales Net Price Prior Year | Net Price | yes | inside 1 measure (Sales Net Price Prior Year YTD) |
| Sales Net Price Prior Year YTD | Net Price | yes | inside 4 measures (Brand Distribution Effect, …) |
| Index Sales Net Price ex VAT Actual/Prior | Net Price ex VAT | yes | Brand matrix, Channel matrix |
| Sales Net Price ex VAT Actual Year | Net Price ex VAT | yes | Brand matrix, Channel matrix |
| Sales Net Price ex VAT Prior Year | Net Price ex VAT | yes | Brand matrix, Channel matrix |
| Sales Quantity Actual Year | Quantity | yes | inside 2 measures (Marketing Price Effect, …) |
| Sales Quantity Prior Year | Quantity | yes | inside 1 measure (Sales Quantity Prior Year YTD) |
| Sales Quantity Prior Year YTD | Quantity | yes | inside 2 measures (Marketing Price Effect, …) |
| Sales Selling Price Actual Year | Selling Price | yes | inside 1 measure (Extra Discount Rate Actual Year) |

### Trade Marketing

| Measure | Folder | Visible | Used on |
|---|---|---|---|
| Credit Notes Issued Actual Year | Credit Notes Issued | yes | inside 1 measure (Budget Spent Actual Year) |
| Credit Notes Received Actual Year | Credit Notes Received | yes | inside 1 measure (Budget Accumulated Actual Year) |
| Extra Discount Actual Year | Discounts | yes | Brand combo chart, Channel combo chart |
| Extra Discount Prior Year | Discounts | yes | Brand combo chart, Channel combo chart |
| Extra Discount Rate Actual Year | Discounts | yes | inside 1 measure (Extra Discount Rate \| Tooltip) |
| Budget Accumulated Actual Year | Trade Spend | yes | inside 3 measures (Budget Accumulated \| Tooltip, …) |
| Budget Difference Actual Year | Trade Spend | yes | inside 1 measure (Budget Difference \| Tooltip) |
| Budget Spent Actual Year | Trade Spend | yes | inside 4 measures (Budget Difference Actual Year, …) |
| Campaigns | Trade Spend | yes | inside 1 measure (Trade Terms Spent) |
| Credit Notes Issued | Trade Spend | yes | inside 1 measure (Credit Notes Issued Actual Year) |
| Credit Notes Received | Trade Spend | yes | inside 1 measure (Credit Notes Received Actual Year) |
| Extra Discount | Trade Spend | yes | inside 2 measures (Extra Discount Actual Year, …) |
| Extra Discount Without VAT | Trade Spend | yes | inside 1 measure (Budget Spent Actual Year) |
| Marketing Accumulated | Trade Spend | yes | inside 1 measure (Marketing Accumulated Actual Year) |
| Marketing Accumulated Actual Year | Trade Spend | yes | Brand combo chart, Channel combo chart |
| Promotions | Trade Spend | yes | inside 1 measure (Trade Terms Spent) |
| Seasonal Fixed Positions | Trade Spend | yes | inside 1 measure (Trade Terms Spent) |
| Service Invoices | Trade Spend | yes | inside 1 measure (Service Invoices Actual Year) |
| Service Invoices Actual Year | Trade Spend | yes | inside 1 measure (Budget Accumulated Actual Year) |
| Trade Terms Spent | Trade Spend | yes | inside 1 measure (Trade Terms Spent Actual Year) |
| Trade Terms Spent Actual Year | Trade Spend | yes | inside 1 measure (Budget Spent Actual Year) |
<!-- register:end -->
