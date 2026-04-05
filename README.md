# Blue Jays Perch
A **Toronto Blue Jays Analytics Dashboard** — Toronto Blue Jays analytics dashboard for roster insights, season trends. built as an end-to-end data pipeline platform designed to track team performance, individual player statistics, and competitive benchmarking
within the AL East division.


##  Data Source
All data is sourced from a single origin to ensure numerical consistency:

| Property         | Details                              |
|------------------|--------------------------------------|
| **Source**       | MLB Stats API                        |
| **Format**       | JSON Response                        |
| **Update Frequency** | Daily at **03:00 AM** via automated scheduler |


##  Project Structure

```
blue-jays-perch/
├── airflow/
│   └── dags/
│       ├── ingest_mlb_data.py       # DAG 1: Data ingestion
│       └── transform_dbt.py         # DAG 2: dbt trigger
├── dbt/
│   ├── models/
│   │   ├── silver/                  # Cleaned & conformed models
│   │   └── gold/                    # Data marts (wide flat tables)
│   └── tests/                       # dbt data quality tests
├── backend/
│   └── main.py                      # FastAPI application
├── frontend/
│   └── ...                          # Next.js + Recharts UI
├── data/
│   └── bronze/                      # Raw JSON files (date-partitioned)
└── README.md
```

##  Data Flow Architecture 

| Stage | Description |
|---|---|
| **Raw JSON (API)** | Source data collected from external APIs |
| **Bronze Layer** | Raw Zone storing date-partitioned JSON files for audit trail and recovery |
| **Silver Layer** | Cleaned Zone where dbt transforms raw data into a Star Schema in PostgreSQL |
| **Data Quality Tests** | Validation checks such as `unique` and `not_null` |
| **Gold Layer** | Serving Zone with pre-joined wide flat tables (Data Marts) optimized for API query performance |

- Full Architecture : [click here](medallion_bluejays_v2.svg)


##  Key Features
### Page 1 — Team Overview
- AL East Standings table with real-time rankings
- Season-long Win/Loss trend analysis
- Last 10 games summary with Home vs. Away performance split

### Page 2 — Roster & Player Insights
- Individual **Player Cards** with per-player stat breakdowns
- Readable **Stat Bars** for both Batting and Pitching metrics
- Advanced **Statcast** data integration — including *Exit Velocity* and *Hard Hit%*

### Page 3 — AL East Comparison
- Team-level standard statistics comparison across all AL East teams
- **Grouped Bar Charts** for multi-dimensional performance comparisons
- Blue Jays entries are **highlighted** for quick visual reference


##  Pipeline Logic

```
1. INGEST
   Airflow DAG triggers at 02:00 AM
   └── Pulls Standings, Roster (Full Refresh), and Games (Incremental)
       from MLB Stats API → stores raw JSON to Bronze Layer

2. TRANSFORM
   On successful ingest, a second DAG is triggered
   └── dbt runs cleaning and modeling jobs
       Bronze → Silver (Star Schema)
       Silver → Gold (Data Marts / Wide Flat Tables)

3. SERVER
   FastAPI reads from Gold Layer
   └── Returns pre-aggregated data to Next.js frontend
       with in-memory caching to minimize page load latency
```
