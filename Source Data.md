# Source Data
Ideal source data is a multi-million row table containing 3+ years history of weekly product store level sales and volume data.  But compromises from the ideal are possible, providing you understand the consequences.  In collecting source data, it is also vital to understand exact definitions of each field.  

Below are considerations for each field in turn.  Conforming to the ideal is less important that understanding exactly what is avaliable, and what the consequences are for the model.

## Week
* Minimum of 2 full years history (in order to give 1 year of year-on-year data).  Ideally 3 or more years.
* Ensure weeks are consistently 7 days long.  Sounds obvious, but some retailers can define weeks with variable lengths to fit within calendar months.
* Ensure there is a means to match each week to the equivalent week last year.

## Product
Every retailer has a different nominaculture for product groupings: Sections, Categories, Departments, Bays, Product Families, Etc.  The most flexible data source involves data at the product level, with a second reference table defining various product groupings.  Product level data will enable the most detailed data cleaning and outlier removal.  Elasticity is best measured after aggregation to some level above product.  So if aggregate product data is all that is available, it is possible to proceed with that.

The  definition of a product: *Something a customer would see as materially the thing*.  This may differ from an SKU (Stock Keeping Unit) or Barcode/EAN.  But pragmatism needs to be applied if SKU or EAN is the data that is easily available.

## Stores
Similar to products, if store level data is available, it is best to take it as source data.  This enables more detailed data cleaning and outlier removal.  For example, removing stores that have not traded consistently across the period of analysis.  If store level data is not available, request as clean a set of stores as possible: Like-for-like stores excluding new store opennings, closures, refits, extensions etc.

## ~~Price~~ Total Sales
Price should reflect what *the average shopper experienced*.  The full retail selling price (shelf-edge price) may not be sufficient as it will not include promotional pricing, multibuys, markdowns, etc.

You could collect data on shelf-edge price plus every form of discount, then calculate some suitably weighted average price.  But there is a simpler approach; use total sales and divide by total volume.  Hence the source data for a price elasticity model needs sales, not price.


## Volume
The number of units sold.  "Units" is straight-forward for items sold as units (tins, packets, boxes...).  For items sold by weight or quantity, ensure the unit is the same as used to define the price.  I.e. if apples are priced and sold per KG, ensure the price is per-KG and the volume is in terms of KG (not a count of purchases).

---

© Versible 2022