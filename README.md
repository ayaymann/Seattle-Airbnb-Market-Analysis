# 🏡 Seattle Airbnb Market Analytics Dashboard

![Power BI](https://img.shields.io/badge/Power_BI-F2C811?style=for-the-badge&logo=powerbi&logoColor=black)
![SQL Server](https://img.shields.io/badge/SQL_Server-CC292B?style=for-the-badge&logo=microsoftsqlserver&logoColor=white)
![Data Analysis](https://img.shields.io/badge/Data_Analytics-0078D4?style=for-the-badge)

## 📌 Executive Summary
This project delivers an end-to-end market analysis of Seattle Airbnb listings, pricing trends, and guest reviews. By extracting business signals from raw listing data using **SQL Server** and constructing an interactive analytical dashboard in **Power BI**, this project provides data-driven insights to optimize host strategies, pricing models, and property positioning.

---

## 💡 Key Business Insights
* **Price vs. Demand & Seasonality:** Nightly prices peak significantly during summer months (July–August at ~$150/night) compared to winter lows (~$122/night in January).
* **Listing Breakdown:** Entire homes/apartments account for **66.55%** of listings, private rooms make up **30.38%**, and shared rooms comprise **3.06%**.
* **Price & Quality Myth:** There is virtually **no correlation (r = 0.12)** between nightly price and review ratings—higher prices do not guarantee better guest experiences.
* **Trust & Volume:** A strong positive correlation **(r = 0.68)** exists between review count and overall rating, showing that established experience builds guest trust.
* **Geographic Premium:** Properties in central areas like Downtown command a significant price premium (~$164/night avg) compared to outer neighborhoods (~$95/night avg).

---

## 🛠️ Data Architecture & ETL Pipeline

### 1. Database Schema & SQL Queries
The relational database (`AirbnbDB`) consists of three core tables: `listings`, `calendar`, and `reviews`.

```sql
-- Insight 1: Top 5 Highest Rated Properties (Minimum 20 Reviews)
SELECT TOP 5
    name,
    review_scores_rating,
    number_of_reviews,
    price
FROM listings
WHERE number_of_reviews >= 20
  AND review_scores_rating IS NOT NULL
ORDER BY review_scores_rating DESC, number_of_reviews DESC;

-- Insight 2: Top 5 Neighborhoods by Average Nightly Price
SELECT TOP 5
    neighbourhood_cleansed AS neighbourhood,
    COUNT(*) AS total_properties,
    ROUND(AVG(price), 2) AS avg_price
FROM listings
WHERE price IS NOT NULL
GROUP BY neighbourhood_cleansed
HAVING COUNT(*) >= 5
ORDER BY avg_price DESC;

-- Insight 3: Property Type Distribution, Rating, and Price
SELECT TOP 5
    property_type,
    COUNT(*) AS total_units,
    ROUND(AVG(review_scores_rating), 2) AS avg_rating,
    ROUND(AVG(price), 2) AS avg_price
FROM listings
WHERE property_type IS NOT NULL
GROUP BY property_type
ORDER BY total_units DESC;

-- Insight 4: Top Hosts by Portfolio Size
SELECT TOP 5
    host_id,
    host_name,
    COUNT(*) AS total_properties_owned
FROM listings
WHERE host_name IS NOT NULL
GROUP BY host_id, host_name
ORDER BY total_properties_owned DESC;
```

---

### 2. Data Cleaning & Transformation (Power Query)
Data transformations were applied in Power Query across all three source datasets:
* **Fact Calendar:** Cleaned available values (t/f $\rightarrow$ TRUE/FALSE), formatted currency values, corrected data types, removed duplicate listing-date records, and added custom conditional availability flags.
* **Dim Listing:** Removed unused text fields/URLs, trimmed whitespace, handled missing values, created review-status custom columns, and removed duplicates.
* **Fact Reviews:** Filtered core attributes (listing_id, date, reviewer_id, comments), corrected data types, cleaned text fields, and removed duplicates.

---

### 3. Data Model (Star Schema)
The Power BI model is built as an optimized Star Schema centered around Dim Listing and Dim Date, connecting to Fact Calendar and Fact Reviews via 1-to-Many relationships.

```text
                                  +-------------------+
                                  |     Dim Date      |
                                  +-------------------+
                                            | (date)
                                            v
          +-------------------+   +-------------------+   +-------------------+
          |    Fact Reviews   |-->|    Dim Listing    |<--|   Fact Calendar   |
          +-------------------+   +-------------------+   +-------------------+
               (listing_id)               (id)                 (listing_id)
```

---

### 4. Key DAX Calculations
Core DAX measures developed for business KPI tracking:

```dax
// Date Table Generation
Dim Date = 
ADDCOLUMNS (
    CALENDAR ( MIN ( 'Fact Reviews'[date] ), MAX ( 'Fact Calendar'[date] ) ),
    "Year", YEAR ( [Date] ),
    "Month Number", MONTH ( [Date] ),
    "Month", FORMAT ( [Date], "MMMM" ),
    "Quarter", "Q" & FORMAT ( [Date], "Q" ),
    "Year-Month", FORMAT ( [Date], "YYYY-MM" )
)

// Key Business Metrics
Total Listings = DISTINCTCOUNT ( 'Dim Listing'[id] )

Total Reviews = COUNTROWS ( 'Fact Reviews' )

Available Days = CALCULATE ( COUNTROWS ( 'Fact Calendar' ), 'Fact Calendar'[available] = TRUE () )

Unavailable Days = CALCULATE ( COUNTROWS ( 'Fact Calendar' ), 'Fact Calendar'[available] = FALSE () )

Availability Rate = DIVIDE ( [Available Days], [Available Days] + [Unavailable Days] )

Average Available Nightly Price = CALCULATE ( AVERAGE ( 'Fact Calendar'[price] ), 'Fact Calendar'[available] = TRUE () )

Average Rating = AVERAGE ( 'Dim Listing'[review_scores_rating] )

Superhost Rate = DIVIDE ( CALCULATE ( [Total Listings], 'Dim Listing'[host_is_superhost] = TRUE () ), [Total Listings] )
```

---

### 📊 Dashboard Overview 
The interactive report includes four primary analytical views:

1. Overview Page: Macro KPIs ($127.7 Avg Price, 85K Reviews, 4K Total Listings), room type distribution, and monthly price trends.
2. Location & Property Analysis: Geographic price vs. demand distributions, top 10 most expensive neighborhoods, and property type volume.
3. Pricing & Reviews Page: Interactive price vs. rating scatter plots, minimum stay impact, and price variations across room categories.
4. Insights Page: High-level strategic conclusions and correlation breakdowns for executive stakeholders.

---

### 📂 Repository Structure
```
├── Data/
│   ├── Seattle_Airbnb_Database.sql          # Database SQL script
├── Dashboard/
│   ├── Seattle_Airbnb_Market_Analysis.pbix  # Power BI report file
│   └── Screenshots/                         # High-res view of dashboard pages
├── Documentation/
│   └── Airbnb_Data_Divas_Presentation.pptx  # Project presentation deck
└── README.md
```
---

### 👥 Project Team — *Data Divas*
* Aya Ayman

* Banan Mag

* Habiba Wa

* Mennatullah Hu

* Mirna Elgho

