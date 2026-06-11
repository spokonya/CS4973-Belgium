---
title: "Zeus - Phase IV"
date: 2026-06-11
draft: true
description: "EU Energy Security Index: Phase IV team update"
slug: "phase4post"
tags: ["project", "phase4", "energy", "Personas"]
showAuthorsBadges: false
showTableOfContents: true
---

# Phase IV — Team Update

## Project Recap

Energy security matters to everyone, but the data needed to understand it is scattered across fragmented official sources: ENTSO-E for electricity, GIE for gas storage, and Eurostat for the wider picture. Zeus pulls those signals into a single platform and turns raw European energy data into country-level insight, including day-ahead electricity price forecasts, gas-storage stress relative to historical norms, import dependence, and how each country compares to its neighbors. The goal is to surface the conditions that precede a supply shock, not just report one after it happens.

Zeus is built around three personas, each with its own view of the same platform:

- **Household Owner:** wants to understand their bill and where electricity prices are headed in their country.
- **Journalist:** needs defensible, country-level evidence to back stories on gas and electricity market stress, including which countries face winter storage risk.
- **Energy Trader:** wants a 30-day price path to trade against, a watchlist, and a journal to log forecast calls and review how they resolved.

Two machine-learning models power the analytics:

- **ML1:** 30-day electricity price forecast across 15 EU countries feeding both the Household Owner and Energy Trader personas.
- **ML2:** winter gas-storage stress classifier powering the Journalist risk pages.

In Phase 4, we focused on finalizing the implementation of the website. We reworked our interface design based on the feedback we got, connected the database to the Energy Trader persona so they can log their trade notes, and finalized our two machine learning models to make them as accurate as we could.


# Phase IV — Team Update

## ML #1 — Electricity Price Forecast

### The model
- A linear regression that forecasts daily electricity prices 30 days out for 15 EU countries, trained on ENTSO-E day-ahead price data. It uses 7 daily lags, rolling 7- and 30-day statistics, and month / day-of-week / country dummies, all standardized before fitting. _[teammate: confirm the feature list and add the data time range / number of rows.]_

### What changed this phase
- _[teammate: did the model itself change this phase, or was the main work deploying it into the app? Fill in what's new since the last phase.]_

### What didn't change
- The core modeling approach stayed the same as the previous phase — same linear regression, same features, and same single-shared-model design. _[teammate: confirm and add any detail.]_

### Model implementation
- The trained weights are exported from the notebook and stored as rows in the `price_model_weights` and `price_model_scaler` tables. The Flask API loads them and rebuilds the prediction in NumPy, rolling forward one day at a time to produce the 30-day forecast, served at `/ml1/forecast`. _[teammate: expand on how the recursive forecast works step by step.]_

### Checks (not shown in the web app)
- Evaluated on a held-out test set with R² ≈ 0.61, MAE ≈ 17.7, and RMSE ≈ 23.2, plus residual and predicted-vs-actual plots. _[teammate: confirm the numbers and link the plots.]_

## ML #2 — Winter Gas-Storage Stress

### The model
- A logistic regression that predicts before winter starts, whether a country's gas storage will drop below 30 during the winter (the "stress" level)
- Trained on GIE AGSI (Gas Infrastructure Europes Aggrgate Gas Supply Index) for 17 countries over the last 10 years
- It outputs a risk probability (0-1) through the sigmoid function

### What changed this phase
- We added an interaction term, `storage_volatility × storage_at_start` so volatility's effect on risk now depends on how full storage is entering winter, instead of being a one-directional linear term
- We found and fixed a real data bug in the volatility feature as volatility was computed on unsorted daily data, so the 90-day window grabbed the wrong days and the feature came out as noise (correlation with stress was only −0.015).
- Our fix was sorting the data by date before taking the last 90 days
- After the fix, volatility became a better predictor** (correlation +0.205) and its model weight changed
- As a result, the risk slider in the app now actually responds to volatility 
- Then we re-fit the model on the corrected data and recomputed all the weights (intercept, coefficients, and the standardization means/standard deviations)

### What didn't change
- Still a logistic regression (chosen over a random forest for its interpretability)
- Still uses balanced class weights as recall on the "stress" class matters most, since missing a genuinely risky winter is worse than a false alarm.
- Still the same 30% stress threshold. Additionally, gas is stored under pressure underground, so as the reservoir empties the internal pressure drops and gas physically comes out more slowly
- Still the same three base storage features (storage at start, 30-day trend, volatility)

### Model implementation
- We train the model in the notebook as a StandardScaler + LogisticRegression pipeline*
- We export the trained numbers of the intercept, the four weights, and each feature's mean and standard deviation and store them as one row in our `gas_storage_model` table.
- Our Flask API's `predict_risk` function rebuilds the prediction in plain Python and standardizes the inputs (including the interaction term), applies the weights, and runs the result through the sigmoid page calls the same function for every country.
- We verified the API's predictions match the trained scikit-learn model 


### Assumption Checks
- The outcome is binary (0/1, no-stress vs stress), with 103 no-stress and 81 stress winters, so the classes are fairly balanced
- In the correlation matrix the only notable link is storage_trend_30d and storage_volatility at 0.34, everything else is under 0.2, so no redundant features
- The one issue was multicollinearity as the raw storage_at_start × storage_volatility interaction pushed VIF up to 43.6, because it ends up 0.95-correlated with storage_volatility
- Mean-centering the features before multiplying fixes it every VIF drops back to about 1, and the interaction still works
- The Box-Tidwell test passes for all four features (p = 0.56, 0.34, 0.93, 0.83), so none need a transform
- For accuracy I used 5-fold stratified cross-validation, checked precision and recall on the confusion matrix, and compared logistic regression against a random forest


## Software Architecture

<!--
- Describe the three-tier, containerized architecture: Streamlit (UI) → Flask
  (API) → MySQL (data).
- Walk through the end-to-end request flow for one concrete user action.
- Explain the train-offline vs. serve-online split: heavy model training happens
  offline; the app only serves lightweight predictions at request time.
- Insert the architecture diagram.
-->

Zeus is built from three parts that each do one job: the Streamlit app the user clicks around in, a Flask API behind it, and a MySQL database underneath. All three run as separate Docker containers connected with docker-compose, so starting the whole project takes one command. The front end never touches the database or runs model math itself. When a page needs data it calls a single helper in `modules/zeus_api.py`, which is the only thing that talks to the API. Flask is the middle layer where the work happens: a request hits a route, the route pulls what it needs from MySQL, runs the model, and returns JSON.

One thing worth pointing out is that we don't load a model file at runtime. Once a model is trained, all that comes out is a set of numbers (the weights and a few scaling values), and we store those in the database. At prediction time the API reads them back and does the math itself, so the "deployed model" is really just rows in a table. All the actual training happens separately in Jupyter notebooks; the app never trains anything, it only serves predictions from numbers we already worked out and saved.

The easiest way to see how it fits together is to follow one request. A trader picks a country on the forecast page, the page calls our API helper, and Flask grabs the weights and recent prices from MySQL, builds the 30-day forecast, and returns it as JSON. The page draws the chart. The user just picks a country and gets a forecast.


<!-- {{< siteimg src="images/diagrams/Architecture.png" alt="Zeus three-tier architecture" width="700" >}} -->

## Final Database Model

<!--
- Present the final set of tables, grouped into:
  - Core / identity (Users, Persona, profiles)
  - Persona features (watchlists, alerts, saved articles, household profiles, etc.)
  - Model tables (price model weights/scaler, storage model, daily price/storage)
- Note primary keys and foreign keys that tie the groups together.
- Explain the "model lives in the DB" decision: why trained model parameters are
  stored as rows rather than as files, and what that buys the architecture.
- Reference / embed the final ER + relational diagrams (already on Phase II,
  updated here if changed).
-->

_TODO: table groupings (core/identity, persona features, model tables), keys/FKs, and the "model lives in the DB" rationale._

## Reflection & Next Steps

<!--
- What went well, what was hard, what you'd do differently.
- Honest limitations of the current models/architecture.
- Concrete next steps beyond this phase.
-->

_TODO: reflection on Phase IV and next steps._

## Individual posts

{{< members >}}
{{< member name="Ari Spokony" role="" href="/ari_spokony/" initials="AS" photo="images/team/ari.jpg" >}}
{{< member name="Bobby Bress" role="" href="/bobby_bress/" initials="BB" photo="images/team/bobby.jpg" >}}
{{< member name="Anjali Patel" role="" href="/anjali_patel/" initials="AP" photo="images/team/anjali.jpg" >}}
{{< member name="Rayna Patel" role="" href="/rayna_patel/" initials="RP" photo="images/team/rayna.jpg" >}}
{{< /members >}}
