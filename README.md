# Dubai Real Estate Analysis and Property Recommender

An end-to-end data analytics project built in **Power BI** to explore Dubai's property-for-sale market and recommend undervalued listings.

## Project Goal

Help buyers and investors answer three questions:
1. What does the Dubai for-sale market look like (areas, types, prices)?
2. Which listings are priced below similar properties?
3. Where do ready and off-plan prices, supply and yields differ?

## Dataset

- **Source:** Bayut for-sale listings for Dubai
- **Size:** about 41,000 listings (41,381 raw rows)
- **Key columns:** price, type, beds, baths, address, furnishing, completion status, building name, area name, city, year of completion, total floors, parking spaces, building area (sqft), average rent, latitude, longitude

## Tools

| Tool | Use |
|---|---|
| Power Query | Data cleaning and transformation |
| Power BI (DAX) | Measures, dashboard, drill-through |

## Process

### 1. Data inspection
- Checked row counts, data types and missing values in Power Query.
- Found that missing values were stored as `0` (for example `year_of_completion`), not as nulls, which produced wrong building ages until they were converted to null.
- Found that `total_building_area_sqft` appears to describe the whole building, not the unit, so **price per sqft was not used** as the main metric.

### 2. Data cleaning (Power Query)
- Kept all valid listings (price greater than 0, area name present) so the full dataset of about 41K rows is analyzed
- Trimmed text and standardized property type names
- Labeled `beds = 0` as Studio
- Converted zero values in `year_of_completion` to null
- Built `Building Age` for completed buildings only (off-plan buildings have no age)
- Fixed a row-loss problem in Applied Steps (a filter was cutting the data from 41,382 to 167 rows)

### 3. Measures (DAX)

```DAX
Listings = COUNTROWS('bayut_selling_properties')
Avg Price = AVERAGE('bayut_selling_properties'[price])
Median Price = MEDIAN('bayut_selling_properties'[price])

Peer Median =
CALCULATE(MEDIAN('bayut_selling_properties'[price]),
  ALLEXCEPT('bayut_selling_properties',
    'bayut_selling_properties'[area_name],
    'bayut_selling_properties'[type],
    'bayut_selling_properties'[beds]))

Peer Listings =
CALCULATE(COUNTROWS('bayut_selling_properties'),
  ALLEXCEPT('bayut_selling_properties',
    'bayut_selling_properties'[area_name],
    'bayut_selling_properties'[type],
    'bayut_selling_properties'[beds]))

Value Score % =
IF([Peer Listings] >= 10,
   DIVIDE(AVERAGE('bayut_selling_properties'[price]) - [Peer Median], [Peer Median]))
```

**Value Score %** compares a listing's price with the median price of properties in the same area, type and bedroom count. A negative score means the listing is cheaper than its peers. Groups with fewer than 10 listings are excluded.

## Dashboard Pages

| Page | What it shows |
|---|---|
| Market Overview | Listings, average and median price, top 10 areas, type mix, price by bedrooms |
| Map | Bubble map of listings by area (size = listings, color = average price), with an area summary |
| Recommender | Slicers (budget, beds, type, area, status, furnishing) and a table ranked by Value Score % |
| Investor Insights | Ready vs off-plan prices by area, top yield areas, area by beds matrix |
| Property Detail | Drill-through page with building details and all listings in that building |

*Add screenshots here:*

```
[Market Overview](#)(screenshots/market_overview.png)
![Map](screenshots/map.png)
![Recommender](screenshots/recommender.png)
![Investor Insights](screenshots/investor_insights.png)
![Property Detail](screenshots/property_detail.png)
```

## Key Findings

- The dataset holds about **41K listings**, with an **average price of AED 3.79M** but a **median of AED 2.00M**. A small number of luxury listings pull the average well above the median, so the median is a better guide to a typical price.
- **Apartments make up about 70%** of all listings.
- The most listed areas are **Jumeirah Village Circle (2,477)**, **Business Bay (2,423)**, **Dubai Marina (2,112)**, **Downtown Dubai (2,064)** and **Al Reem Island (1,658)**.
- Average price rises steeply with bedroom count, but groups of 7+ bedrooms contain few listings, so those averages are unreliable.
- The Recommender surfaces listings priced roughly 5% to 25% below their peer median in the same area, type and bedroom count.

*Replace or extend these with numbers from your final filtered dashboard.*

## Limitations

- **Building area is building-level**, so unit price per sqft could not be calculated.
- **Hotel apartments** (hotel-managed buildings) are priced differently from standard units and can distort Value Score, so they should be filtered out of recommendations.
- **Value Score compares price only.** It does not account for view, floor, condition or exact location within an area.
- **Rental yield** depends on `average_rent`. Confirm whether it is annual or monthly, and use only areas with enough listings, because small samples produce unrealistic yields.
- Listings are asking prices, not final sale prices.

## Possible Next Steps

- Add a Similar Properties page using latitude and longitude distance
- Repeat the cleaning and analysis in SQL (MySQL or SQL Server) and compare results
- Build a price prediction model in Python and compare predicted and listed prices
- Track listings over time with `post_date` to measure price trends

## Repository Structure

```
.
├── README.md
├── Dubai_Real_Estate.pbix
└── screenshots/
```

## Author

**Your Name**
[LinkedIn](#) | [GitHub](#)
