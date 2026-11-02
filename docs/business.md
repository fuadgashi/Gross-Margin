# The business

[← README](../README.md)

A distributor buys fast-moving consumer goods from brand owners and sells them to shops,
restaurants and other customers. What it earns is the gap between what it sells for and what the
goods cost: **gross profit**, and as a share of sales, **gross margin**. The model answers three
questions:

1. **How much does each brand and product earn, and is it more or less than last year?** Sales,
   gross profit and gross margin, this year to date against last year — see
   [gross-margin.md](gross-margin.md#gross-profit-and-gross-margin).
2. **How far is the margin from the one the price was built for?** The margin at the regular
   price, before discounts, beside the margin made — see
   [gross-margin.md](gross-margin.md#the-plan-margin-at-the-regular-price).
3. **Why did sales and profit move?** For a brand: price against volume, and whether the trade
   budget covers its spend. For a channel: more or fewer outlets against more or less per outlet,
   and which products and outlets were kept, gained or lost — see
   [gross-margin.md](gross-margin.md#the-brand-tooltip) and
   [gross-margin.md](gross-margin.md#the-channel-tooltip).

## Customers, outlets and channels

- A **customer** is the business partner that is invoiced; an **outlet** is one of its points
  of sale. Sales are recorded per outlet.
- Every outlet has a **channel**, a **region** and a **city**, kept in the source; the `Channel`
  page groups by them.

## Brands, divisions and products

- A **product** is one catalogue item, counted in its base unit.
- Its **brand** carries the four rates the price sets aside; brands are grouped in commercial
  **divisions**.
- **Goods only.** Service and fixed-asset lines never reach a sales or cost figure, so cost and
  price sit on one base.

## Prices

| Price | Means |
|---|---|
| **list price** | the regular selling price, before any discount |
| **gross price** | after the regular discount |
| **net price** | after the extra discount too: what the customer pays |
| **unit cost** | the average cost the source posts to the cost of sales, without VAT |

Gross profit is made on the net price; the plan profit on the list price. The gap between them is
the discounts given.
