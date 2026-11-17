# Cancellation & Guest Behavior Analysis (SQL Server + Power BI)

An end-to-end analytics project on hotel booking data: raw CSV is loaded into SQL Server, cleaned and analysed in T-SQL, then explored in a three-page Power BI dashboard.

**Project question:** Which bookings are most likely to be cancelled, where does revenue come from, and how do pricing and seasonality behave across two hotels in Portugal?

---

## Why this project

I have 8+ years of hotel reservations experience, where cancellations, lead time and deposit policy directly affect revenue. This project looks at the same questions with data: who cancels, how early they book, and which segments bring in revenue.

## Key findings

| # | Finding | Evidence |
|---|---------|----------|
| 1 | **Cancellation differs sharply by hotel.** | City Hotel 41.79% vs Resort Hotel 27.77%. |
| 2 | **Lead time matters.** | Cancelled bookings were made 144.9 days ahead on average vs 80.1 days for bookings that were kept. The earlier the booking, the higher the risk. |
| 3 | **Deposit type behaves unexpectedly.** | `Non Refund` bookings show a counter-intuitive cancellation pattern (a "Non Refund paradox") that needs a closer look before drawing pricing or policy conclusions. |
| 4 | **Portugal leads bookings and revenue.** | Portugal (PRT) has 48,483 bookings and about €5.54M revenue. |
| 5 | **Online travel agents are the top revenue segment.** | Online TA brings in about €13.7M. |
| 6 | **Pricing varies against the monthly average.** | Comparing each booking's ADR with its month's average (Query 7) highlights pricing outliers and seasonal price swings. |

## Business questions

A hotel group operating a **City Hotel** and a **Resort Hotel** wants to understand:

1. Which bookings are most likely to be cancelled, and why?
2. Where does revenue come from (market segment, country, hotel)?
3. How do pricing (ADR) and seasonality behave over time?

## Dataset

- **Source:** *Hotel Booking Demand* (Antonio, Almeida & Nunes, 2019)
- **Size:** 119,390 rows × 32 columns (one row = one booking)
- **Period:** 2015 to 2017
- **Key fields:** `is_canceled`, `lead_time`, `deposit_type`, `market_segment`, `country`, `adr`, `stays_in_weekend_nights`, `stays_in_week_nights`, `customer_type`

> Revenue is not in the raw data. It is derived as `adr × (stays_in_weekend_nights + stays_in_week_nights)` and counted only for non-cancelled bookings.

## Tech stack

| Layer | Tool |
|---|---|
| Database | SQL Server |
| Query tool | SSMS |
| BI | Power BI Desktop (Power Query, DAX) |
| Version control | Git / GitHub |

## Workflow

### 1. Load data
Created `HotelBookingDB` and imported the CSV into `raw_bookings` with the SSMS Import Flat File wizard. Verified `COUNT(*) = 119,390`.

### 2. Clean data (SQL)
- Removed **180** junk rows with 0 adults, 0 children and 0 babies, leaving **119,210** rows in `clean_bookings`.
- Replaced missing values: `children`, `agent` and `company` set to 0.
- Found that `company` stored missing values as the **text string `'NULL'`**, not a real NULL, so a plain `IS NULL` check would have missed them.
- Checked data types: `is_canceled` is `bit`, so it is cast to `INT` before `SUM` and `AVG`.

### 3. Analyse (SQL)
Ten queries in [`SQL/queries.sql`](SQL/queries.sql) covering:

- `GROUP BY` and aggregate functions
- CTEs, including nested CTEs
- Window functions: `ROW_NUMBER`, `RANK`, `AVG() OVER (PARTITION BY ...)`
- Subqueries
- `OFFSET / FETCH`

Four views feed Power BI: `vw_bookings_detail`, `vw_cancellation_factors`, `vw_revenue_by_segment`, `vw_revenue_by_country`.

### 4. Build the dashboard (Power BI)
Three pages: **Overview**, **Cancellation Analysis**, **Revenue Analysis**.

- Connected to SQL Server; built a date dimension table and relationships
- DAX measures kept in a dedicated `_Measures` table
- Slicers for Year and Hotel, synced across pages
- Page navigation buttons

## Dashboard preview

![Overview](Screenshots/overview.png)
![Cancellation Analysis](Screenshots/cancellation.png)
![Revenue Analysis](Screenshots/revenue.png)

## Repository structure

```
├── SQL/
│   └── queries.sql      # cleaning, 10 analysis queries, 4 views
├── Powerbi/
│   └── *.pbix           # Power BI dashboard
├── Screenshots/         # dashboard screenshots
└── README.md
```

The raw dataset is not included. Download *Hotel Booking Demand* from Kaggle (`hotel_bookings.csv`).

## How to reproduce

1. Install SQL Server, SSMS and Power BI Desktop.
2. Create the database `HotelBookingDB`.
3. Download `hotel_bookings.csv` from Kaggle and import it into a table named `raw_bookings` (SSMS: Tasks, Import Flat File).
4. Open `SQL/queries.sql` in SSMS and run it in order. `clean_bookings` is created with `SELECT INTO`; to re-run, use `DROP TABLE IF EXISTS clean_bookings` first.
5. Open the `.pbix` file in `Powerbi/` and point the data source to your SQL Server instance (Transform data, Data source settings).

## Limitations

- Data covers only 2015 to 2017 for two hotels in Portugal, so results may not generalise.
- Rows with `adr = 0` (possible complimentary stays) and extreme ADR values are treated as outliers when computing revenue.
- Revenue is an estimate derived from ADR × nights.
- Findings are descriptive and should not be read as causal.

## Author

**Chloe Nguyen**, Sydney NSW
Power BI Data Analyst Associate · IBM Data Analyst Professional
[LinkedIn](https://www.linkedin.com/in/chloe-nguyen-990856166) · [GitHub](https://github.com/ngochuyen27196)

*Data: Hotel Booking Demand (Antonio, Almeida & Nunes, 2019).*
