# Social Media Ad Performance Dashboard

**[The live dashboard on Tableau Public →](https://public.tableau.com/views/socialmediaadsperformancedashboard/Dashboard1)**

![Dashboard screenshot](../images/dashboard_screenshot.png)

Dataset: [Social Media Advertisement Performance](https://www.kaggle.com/datasets/alperenmyung/social-media-advertisement-performance) (Kaggle), a synthetic but realistic event log covering ~400K interactions across 200 ads, 50 different campaigns, and 10,000 users.

## The Objective

Built to help provide answers when determining a budget for different social ad campaigns:

- Where is the funnel leaking — impression to click to engagement to purchase?
- Which platform (Facebook & Instagram) is more efficient, not just larger?
- What is the difference in performance by audience (gender) or by day of the week?
- Which campaigns are worth scaling, and which have little return?

## Data Cleaning & Prep

Full detail in the [cleaning notebook](../notebook/01_data_cleaning.ipynb). In short:

- **Referential integrity check** — confirmed every `ad_id`, `user_id`, and `campaign_id` in the event log actually matched a record in its parent table before joining.
- **Duplicates** — found and dropped 50 duplicate `user_id` rows in the users table.
- **Datetime parsing** — event timestamps and campaign start/end dates loaded as strings by default; converted to proper datetimes.
- **Campaign date range investigation** — 56% of events fell outside their campaign's start/end window. Digging in deeper, the event timestamps only span about 3 months while the campaigns collectively span about 8, so this reads as a data generation artifact rather than a real signal about delayed engagement. Filtering these out would mean losing entire campaigns, so instead I attributed every event to its campaign by ID only, and left date matching out of that logic entirely.

## Dashboard Data Decisions

**No ROAS (Return on Ad Spending).** The dataset doesn't include a revenue field, only campaign level budget. Rather than create a dollar value per purchase to force a quantitative metric, I built the dashboard around the funnel and CTR, engagement rate, conversion rate, and cost per purchase. 

**Preventing Budget Duplication.** In Tableau, to stop the duplication of campagin budget across conversion events, a Level of Detail (LOD) expression ({FIXED [campaign_id] : MIN([total_budget])}) ensured each campaign's budget is aggregated exactly once.

## The Dashboard

- **KPIs** — CTR, engagement rate, conversion rate, cost per purchase, total budget, and total impressions, all filterable.
- **Ad funnel** — impression → click → engagement → purchase, with retention percentage at each stage so the drop-off is visible.
- **Efficiency by platform** — Facebook vs. Instagram, side by side across three rate metrics.
- **Efficiency by gender and day of the week** — same three metrics.
- **Campaign budgets & cost per purchase** — every campaign sorted by budget with cost per purchase overlaid as a line to spot campaign returns.
- Filters connected across every chart for platform, ad type, and date range.

## Key Findings

- Instagram edges out Facebook on CTR (11.86% vs. 11.76%) and engagement rate (5.38% vs. 5.28%), but Facebook has a better conversion rate (5.21% vs. 4.82%). So which overall platform is "better" depends on the stage of the funnel that's most important to decision makers and the objective.
- Performance by gender and by day of week is essentially flat. Nothing jumps out as an indicator to shift the budget here and there. That itself is useful to know as the audience and scheduling aren't where the optimization opportunity is in the dataset. Budgeting efficiently by *campaign* is where the spread shows.
- A handful of campaigns spend well above the median budget with spikes in cost per purchase to indicate that. Worth flagging as the first place to look if this were a real budget review.

## Repository Structure

```
├── README.md
├── notebooks/
│   ├── 01_data_cleaning.ipynb
├── images/
│   └── dashboard_screenshot.png
├── data/
│   ├── ad_events.csv
│   ├── events_master.csv
│   ├── ads.csv
│   ├── campaign_summary.csv
│   └── campaigns.csv
│   └── platform_summary.csv
│   └── users.csv
└── dashboard/
    └── social_media_ad_performance_dashboard.twbx
```

**Note:** `data/events_master.csv` (~400K rows) is excluded from this repo due to file size. 
It's fully reproducible by running `notebooks/01_data_cleaning.ipynb` against the raw source files.

## Tools
Python (pandas) for cleaning, joining, and building the funnel/summary tables. Tableau for the dashboard itself.
