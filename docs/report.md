# The report

[← README](../README.md)

**Two pages and two tooltips**, in English. The report opens on `Brand`.

| Page | Asks | Main visuals |
|---|---|---|
| `Brand` | how much does each brand and product earn? | gross profit, gross margin and extra discount, this year against last year and the regular price; the margin by week; the profit table by brand and product |
| `Channel` | how much does each channel earn? | the same four charts; the profit table by channel, region and city |
| `Brand Tooltip` | what is behind a brand's or product's margin? | one table: sales, margin, the trade budget, extra discount, price and volume effects |
| `Channel Tooltip` | what is behind a channel's growth? | one table: products and outlets kept, new and lost, reach and sales-per-outlet effects |

What each figure means is in [gross-margin.md](gross-margin.md).

## Page navigation

Each page carries a **tab row** at the top right: Brand · Channel, each reading its text from a
label measure (`PageNav1 | Title`, `PageNav2 | Title`). The tab of the page you are on is filled.
There is no menu page and no drawer; the two tabs are the whole navigation. Every page also has a
phone layout — see [mobile.md](mobile.md).

The two tooltips are report-page tooltips, filtered by the data point under the pointer
[[report-tooltips]](sources.md#report-tooltips), and hidden from the tab row.

| Tooltip | Opens from | Filtered by |
|---|---|---|
| `Brand Tooltip` | the profit table on `Brand` | `DimBrand[BrandName]`, `DimProduct[ProductName]` |
| `Channel Tooltip` | the profit table on `Channel` | `DimOutlet[Channel]`, `DimOutlet[Region]`, `DimOutlet[City]` |

## Slicers

Every page has **five slicers in a row** (x = 725, 965, 1205, 1445, 1685), each under a title
button that reads a `… | SlicerTitle` label measure. A field sits in the same slot on both pages,
and every slicer is synced across them.

| Slot | Slicer | Fields | Sync group |
|---|---|---|---|
| 1 | Date Range | `DimDate[Date]` | `Date` |
| 2 | Division / Product | `DivisionName` › `BrandName` › `ProductName` | `Product` |
| 3 | Channel / Region | `Channel` › `Region` › `City` | `Channel` |
| 4 | Customer / Outlet | `PartnerName` › `OutletName` | `Customer` |
| 5 | Unit / Warehouse | `DimOrgUnit[OrgUnitName]` | `OrgUnit` |

- **The date slicer saves no dates.** It opens on the whole calendar, so it never goes stale at
  New Year; the measures cut at today themselves.
- **The product and unit slicers list only items with sales** in the selected dates, through a
  hidden filter on `Slicer Has Data`.
- **What a channel or outlet pick reaches** is in
  [gross-margin.md](gross-margin.md#what-the-slicers-reach).

## Labels

**No text is typed into a visual.** Page titles and subtitles, tab labels, visual titles and
subtitles, slicer titles, the captions under values and every alt text are label measures in
`LabelMeasures`, each a call to `LabelText( LabelType, OwnerType, OwnerNumber )`, which reads one
row of `Labels`.

- **Owner numbers are one sequence**: pages 1–2, then the slicers, then the visuals. A number is
  never shared across owner types.
- **A missing row shows ⚠**, never a quiet fallback, so a gap is seen in review.
- **One language, English.** `User Culture` returns `en-US`, and `Labels` holds one row per
  label. To add a language: add its rows to `Labels`, and make `User Culture` return the viewer's
  culture.
- **Alt texts** are `Alt … | VisualSubtitle` label measures: the visual's kind, then what it
  shows.
- **Field names** come from captions (`cultures/en-US.tmdl`): a field shows its caption in a
  visual, never a name typed into it.

## Number formats

- **`NumberFormat`** on the values, **`IndexFormat`** on the indexes — see
  [measures.md](measures.md#formats). A dynamic format string keeps the measure numeric
  [[dynamic-format-strings]](sources.md#dynamic-format-strings).
- **Colours** are theme sentiment names — good, neutral, bad — from the colour measures, so they
  follow the theme. The theme is `cortnes-light.json`.

## Sort order

Four names sort by a hidden column that comes from the source:

| Column | Sorted by |
|---|---|
| `DimProduct[ProductName]` | `ProductSortOrder` |
| `DimPartner[PartnerName]` | `PartnerSortOrder` |
| `DimOutlet[OutletName]` | `OutletSortOrder` |
| `DimOrgUnit[OrgUnitName]` | `OrgUnitSortOrder` |
