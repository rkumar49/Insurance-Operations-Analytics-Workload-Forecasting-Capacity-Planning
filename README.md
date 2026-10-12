# Insurance Operations Analytics: Workload Forecasting & Capacity Planning

I built this project to understand how an insurance operations center works — how work arrives, how teams handle it, and what happens when demand outpaces capacity. It simulates two years of operations data, calculates the KPIs that matter, forecasts future workload, and recommends how to fix the staffing problem.

The short version: the team isn't short on people overall. They're short **at the wrong times**. This project shows why, and what to do about it.

---

## The Problem

Imagine an insurance company with a 70-person operations team. Every day they handle about 1,200 calls and 800 emails/tickets. Each person works 7.5 productive hours a day.

Now here's the catch — **work doesn't arrive evenly**:

- Mondays are 20% busier than average
- November and December jump 30%
- Weekends are quiet (40–50% less work)

But the team is the same 70 people every single day. So what happens?

- Work piles up on busy days
- The pile never fully clears
- Customers wait longer
- The 2-day resolution promise (SLA) gets broken

By the end of 2024, customers were waiting **20+ days** for their issues to be resolved. That's not a small problem — that's a regulatory and reputational risk.

---

## What I Did

I broke the project into four steps:

1. **Generated realistic data** — 2 years of daily operations, with seasonality, weekday patterns, and random noise
2. **Calculated KPIs in SQL** — workload, capacity, backlog, lead time, FTE gap
3. **Explored the data** — charts showing trends, patterns, and problems
4. **Forecasted the future** — Holt-Winters model predicting the next 30 days of workload

The goal was to answer one question: **How many people do we actually need, and when?**

---

## The KPIs (Plain English)

| KPI | What it means |
|-----|---------------|
| **Workload** | Total hours of work that arrives in a day |
| **Capacity** | Total hours the team can actually work |
| **AHT** | Average time to handle one call |
| **Backlog** | Unfinished work carried over to the next day |
| **Lead time** | How many days it would take to clear the backlog |
| **FTE** | Full-Time Equivalent — one person working full-time |
| **FTE gap** | How many people we're short (or over) |
| **SLA breach** | Days we missed our 2-day resolution promise |

If you want the formulas:
Workload = (calls × AHT ÷ 60) + (tickets × 30 min ÷ 60)
Capacity = FTE × 7.5 hours
Backlog = yesterday's backlog + (workload − capacity)
Lead time = backlog ÷ daily capacity
FTE required = workload ÷ 7.5
FTE gap = FTE required − FTE available

text

---

## How I Built the Data

Real insurance data is confidential, so I simulated it — but not randomly. I layered **five realistic patterns**:

1. **Base volume** — 1,200 calls and 800 tickets per day
2. **Seasonality** — Q4 +30%, summer −20%
3. **Day-of-week** — Monday +20%, Sunday −50%
4. **Month-end** — Days 26–31 get +15% (billing cycles)
5. **Random noise** — natural daily variation

This makes the data behave like real operations — which is why the forecast is meaningful.

**Period covered:** January 2023 to December 2024 (731 days).

---

## What I Found

Five things stood out:

**1. Q4 is a crisis every year.**
November and December need about 20 more people than we have. This happened in 2023 and again in 2024 — same mistake twice.

**2. Backlog compounds.**
Once Q4 2023 hit, the backlog never fully cleared. Lead time went from hours to 20+ days — and stayed there all through 2024. A bad quarter poisons the entire next year.

**3. Mondays are the worst.**
Monday needs 119 FTEs. Sunday needs 54. Flat staffing fails both days — drowning on Monday, wasting capacity on Sunday.

**4. It's a scheduling problem, not a headcount problem.**
On an annual average, the team is correctly sized. The problem is timing — overstaffed in summer and weekends, understaffed on Mondays and in Q4.

**5. We've been breaching SLA 100% of days since November 2023.**
Fourteen months of missed promises. That's a red flag for customers and regulators.

---

## Forecasting

I used **Holt-Winters Exponential Smoothing** — a time-series model that captures three things:

- The base workload level (~470 hours/day)
- Long-term trend (slight growth)
- Weekly seasonality (Mondays high, weekends low)

**Why Holt-Winters?**
Because it handles both trend and seasonality, it's easy to explain to management, and it's fast. No black box.

**How accurate is it?**

| Metric | Result |
|--------|--------|
| MAPE | **5.4%** (94.6% accuracy) |
| MAE | 30.8 hours/day |
| RMSE | 40.1 hours/day |

In plain terms: if we predict 700 hours for a day, actual will be between 662 and 738 hours. That's good enough for planning.

---

## Recommendation

The forecast says: over the next 30 days, we need **90 FTEs on average**. We have 70. That's a **20-person gap**.

But hiring 20 permanent people would cost ~€1M per year and be wrong for summer. Here's what I'd actually recommend:

| Action | Impact | Cost |
|--------|--------|------|
| Shift 10 FTEs from weekends to Mondays | Fixes Monday crisis | €0 |
| Hire 10 permanent FTEs | Covers Tue–Fri gap | ~€500k/yr |
| Add 20 part-time weekend staff | Weekend surge cover | ~€150k/yr |
| Automate email triage | −12 FTE workload | ~€50k one-time |

**Net saving vs blanket hiring: ~€400k/year. SLA goes from 0% → 95%+.**

---

## Project Structure
Insurance-Operations-Analytics-Workload-Forecasting-Capacity-Planning/
│
├── data/
│ └── insurance_operations_data.csv # 731 days of simulated data
│
├── scripts/
│ ├── data_generation.py # Creates the synthetic data
│ ├── load_to_sqlite.py # Loads CSV into SQLite
│ ├── kpi_queries.sql # 10 SQL queries for KPIs
│ ├── eda_analysis.py # Charts and insights
│ └── forecast_model.py # Holt-Winters forecasting
│
├── outputs/
│ ├── kpi_monthly.csv
│ ├── kpi_weekly.csv
│ ├── kpi_channel_split.csv
│ ├── kpi_sla_breach.csv
│ ├── kpi_worst_days.csv
│ ├── kpi_quarterly.csv
│ └── forecast_next_30_days.csv
│
├── charts/
│ ├── 01_workload_vs_capacity.png
│ ├── 02_backlog_trend.png
│ ├── 03_lead_time.png
│ ├── 04_fte_gap.png
│ ├── 05_day_of_week.png
│ ├── 06_monthly.png
│ ├── 07_correlation.png
│ ├── 08_forecast.png
│ └── 09_forecast_fte_gap.png
│
├── .gitignore
├── README.md
└── requirements.txt

text

---

## How to Run

```bash
# 1. Install the libraries
pip install -r requirements.txt

# 2. Generate the data
python scripts/data_generation.py

# 3. Load into SQLite
python scripts/load_to_sqlite.py

# 4. Run the analysis and charts
python scripts/eda_analysis.py

# 5. Train the forecast model
python scripts/forecast_model.py
Tools I Used
Python — Pandas, NumPy for data · Matplotlib, Seaborn for charts · Statsmodels for forecasting

SQL — SQLite for storing data and calculating KPIs

Forecasting — Holt-Winters Exponential Smoothing

Version control — Git and GitHub

What This Project Taught Me
Backlog compounds. A bad quarter doesn't just cost you that quarter — it costs you the next year.

Averages hide problems. The team looks correctly sized on paper, but the timing is wrong.

Forecasting is a decision tool, not a prediction. The value isn't the number — it's the action it triggers.

Data quality matters. Even simulated data needs realistic patterns, or the forecast is meaningless.

Author
Rahul Kumar


Note
All data in this project is synthetic. No real customer or company data is used. The methodology transfers directly to real data — only the source changes.
