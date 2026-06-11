---
title: "Week 4 Blog Post"
date: 2026-06-11
draft: false
description: "Rayna's Fourth Blog Post"
slug: "rayna-week4"
tags: ["DS3000", "Leuven", "Linear Algebra", "Belgium"]
authors:
  - "Rayna_Patel"
showAuthorsBadges : false
---

# Phase 4 Blog Post

## My Contributions to Phase IV
My contribution to Phase IV was connecting ML1 to the full application, taking the model from a Jupyter notebook into a live, database-backed REST API serving real-time 30-day electricity price forecasts to an interactive frontend dashboard.

### Backend Integration

The core of Phase IV for ML1 was building the backend to serve predictions at runtime. I created `electricity_price_model.py`, which loads the model's 42 learned weights and intercept from the `price_model_weights` database table and the StandardScaler parameters from `price_model_scaler`, builds the full feature vector x at prediction time from the most recent prices (aligning with the country chosen by the user) pulled from `price_daily`, and computes wTx + intercept iteratively 30 times rolling each prediction forward as the new lag_1 input for the next day.

### Frontend Dashboard

On the frontend, I built the Household Owner Dashboard (`40_Household_Owner_Dashboard.py`) for persona Lena. The dashboard includes a country selector that defaults to the user's saved profile country (but can be changed through the drop down selector), a live ENTSO-E API call for today's actual day-ahead price, a predicted price change metric computed from the 30-day forecast, a hybrid historical/forecast line chart showing 15 days of real histroical prices alongside the 30-day model output, and a monthly seasonality bar chart pulling historical averages from the database. I also contributed to the Energy Trader dashboard pages for persona Niels, adding uncertainty shading to the 30-day forecast chart and a redesigned My Markets page with a combined multi-zone chart and side-by-side metrics and price alerts.

## Belgium Update

During our final week in Belgium, we visited the C-Mine in Genk, a former coal mining site, and EnergyVille, a research institute focused on sustainable energy systems. These visits were intriguing to me as they tied directly to the themes of our project. Seeing the actual research being done on the changing energy industry in person added real context to everything we built. Belgium has been an incredible experience from start to finish and I am grateful for every part of it.

{{< figure src="https://upload.wikimedia.org/wikipedia/commons/thumb/2/29/C_mine_limburg.jpg/1920px-C_mine_limburg.jpg?_=20170105163710" alt="C-Mine" caption="C-Mine — Wikimedia Commons" >}}