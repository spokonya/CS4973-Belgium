---
title: "Zeus - Phase IV"
date: 2026-06-11
draft: false
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
A linear regression that forecasts daily electricity prices 30 days forward for 15 EU countries (AT, BE, BG, CZ, DE, ES, FR, HR, HU, LV, NL, PL, PT, RO, SK), trained on ENTSO-E day-ahead price data from January 2021. through May 2026. It uses 7 daily lags (`lag_1` through `lag_8`), rolling 7-day mean, 30-day mean, 7-day standard deviation, and price vs 7-day average ratio as rolling statistics, plus one-hot encoded month (drop_first drops January), day of week (drop_first drops Monday), and country (drop_first drops Austria) dummies — 42 features total, all standardized with StandardScaler.

### What changed this phase
- The model itself did not change
- The main work this phase was deploying the model end-to-end into the application: 
  - Exporting the trained weights and scaler parameters from the notebook into the database
  - Building the backend to have live predictions
  - Connecting the frontend dashboard to the API
  - Adding the historical price chart and hybrid forecast visualization (a chart with 15 days of historical data and the 30-day price forecast)

### What didn't change
- The modeling approach stayed the same as Phase III with the same linear regression, same 42 features, and same single shared model across all 15 countries
- No retraining was done this phase

### Model implementation
- The trained weights are exported from the notebook and stored as named columns in the `price_model_weights` table (one column per feature weight plus intercept) and the StandardScaler means and standard deviations are stored in `price_model_scaler`
- At prediction time, the API loads both tables, fetches the 30 most recent real prices for the selected country from `price_daily`
- A 30-step loop. Each step builds the full 42-element feature vector x from the current price history, scales x using the stored means and standard deviations, then computes wTx + intercept to produce one predicted EUR/MWh value
- Each prediction is appended to the price history so the next step uses it as lag_1, rolling the forecast forward one day at a time

### Assumption Checks
- We evaluated the model on the final 90 days of data withheld from training, achieving R²=0.608, MAE=17.68 EUR/MWh, and RMSE=23.21 EUR/MWh
- We validated the linear regression assumptions using three plots: 
  - Residuals over time chart (actual minus predicted on the test set) confirming residuals are randomly distributed around zero with no systematic bias
  - Residual distribution histogram showing the errors are approximately normally distributed

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
