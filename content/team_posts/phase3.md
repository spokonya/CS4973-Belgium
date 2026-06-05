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

Since our last update we have focused on shipping an integrated experience rather than isolated notebooks and wireframes. The most notable changes are documented below:

- *[Bullet: User story changes]*
- ML1 was significantly improved through feature engineering, the addition of 14 new EU countries with a single shared model with one-hot encoded country features. These changes increased the R² from 0.265 to 0.608.
- ML2 justified the 30% stress threshold with EU policy, dropped the random forest for the logistic regressionn, fixed a standardization bug, corrected our reported accuracy , and deployed the model into three interactive app pages
- *[Bullet: backend routes and how they map to Phase II “what’s left” items]*
- *[Bullet: frontend implementation status for household and journalist flows]*

## Updated user personas

We refined Lena and Marco against the flows we actually built in Phase III (saved preferences, spend input, forecast API, country snapshots, etc.). Sofia remains in scope for later phases; this update centers on household and journalist delivery.

### Persona 1 — Household Owner

{{< persona name="Lena Müller" role="Household Owner" age="30" location="Hamburg, Germany" initials="LM" type="household" photo="images/team/Lena.jpg" >}}
*[One paragraph: what changed since Phase I/II — e.g. monthly spend input, country selector, plain-language forecast summary on the dashboard.]*

**User stories**

1. As Lena, I want to see how electricity prices in *[country]* are forecast to move over the next 30 days, so I can decide whether to lock in a fixed-rate contract now or wait.
2. As Lena, I want to see how current gas storage levels compare to recent winters, so I can judge whether the upcoming winter is likely to be a high-bill season.
3. As Lena, I want a plain-language summary of the forecast, so I can understand what is driving the prediction without an economics background.
4. As Lena, I want to compare *[country]*'s situation to its neighbors, so I can tell whether price pressure is local or EU-wide.
5. *[Optional new story from Phase III — e.g. enter household spend and compare to national average.]*
{{< /persona >}}

### Persona 2 — Energy Journalist

{{< persona name="Marco Frite" role="Energy Journalist" age="30" location="Brussels, Belgium" initials="MF" type="journalist" photo="images/team/marco.jpg" >}}
*[One paragraph: what changed since Phase I/II — e.g. unified country snapshot, export/screenshot path, saved searches.]*

**User stories**

1. As Marco, I want current electricity prices, gas storage, and import dependence for each EU country in one place, so I can gather facts without bouncing between five sources.
2. As Marco, I want to compare a country's indicators to its neighbors, so I know which countries deserve attention in a story.
3. As Marco, I want a country snapshot I can screenshot or export, so figures in my article stay accurate without manual copy-paste.
4. As Marco, I want to compare today's prices and risk score to the same date in prior years, so I can frame whether the moment is historically unusual.
5. *[Optional new story from Phase III — e.g. save a search or filter set to resume reporting.]*
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

## Model implementation

How trained models are packaged, versioned, and invoked from the application backend.

**Training & artifacts**

- *[Where training scripts live; output paths for pickles, scalers, or ONNX/etc.]*
- *[How often models are retrained vs loaded from disk at startup]*

**Inference path**

1. *[Request hits route → validation → feature build from DB/API/cache]*
2. *[Load model + scaler → predict → format JSON for frontend]*
3. *[Error handling / fallback when country or date out of range]*

**Dependencies**

- *[Python stack: e.g. scikit-learn, pandas; any joblib or custom serializers]*

**Configuration**

- *[Env vars or config file: model paths, default country, API keys for live features]*

## Routes

REST (or equivalent) endpoints that connect the React frontend to model outputs and curated data.

| Method | Path | Purpose | Persona |
| ------ | ---- | ------- | ------- |
| GET    | *[e.g. `/api/forecast`]* | 30-day electricity price forecast | Household (Lena) |
| GET    | *[e.g. `/api/storage`]* | Gas storage vs norms | Household, Journalist |
| GET    | *[e.g. `/api/country/{code}/snapshot`]* | Country snapshot for reporting | Journalist (Marco) |
| GET    | *[e.g. `/api/compare`]* | Neighbor comparison | Both |
| POST   | *[e.g. `/api/household/spend`]* | Save household spend for comparison | Household (Lena) |
| *[…]*  | *[…]* | *[…]* | *[…]* |

**Example request/response**

```json
// GET /api/forecast?country=DE&start=2026-06-01
{
  "country": "DE",
  "start_date": "2026-06-01",
  "horizon_days": 30,
  "forecast": []
}
```

**Notes**

- *[Auth, CORS, rate limits, or mock vs live data flags]*
- *[Which routes are wired in Phase III vs stubbed]*

## UI implementation

Phase III UI moves from wireframes to working views for the two personas.

### Household persona (Lena)

Lena's flow is built around her own situation rather than the EU-wide view: a landing dashboard summarizing what her bill looks like now and where it's heading, a saved household profile that personalizes the bill-cycle details, and a curated news feed so she can put the forecast in context.

**Views & components**

- **Household owner dashboard** — default landing page after logging in. Three KPI cards summarize "your energy at a glance" (current per-kWh price, predicted % change for next month, days until the next bill), followed by a scatter-plot forecast of household energy price over the next 30 days produced by the ML price model.

- **Household persona info** — profile form that captures the household-level details used for bill reminders and country-level context: household/contact name, email, country, utility provider, typical bill amount (€), next bill due date, billing frequency, average monthly usage (kWh), tariff type, and free-form notes. Profile state is reflected in the banner ("No household profile saved yet…" before save, the saved details after) and submitted via "Create profile."

- **EU energy news** — separate page driven by NewsData.io's top-domain filter and a broad energy keyword search across major EU countries. A single "Fetch EU energy news" action loads the latest headlines on electricity, gas, renewables, and related EU energy topics, giving Lena editorial context for the model's numbers without leaving the app.

**Data binding**

- Dashboard KPIs and forecast chart → `/api/household/forecast` (returns current price, predicted change, days until next bill, and the 30-day price series).
- Profile create/update → `POST /api/household/profile` (stores the form fields keyed by email).
- EU energy news → `/api/news/eu` (proxies NewsData.io with the configured domain filter and keyword set).

### Journalist persona (Marco)

Marco's flow centers on three views built around the gas storage stress model: a 10-year historical country snapshot for grounding a story, a multi-country risk ranking for finding the next angle, and an interactive what-if model for explaining the underlying drivers.

**Views & components**

- **Country snapshot** — per-country dashboard with four KPIs (current storage level with 30-day delta, stressed winters on record, lowest winter level on record, and the 30% stress threshold), a 2014-present time series of storage % full with the 30% stress line marked, and a "Context for your story" panel with EU policy background, a risk-tier-specific recommendation banner, curated external links (GIE AGSI, EC gas storage policy, national TSO announcements, Eurostat import dependency, Ember, ENTSOG), and quick-jump buttons to the risk model and comparison views.

  {{< siteimg src="images/ui/journalist_snapshot_kpis.png" alt="Country snapshot — Poland KPIs" width="600" >}}

  {{< siteimg src="images/ui/journalist_snapshot_chart.png" alt="Country snapshot — Poland storage time series" width="600" >}}

  {{< siteimg src="images/ui/journalist_snapshot_story.png" alt="Country snapshot — context for your story" width="600" >}}

- **Country comparison** — runs the gas storage stress model across every available country and ranks them by risk probability. A summary banner flags how many countries (and which) the model predicts could fall below 30% storage this winter, naming the highest-risk country. A horizontal bar chart shows the full ranking with the 50% decision boundary marked, and an optional country filter narrows the view.

  {{< siteimg src="images/ui/journalist_comparison.png" alt="Country comparison — winter risk ranking" width="600" >}}

- **Gas storage risk** — logistic regression trained on GIE AGSI winters 2015–2024. Country dropdown plus three sliders (storage level entering winter, change in storage over October, storage volatility over the past 90 days) returning a risk probability and an at-risk/not-at-risk label. Sliders default to the country's most recent winter so journalists see the live prediction first and can then explore what-if scenarios.

  {{< siteimg src="images/ui/journalist_storage_risk.png" alt="Gas storage risk model — Poland" width="600" >}}

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
