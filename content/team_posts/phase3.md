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

## At a glance

{{< glance >}}
Updates since Phase II :: *[Persona refinements, multi-country ML1, API + frontend wiring — summarize here]*
ML 1 (price forecast) :: *[Country selection, feature additions, holdout metrics, 30-day API — summarize here]*
ML 2 (winter stress) :: *[Threshold tuning, model choice, cross-validation results — summarize here]*
Model implementation :: *[Training/serving pipeline, persistence, how models are loaded at request time — summarize here]*
Routes :: *[REST endpoints, request/response shape, which persona each route serves — summarize here]*
UI :: *[Household and journalist views connected to live data and models — summarize here]*
Phase III outcome :: *[Integrated app: models behind API, routes documented, two persona UIs demonstrable — summarize here]*
{{< /glance >}}

## Updates since Phase II

Since our last update we have focused on shipping an integrated experience rather than isolated notebooks and wireframes. The most notable changes are documented below:

- *[Bullet: persona or user-story updates tied to implemented UI/API behavior]*
- *[Bullet: ML1 changes — e.g. country selection, new features, improved metrics]*
- *[Bullet: ML2 changes — e.g. threshold, model selection, evaluation]*
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

**Price forecast (electricity)** — short-horizon outlook for household contract decisions.

**What changed since Phase II**

- *[Data: countries/zones added beyond DE_LU, date range, cleaning notes]*
- *[Features: new lags, rolling means, gas storage or demand if added]*
- *[Model: algorithm or hyperparameter changes, train/test split]*

**Performance**

| Metric | Phase II (DE_LU) | Phase III |
| ------ | ---------------- | --------- |
| R²     | ~0.27            | *[TBD]*   |
| MAE    | *[TBD]*          | *[TBD]*   |
| RMSE   | *[TBD]*          | *[TBD]*   |

**30-day forecast behavior**

- *[How start date and country are passed in; iterative lag roll-forward]*
- *[Link to chart or notebook artifact if not yet in UI]*

**What's left**

- *[Remaining ML1 items for final delivery or polish]*

## Model 2 updates

**Winter storage stress classifier** — predicts whether gas storage drops below 30% full during winter (will also take into account weather eventually)

**What changed since Phase II**

- *[Label or threshold adjustments; feature engineering updates]*
- *[Model choice: logistic regression vs random forest and why]*
- *[Evaluation: cross-validation, accuracy, precision/recall for stress class]*

**Performance**

| Model              | Accuracy | Stress-class precision/recall | Notes        |
| ------------------ | -------- | ----------------------------- | ------------ |
| Logistic regression | ~64%    | ~0.62 / ~0.62                 | Phase II baseline |
| Random forest      | ~66%    | *[TBD]*                       | *[Phase III]* |
| Phase III (chosen) | *[TBD]* | *[TBD]*                       | *[TBD]*      |

**Feature importance (dashboard copy)**

- *[Top drivers: storage_at_start, storage_trend_30d, storage_volatility — update if changed]*

**What's left**

- *[Supply-shock classifier, import dependence, export for policy persona, etc.]*

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

*[Brief intro: primary views, navigation from home, key interactions.]*

**Views & components**

- *[View 1: e.g. 30-day forecast chart + plain-language summary]*
- *[View 2: e.g. gas storage vs winter norms, neighbor comparison]*
- *[CRUD: country selection, optional monthly spend input]*

**Data binding**

- *[Which routes power which widgets; loading and error states]*

**Screenshots**

<!-- Replace with siteimg figures when assets are ready, e.g.:
{{< siteimg src="images/ui/household_forecast.png" alt="Household forecast view" width="600" >}}
-->

*[Add screenshots or link to deployed demo.]*

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
