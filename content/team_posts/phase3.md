---
title: "Zeus - Phase III"
date: 2026-06-04
draft: false
description: "EU Energy Security Index: Phase III team update"
slug: "phase3post"
tags: ["project", "phase3", "energy", "Personas"]
showAuthorsBadges: false
showTableOfContents: true
---

# Phase III — Team Update

The EU Energy Security Index is our per-country dashboard for energy security risk, import dependence, gas storage, and electricity prices. In Phase III we moved from prototypes to integration: refined personas around live app flows, improved both ML models, exposed forecasts and risk scores through REST routes, and shipped persona-specific UI for Lena (household) and Marco (journalist). This post summarizes what changed since Phase II and how the pieces connect end to end.

## Updates since Phase II

Since our last update, we have focused on refining our personas and user stories to more accurately reflect how individuals would interact with the application and to identify which features carry the most value for each user type. This, combined with our finalized models and updated database schema, made meeting the Phase III requirements more manageable.

- User stories and personas were revamped to more accurately represent realistic user needs and to motivate a fuller set of CRUD operations across all three personas.
- ML1 was significantly improved through feature engineering, the addition of 14 new EU countries with a single shared model with one-hot encoded country features. These changes increased the R² from 0.265 to 0.608.
- ML2 justified the 30% stress threshold with EU policy, dropped the random forest for the logistic regressionn, fixed a standardization bug, corrected our reported accuracy , and deployed the model into three interactive app pages
- Our database schema was updated to better support our finalized features and ML models. Updated ER diagrams and SQL will replace the outdated resources from our previous blog post in the coming days.

## Updated user personas

We refined Lena and Marco against the flows we actually built in Phase III (saved preferences, spend input, forecast API, country snapshots, etc.). Sofia remains in scope for later phases; this update centers on household and journalist delivery.

### Persona 1 — Household Owner

{{< persona name="Lena Müller" role="Household Owner" age="30" location="Hamburg, Germany" initials="LM" type="household" photo="images/team/Lena.jpg" >}}
Since Phase I, Lena's persona has been grounded in a concrete billing intake flow. She now enters her utility provider, monthly bill amount, tariff type, and billing cycle dates through a profile form on her dashboard. This gives the dashboard enough context to count down to her next payment due date and frame the 30-day electricity price forecast directly against what she is currently paying. The plain-language summary remains central to her experience — she has no economics background and needs the forecast translated into a simple contract decision.

**User stories**

1. As Lena, I want to see how electricity prices in Germany are forecast to move over the next 30 days, so I can decide whether to lock in a fixed-rate contract now or wait.
2. As Lena, I want to see how current gas storage levels compare to recent winters, so I can judge whether the upcoming winter is likely to be a high-bill season.
3. As Lena, I want a plain-language summary of the forecast, so I can understand what is driving the prediction without an economics background.
4. As Lena, I want to compare Germany's situation to its neighbors, so I can tell whether price pressure is local or EU-wide.
5. **[NEW]** As Lena, I want to enter my billing cycle dates, monthly rate, and tariff type so the dashboard can show me a countdown to my next payment and flag whether the forecast suggests prices will be higher or lower by then.
6. **[NEW]** As Lena, I want to save news articles about EU energy to a personal reading list, so I can review them later before making a contract decision.
{{< /persona >}}

---

### Persona 2 — Energy Journalist

{{< persona name="Marco Frite" role="Energy Journalist" age="30" location="Brussels, Belgium" initials="MF" type="journalist" photo="images/team/marco.jpg" >}}
Since Phase I, Marco's persona has been sharpened around two problems: the need to cite figures accurately at a specific point in time, and the need to track story leads without losing context between sessions. The country snapshot feature addresses the first — Marco can freeze a country's indicators and ML outputs as a timestamped record rather than relying on whatever the dashboard shows at publication time. Saved notes address the second by offering the ability to attach notes to any data visualization, and edit or delete them as a story develops or goes cold.

**User stories**

1. As Marco, I want current electricity prices, gas storage, and import dependence for each EU country in one place, so I can gather facts without bouncing between five sources.
2. As Marco, I want to compare a country's indicators to its neighbors, so I know which countries deserve attention in a story.
3. As Marco, I want a country snapshot I can screenshot or export, so figures in my article stay accurate without manual copy-paste.
4. As Marco, I want to compare today's prices and risk score to the same date in prior years, so I can frame whether the moment is historically unusual.
5. **[NEW]** As Marco, I want to save a country's indicators and model outputs as a timestamped snapshot, so the figures I cite in an article reflect the data as it was when I checked
6. **[NEW]** As Marco, I want to annotate a data point with a private note flagging why it looks newsworthy
7. **[NEW]** As Marco, I want to browse, edit, and delete my notes, so I can track which leads I have already investigated and which are still open.
{{< /persona >}}

---

### Persona 3 — Energy Trader

{{< persona name="Niels Becker" role="Energy Trader" age="34" location="Amsterdam, Netherlands" initials="NB" type="trader" photo="images/team/niels.jpg" >}}
Niels trades electricity on the EPEX day-ahead market from a desk in Amsterdam. He positions his book 24 to 72 hours out and uses the 30-day price forecast as a directional input to that process. He works fast, trusts quantitative outputs, and keeps a running log of the decisions he makes against the data he acted on so he can evaluate forecast quality over time. Unlike Lena, who checks prices a few times a year, Niels interacts with the forecast daily and needs the dashboard scoped tightly to the markets he is actively trading.

**User stories**

1. As Niels, I want to manage a watchlist of bidding zones so my dashboard shows only the markets I am actively trading, without noise from the full EU view.
2. As Niels, I want to see the 30-day price forecast and gas storage stress risk score side by side for each watched zone, so I can assess both the price direction and the supply-side risk driving it in one view.
3. As Niels, I want to set a price alert on a bidding zone so the dashboard flags me when the forecast crosses a threshold I define, without me having to check manually throughout the day.
5. As Niels, I want to add a post-trade outcome annotation to a past trade note, so I can record whether the forecast called it correctly and build a personal track record over time.
6. As Niels, I want to browse my trade note history filtered by bidding zone and date range, so I can review patterns in how I have been responding to the forecast across different markets.

{{< /persona >}}

## Model 1 updates

**Price Forecast** — 30-day forecast of electricty prices in a certain EU country

**What changed since Phase II**

- We removed `year` as a feature, as it does not support a linear trend, especially considering the 2022 outlier 
- We changed the lag features from 1, 2, 3, 7, 14, and 30 to a consecutive 1-7
- We used one-hot encoding to replace raw `month` and `dayofweek` numbers, as well as `drop_first=True`
- We added `rolling_7d_std` as a feature to capture price volatility and `price_vs_7d_avg` as a feature to capture if current prices are above/blow the recent trend
- We added another EDA graph, a scatterplot, to check that a quadratic, cubic, square root, etc. relationship didn't exist (each year in the scatter plot had a different shape, especially 2022)

  *(The changes above increased the R² from 0.265 to 0.320.)*

- We added 14 additional countries beyond Germany (SK, HR, PT, BG, RO, HU, CZ, PL, ES, AT, BE, NL, LV, FR), all of which pulled from ENTSOE-E API, and combined them into one dataset of 29,565 rows across 15 countries
- We made country a one-hot encoded feature, so the model was able to learn country-specific price level effects while also sharing EU patterns (this will be added as an interactive input feature in the front-end)

  *(The changes above increased the R² from 0.320 to 0.608.)*

**Performance**

| Metric | Phase II (DE_LU) | Phase III |
| ------ | ---------------- | --------- |
| R²     | 0.265            | 0.608     |
| MAE    | --               | 17.68     |
| RMSE   | --               | 23.21.    |

**30-day forecast behavior**

- ML1 accepts any start date and any of the 15 country codes — if the start date is within the dataset it uses real historical prices to predict the 30-day forecast, if it's beyond, it forecasts forward from the last known date
- The forecast output is a 30-row dataframe of date and predicted EUR/MWh, saved to a CSV and visualized as an interactive Plotly chart

**What's left**

- Connect the model to the frontend in Phase 4

## Model 2 updates

**Winter storage stress classifier** — predicts whether gas storage drops below 30% full during winter and if the country will be at risk

**Changes since Phase II**

- We justified the 30% threshold more clearly. Essentially, it's a policy relevant EU regulation set in 2022 for countries to mandate 90% storage by Nov 1st. EU Analysts now treat 28-30% as the stress zone. It's also physically meaningful as lower storage causes lower pressure meaning lower withdrawls. Additionally, when testing other thresholds like 40%, every 1 in 5 winters were flagged and I thought it would be too common to use as an alert
- Decided to drop the random forest model and only stick with logistic regression. 
- We caught a bug in our feature importance as features have different scales, so ranking raw coefficients reflected units rather than actual influence. We fixed it by standardizing all features with a StandardScaler pipeline, and scaled inside each CV fold
-  We previously wrote that StratifiedKFold stratifies by country, but it actually stratifies on the class label so each fold keeps the same ratio of stress to no-stress winters. We also tried GroupKFold by country but dropped it since we're predicting future winters for these same 17 countries, and grouping by country made the folds too small and unstable.
- Corrected reported numbers
- We deployed to the app so now the model powers three pages: Gas Storage Risk, Country Comparison, and Country Snapshot

**Features**

- `storage_at_start` — how full a country's storage is at the start of winter. The bigger the buffer, the more cold weather it can absorb before hitting 30%
- `storage_trend_30d` — whether storage was filling up or draining in the last 30 days before winter. Two countries can start at the same level, but one still adding gas is in much better shape than one already using
- `storage_volatility` — how steady or jumpy storage levels were before winter. Smooth changes suggest planned filling and big swings suggest supply problems or demand shocks that may carry into winter

**Performance**

| Model | Accuracy | Stress-class precision/recall | Notes |
| --- | --- | --- | --- |
| Logistic regression (Phase II, unstandardized) | 66% | 0.62 / 0.62 | Baseline |
| Logistic regression (Phase III, standardized, chosen) | 66% | 0.62 / 0.56 | Deployed in app |

Standardizing barely moved performance since logistic regression predictions are fairly scale-robust but it fixed the interpretation

**Feature importance (standardized logistic regression coefficients)**

| Feature | Coefficient |
| --- | --- |
| `storage_at_start` | −0.774 |
| `storage_trend_30d` | −0.374 |
| `storage_volatility` | −0.127 |

This was honestly surprising because before standardizing, `storage_at_start` looked like the weakest predictor simply because its 0–100 scale shrinks its coefficient, but once all three features were put on the same scale it became by far the strongest driver and since all three coefficients are negative (fuller, rising, steadier storage all lower stress risk), this also corrects our Phase II claim that volatility mattered more than the trend.

**What's left**
 
- Import dependence features so the model isn't storage-only
    - Weather/temperature features
- Keep refining the journalist persona pages on webapp

## Routes
REST endpoints that connect the Streamlit frontend to model outputs and curated data.

| Method | Path | Purpose | Persona |
| ------ | ---- | ------- | ------- |
| GET | `/users?persona=household_owner` | List demo users for login dropdown | Both |
| GET | `/users?persona=journalist` | List demo users for login dropdown | Both |
| GET | `/users/{user_id}` | Fetch logged-in user record | Both |
| GET | `/users/{user_id}/household-profile` | Read saved household profile | Household (Maria) |
| POST | `/users/{user_id}/household-profile` | Create household profile | Household (Maria) |
| PUT | `/users/{user_id}/household-profile` | Update household profile | Household (Maria) |
| DELETE | `/users/{user_id}/household-profile` | Delete household profile | Household (Maria) |
| GET | `/stats/storage/history?country=DE` | Daily gas storage % series (AGSI) | Journalist (James) |
| GET | `/countries/{code}/storage/summary` | Latest storage level, 30-day delta, stressed-winter stats | Journalist (James) |
| GET | `/stats/storage/winters?country=DE` | Winter feature rows (slider defaults, charts) | Journalist (James) |
| GET | `/stats/storage/winters` | All countries' winter rows | Journalist (James) |
| GET | `/stats/storage/risk/compare` | Gas storage risk ranking (logistic regression) | Journalist (James) |
| POST | `/stats/storage/risk` | What-if risk prediction from model inputs | Journalist (James) |
| GET | `/news/eu-energy` | Latest EU energy news (NewsData.io proxy) | Household (Maria) |

**Notes**

- Streamlit calls the API via `app/src/modules/zeus_api.py` (`ZEUS_API_BASE`, default `http://web-api:4000`).
- Auth is mock-only: persona chosen on Home; no passwords or JWT.
- Gas storage data lives in MySQL, seeded from `datasets/apsi/agsi_clean.csv` and `dataset.csv` (`docker compose exec api python scripts/seed_gas_storage.py`). Model weights are in `gas_storage_model` (inserted by `03_gas_storage_schema.sql`).
- News uses live NewsData.io data; requires `NEWSDATA_API_KEY` in `api/.env`.
- Not implemented yet: `/stats/prices/forecast` and `/stats/prices/history` — `40_Household_Owner_Dashboard` still uses placeholder metrics; price ML is in `datasets/entsoe/` notebooks only.

**Streamlit page wiring**

| Page | Routes used |
|------|-------------|
| `Home.py` | `GET /users` |
| `41_Household_Persona_Info.py` | Household profile CRUD |
| `42_Household_Energy_News.py` | `GET /news/eu-energy` |
| `Country_Snapshot.py` | `GET /countries/{code}/storage/summary`, `GET /stats/storage/history` |
| `Country_Comparison.py` | `GET /stats/storage/risk/compare` |
| `Gas_Storage_Risk.py` | `GET /stats/storage/winters`, `POST /stats/storage/risk` |



## UI implementation

Phase III UI moves from wireframes to working views for the two personas.

### Household persona (Lena)

Lena's flow is built around her own situation rather than the EU-wide view: a landing dashboard summarizing what her bill looks like now and current energy pricing, a saved household profile that personalizes the bill-cycle details (amongst other details specific to the user which we plan to use for the creation of additioanl screen that could for additional CRUD operations), and a curated news feed so she can put the models prediction in context of current events.

**Views & components**

- **Household owner dashboard** — default landing page after logging in. Three KPI cards summarize "your energy at a glance" (current per-kWh price, predicted % change for next month, days until the next bill), followed by a scatter-plot forecast of household energy price over the next 30 days produced by the ML price model (Figures 1 and 2).

  {{< siteimg src="images/phase3/figure1.png" alt="Household dashboard — your energy at a glance KPI cards" width="600" caption="Figure 1: Your energy at a glance — household dashboard KPIs" >}}

  {{< siteimg src="images/phase3/figure2.png" alt="Household dashboard — 30-day ML price forecast scatter plot" width="600" caption="Figure 2: Predicted energy price forecast — 30-day ML outlook" >}}

- **Household persona info** — profile form that captures the household-level details used for bill reminders and country-level context: household/contact name, email, country, utility provider, typical bill amount (€), next bill due date, billing frequency, average monthly usage (kWh), tariff type, and free-form notes. Profile state is reflected in the banner ("No household profile saved yet…" before save, the saved details after) and submitted via "Save profile" (Figure 3).

  {{< siteimg src="images/phase3/figure3.png" alt="Household persona info — saved profile form" width="600" caption="Figure 3: Household persona info — profile form and saved state" >}}

- **EU energy news** — separate page driven by NewsData.io's top-domain filter and a broad energy keyword search across major EU countries. A single "Fetch EU energy news" action loads the latest headlines on electricity, gas, renewables, and related EU energy topics, giving Lena editorial context for the model's numbers without leaving the app (Figure 4).

  {{< siteimg src="images/phase3/figure4.png" alt="EU energy news — fetch headlines from NewsData.io" width="600" caption="Figure 4: EU energy news — curated headlines for household context" >}}

**Data binding**

- Dashboard KPIs and forecast chart → `/api/household/forecast` (returns current price, predicted change, days until next bill, and the 30-day price series).
- Profile create/update → `POST /api/household/profile` (stores the form fields keyed by email).
- EU energy news → `/api/news/eu` (proxies NewsData.io with the configured domain filter and keyword set).

### Journalist persona (Marco)

Marco's flow centers on three views built around the gas storage stress model: a 10-year historical country snapshot for grounding a story, a multi-country risk ranking for finding the next angle, and an interactive what-if model for explaining the underlying drivers.

**Views & components**

- **Country snapshot** — per-country dashboard with four KPIs (current storage level with 30-day delta, stressed winters on record, lowest winter level on record, and the 30% stress threshold), a 2014-present time series of storage % full with the 30% stress line marked, and a "Context for your story" panel with EU policy background, a risk-tier-specific recommendation banner, curated external links (GIE AGSI, EC gas storage policy, national TSO announcements, Eurostat import dependency, Ember, ENTSOG), and quick-jump buttons to the risk model and comparison views (Figures 5–7).

  {{< siteimg src="images/ui/journalist_snapshot_kpis.png" alt="Country snapshot — Poland KPIs" width="600" caption="Figure 5: Country snapshot — Poland KPI cards" >}}

  {{< siteimg src="images/ui/journalist_snapshot_chart.png" alt="Country snapshot — Poland storage time series" width="600" caption="Figure 6: Country snapshot — Poland storage time series" >}}

  {{< siteimg src="images/ui/journalist_snapshot_story.png" alt="Country snapshot — context for your story" width="600" caption="Figure 7: Country snapshot — context for your story" >}}

- **Country comparison** — runs the gas storage stress model across every available country and ranks them by risk probability. A summary banner flags how many countries (and which) the model predicts could fall below 30% storage this winter, naming the highest-risk country. A horizontal bar chart shows the full ranking with the 50% decision boundary marked, and an optional country filter narrows the view (Figure 8).

  {{< siteimg src="images/ui/journalist_comparison.png" alt="Country comparison — winter risk ranking" width="600" caption="Figure 8: Country comparison — winter risk ranking by country" >}}

- **Gas storage risk** — logistic regression trained on GIE AGSI winters 2015–2024. Country dropdown plus three sliders (storage level entering winter, change in storage over October, storage volatility over the past 90 days) returning a risk probability and an at-risk/not-at-risk label. Sliders default to the country's most recent winter so journalists see the live prediction first and can then explore what-if scenarios (Figure 9).

  {{< siteimg src="images/ui/journalist_storage_risk.png" alt="Gas storage risk model — Poland" width="600" caption="Figure 9: Gas storage risk model — what-if scenario sliders" >}}

**Data binding**

- Snapshot KPIs, time series, and context panel → `/api/country/{code}/snapshot` backed by GIE AGSI historical storage.
- Comparison ranking → `/api/storage/risk/ranking` (runs the same model per country, returns sorted probabilities).
- Storage risk what-if → `/api/storage/risk` (POST with slider values, returns probability + class label).


## Individual posts

Each teammate also published an individual update documenting their direct Phase III contributions.

{{< members >}}
{{< member name="Ari Spokony" role="" href="/ari_spokony/" initials="AS" photo="images/team/ari.jpg" >}}
{{< member name="Bobby Bress" role="" href="/bobby_bress/" initials="BB" photo="images/team/bobby.jpg" >}}
{{< member name="Anjali Patel" role="" href="/anjali_patel/" initials="AP" photo="images/team/anjali.jpg" >}}
{{< member name="Rayna Patel" role="" href="/rayna_patel/" initials="RP" photo="images/team/rayna.jpg" >}}
{{< /members >}}
