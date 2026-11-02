# Design decisions

[← README](../README.md)

The rules a future maintainer would otherwise re-litigate, each with the reason it holds.

- **Everything is named in English, after the source views.** Tables, columns, measures and
  functions read the same words as the `ti` views ([data-source.md](data-source.md)), so a figure
  traces from a visual to the SQL under one name. Mapping a source system to these names happens
  once, in the views, not in the model.
- **Classifications and rates are columns of the source, not lists in the queries.** Sales
  document types, goods, term groups, the supplier-to-brand booking and the four brand rates:
  each is a column of a view. A code list or a rate inside a query goes stale silently.
- **Margin is without VAT on both sides.** The cost is posted without VAT, so the sales side
  takes the VAT out line by line; a margin over sales with VAT would not reconcile
  ([gross-margin.md](gross-margin.md#gross-profit-and-gross-margin)).
- **The plan margin has the actual margin's scope.** Goods only, without VAT, sales-out
  documents: the gap between the two is exactly the discounts given
  ([gross-margin.md](gross-margin.md#the-plan-margin-at-the-regular-price)).
- **Margin and budget are kept apart.** No margin measure takes budget or spend. A margin after
  trade spend mixes the margin with marketing spend, which are read separately; the brand tooltip
  shows them side by side ([gross-margin.md](gross-margin.md#the-trade-budget)).
- **A change is split, not described.** Price × volume and reach × sales per outlet are exact
  identities: each pair of effects adds up to the change with no remainder, because each factor
  is held at the opposite period ([gross-margin.md](gross-margin.md#price--volume)).
- **Kept, new and lost compare the same days.** The prior window is the current one a year back,
  never the whole prior year, so a part year is not set against a full one
  ([gross-margin.md](gross-margin.md#kept-new-and-lost)).
- **The source holds one row per trade term.** Cleaning the register belongs in the source view,
  where the data is entered; the model reads each term once, as it is
  ([data-source.md](data-source.md#source-views)).
- **Credit notes and service invoices stay unsplit by outlet.** They carry a customer or supplier
  and a brand, never an outlet; spreading them over outlets would invent a split the source does
  not hold ([gross-margin.md](gross-margin.md#what-the-slicers-reach)).
- **A tooltip reads its own views of the roots** (`Lookup\…`), so its captions and units change
  without touching a measure the pages use ([measures.md](measures.md#tooltip-measures)).
- **Only figures with a direction are coloured.** Whether a margin is good depends on the brand
  and the channel, so margin itself is not coloured ([gross-margin.md](gross-margin.md#colours)).
- **Per-unit figures are quantity-weighted**, value over quantity, so unit × quantity gives back
  the money ([measures.md](measures.md#cost-and-prices)).
- **Sort keys come from the source, not DAX**, in the same view as the name they sort, so the
  name and its key arrive together on every refresh ([report.md](report.md#sort-order)).
- **No section navigation.** Two pages need no menu: the tab row on each page is the whole
  navigation, so there is no menu page, drawer or bookmark to keep in step.
- **One language, kept in `Labels`.** The report is English only, yet every UI text stays a
  `Labels` row: titles never sit hard-coded on the canvas, and a language is added as data
  ([report.md](report.md#labels)).
- **The calendar restates the fact window instead of reading it.** A calendar derived from a
  fact query depends on that query, and goes short with no error when the query returns nothing.
