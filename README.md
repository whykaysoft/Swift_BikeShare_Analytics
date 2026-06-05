# 🚲 Swift_BikeShare_Analytics

> **An end-to-end data analysis project** from raw database creation in SQL to an interactive Power BI dashboard, answering a real business brief: *"Should we raise prices next year, and by how much?"*

---

## Project Overview

**Swift Bike Share** needed a data analyst to build a performance dashboard and answer a pricing question using their historical ride data.

This project covers the full analyst workflow:

| Step | Task |
|------|------|
| 1 | Create a relational database |
| 2 | Write and develop SQL queries |
| 3 | Connect Power BI to the database |
| 4 | Build an interactive dashboard in Power BI |
| 5 | Answer the business analysis questions |

---

## Business Brief (Email Request)

> *The following is the original stakeholder request that initiated this project.*

---

**Subject:** Request for Development of Swift Bike Share Dashboard

Dear Data Analyst,

We need your expertise to develop a dashboard for **"Swift Bike Share"** that displays our key performance metrics for informed decision-making.

**Requirements:**

- **Hourly Revenue Analysis** — understand which hours drive the most revenue
- **Profit and Revenue Trends** — track performance over time
- **Seasonal Revenue** — identify seasonal patterns across the year
- **Rider Demographics** — breakdown of casual vs. registered riders

**Design and Aesthetics:** Use our company colours and ensure the dashboard is easy to navigate.

**Data and Source:** Access to our databases will be provided. If no database exists, please create one.

**Deadline:** We need a preliminary version ASAP.

Please provide an estimated timeline for completion and a **recommendation on raising prices next year.**

Best regards,
*Swift Bike Share Management*

---

## Workflow & Methodology

```
Raw Data
   │
   ▼
┌─────────────────────┐
│  Step 1             │
│  Create Database    │  ← Build relational tables in SQL Server / PostgreSQL
└────────┬────────────┘
         │
         ▼
┌─────────────────────┐
│  Step 2             │
│  Develop SQL        │  ← Write queries: revenue, profit, hourly/seasonal trends
│  Queries            │
└────────┬────────────┘
         │
         ▼
┌─────────────────────┐
│  Step 3             │
│  Connect Power BI   │  ← Live connection from Power BI Desktop to SQL database
│  to Database        │
└────────┬────────────┘
         │
         ▼
┌─────────────────────┐
│  Step 4             │
│  Build Dashboard    │  ← Design visuals, apply company branding, add slicers
└────────┬────────────┘
         │
         ▼
┌─────────────────────┐
│  Step 5             │
│  Answer Analysis    │  ← Derive insights and price recommendation from data
│  Questions          │
└─────────────────────┘
```

---

## Database Setup in SQL

The database was created from scratch to house Swift Bike Share's historical ride and revenue data.

### Tables Created

```sql
-- cost_table
CREATE TABLE cost_table (
    yr      INT,        -- Year identifier (0 or 1)
    price   FLOAT,      -- Selling price
    COGS    FLOAT       -- Cost of Goods Sold
);
```

```sql
-- bike_share_yr_0
CREATE TABLE bike_share_yr_0 (
    dteday      DATE,       -- Date of record
    season      INT,        -- Season (1=Winter, 2=Spring, 3=Summer, 4=Fall)
    yr          INT,        -- Year index (0 = 2021)
    mnth        INT,        -- Month (1–12)
    hr          INT,        -- Hour of day (0–23)
    holiday     INT,        -- Holiday flag (0 or 1)
    weekday     INT,        -- Day of week (0=Sunday)
    workingday  INT,        -- Working day flag (0 or 1)
    weathersit  INT,        -- Weather situation code (1–4)
    temp        FLOAT,      -- Normalized temperature
    atemp       FLOAT,      -- Normalized feeling temperature
    hum         FLOAT,      -- Normalized humidity
    windspeed   INT,        -- Normalized wind speed
    rider_type  VARCHAR(50),-- Rider category (casual / registered)
    riders      INT         -- Count of riders
);
```

```sql
-- bike_share_yr_1
CREATE TABLE bike_share_yr_1 (
    dteday      DATE,       -- Date of record
    season      INT,        -- Season (1=Winter, 2=Spring, 3=Summer, 4=Fall)
    yr          INT,        -- Year index (1 = 2022)
    mnth        INT,        -- Month (1–12)
    hr          INT,        -- Hour of day (0–23)
    holiday     INT,        -- Holiday flag (0 or 1)
    weekday     INT,        -- Day of week (0=Sunday)
    workingday  INT,        -- Working day flag (0 or 1)
    weathersit  INT,        -- Weather situation code (1–4)
    temp        FLOAT,      -- Normalized temperature
    atemp       FLOAT,      -- Normalized feeling temperature
    hum         FLOAT,      -- Normalized humidity
    windspeed   FLOAT,      -- Normalized wind speed (decimal in yr_1)
    rider_type  VARCHAR(50),-- Rider category (casual / registered)
    riders      INT         -- Count of riders
);
```

### Key SQL Queries

**1. Hourly Revenue Analysis and Seasonal Revenue**
```sql
WITH combined_data AS (
    SELECT * FROM bike_share_yr_0
    UNION ALL
    SELECT * FROM bike_share_yr_1
),
revenue_calc AS (
    SELECT cd.*, cd.riders * ct.price AS revenue,
           cd.riders * ct.price - cd.riders * ct.COGS AS profit
    FROM combined_data cd
    LEFT JOIN cost_table ct ON cd.yr = ct.yr
)
SELECT * FROM revenue_calc;
, seasonal_summary AS (
    SELECT 
        CASE season WHEN 1 THEN 'Winter' WHEN 2 THEN 'Spring'
                    WHEN 3 THEN 'Summer' WHEN 4 THEN 'Fall' END AS season_name,
        SUM(revenue) AS total_revenue,
        LAG(SUM(revenue)) OVER (ORDER BY season) AS prev_season_revenue,
        ROUND(100.0 * (SUM(revenue) - LAG(SUM(revenue)) OVER (ORDER BY season))
              / LAG(SUM(revenue)) OVER (ORDER BY season), 2) AS growth_pct
    FROM revenue_calc
    GROUP BY season
)
```

**2. Profit and Revenue Trends Over Time**
```sql
SELECT
    YEAR(ride_date)                    AS year,
    MONTH(ride_date)                   AS month,
    SUM(rides)                         AS total_rides,
    ROUND(SUM(revenue), 2)             AS total_revenue,
    ROUND(SUM(profit), 2)              AS total_profit,
    ROUND(SUM(profit) / SUM(revenue)
          * 100, 2)                    AS profit_margin_pct
FROM bike_share_yr_1
GROUP BY YEAR(ride_date), MONTH(ride_date)
ORDER BY year, month;
```

**4. Rider Demographics**
```sql
SELECT
    rider_type,
    COUNT(*)                                AS total_records,
    SUM(rides)                              AS total_rides,
    ROUND(SUM(revenue), 2)                  AS total_revenue,
    ROUND(SUM(rides) * 100.0
          / SUM(SUM(rides)) OVER (), 2)     AS ride_share_pct
FROM bike_share_yr_1
GROUP BY rider_type
ORDER BY total_rides DESC;
```

---

## Power BI Dashboard

The dashboard was built in **Power BI Desktop**, connected live to the SQL database, and designed using Swift Bike Share's company colours.

### Dashboard Views

> 📎 **See:** `Swift_BikeShare_Dashboard.pbix` in this repository  
> 📎 **See:** `Dashboard_Screenshot.png` for a quick preview

---

## Key Findings

Based on the SQL analysis and dashboard exploration:

- **Peak hours** drive the highest revenue — morning commute and evening leisure windows outperform all others
- **Registered riders** account for the majority of rides and revenue, while **casual riders** show stronger growth potential
- **Summer and Fall** are the highest-revenue seasons; Winter is the lowest
- Revenue and profit showed a **significant increase** from the prior year, suggesting strong price elasticity in the existing customer base

---

## Recommendation

### On Raising Prices for Next Year

Given the substantial revenue increase observed in the prior year, a **measured and data-backed pricing strategy** is recommended:

---

### Conservative Increase (Recommended)

Considering the strong increase already recorded, a conservative approach is prudent — raising prices too aggressively risks hitting a **price ceiling** where demand starts to drop.

**Recommended range: 10–15% increase**

| Scenario | 2022 Price | New Price |
|----------|-----------|-----------|
| 10% increase | $4.99 | **~$5.49** |
| 15% increase | $4.99 | **~$5.74** |

---

### Recommended Strategy

**1. Market Analysis First**
> Conduct further market research to understand customer satisfaction levels, potential competitive changes, and the broader economic environment. This will guide whether to lean toward the lower or higher end of the 10–15% range.

**2. Segmented Pricing**
> Consider **different pricing for casual vs. registered users** — they have different price sensitivities. Registered users (commuters) are likely less price-sensitive than casual/leisure riders.

**3. Monitor and Adjust**
> Implement the new prices at the start of the season but remain ready to adjust based on immediate customer feedback and early sales data. A/B testing a subset of markets or time periods can provide signal before a full rollout.

---

## Tools Used

| Tool | Purpose |
|------|---------|
| **SQL Server / PostgreSQL** | Database creation and querying |
| **Power BI Desktop** | Dashboard design and visualisation |
| **Microsoft Excel** | Initial data inspection and cleaning |

---
