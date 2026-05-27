---
title: "Week 2 Blog Post"
date: 2026-05-27
draft: false
description: "Rayna's Second Blog Post"
slug: "rayna-intro"
tags: ["DS3000", "Leuven", "Linear Algebra", "Belgium"]
authors:
  - "Rayna_Patel"
showAuthorsBadges : false
---

# Phase 2 Blog Post

## My Contributions to Phase II
My contribution to Phase II was building the ML1 pipeline, from pulling raw data to training and evaluating the linear regression model.

### Data Pipeline

First, I pulled hourly day-ahead electricity prices for Germany (DE_LU bidding zone) from the ENTSO-E API using `entsoe-py`, covering January 2021 through May 2026 ~ 64,267 hourly records. Then, I wrote the full cleaning pipeline: parsed timezone-aware timestamps, extracted time features (hour, day of week, month, year, whether it was a weekend), and aggregated hourly data into 1,971 daily averages. I saved both hourly and daily cleaned files to CSVs to avoid re-running the API.

### EDA and Visualizations

For EDA and visualizations, I built all three EDA charts using Plotly: the full price history with the Ukraine invasion period highlighted, the monthly seasonality bar chart, and the year-over-year comparison. These charts answered the question of what seasonal patterns Lena should know about when deciding whether to lock in a fixed-rate contract.

### ML Model

I trained and evaluated a Linear Regression model with StandardScaler normalization on a 90-day test set and built the `predict_from_date()` function that accepts any start date and iteratively forecasts 30 days forward, seeding from real historical prices when available and rolling lag features forward with each step. The model achieved an R² of 0.265 on the test set, with the coefficient analysis showing lag_1 as the dominant predictor, confirming that recent price momentum drives short-term prices more than seasonality.

### What's Next for ML1

In Phase III, a key focus will be adjusting this model to support country selection beyond Germany alone. I also plan to improve the overall model performance to bring up the R² score, which currently sits at 0.265, by adding gas storage and electricity demand as features once AGSI access is approved. The forecast function will also be connected to the frontend and final MAE and RMSE scores will be updated once the model completes its latest run.

## Belgium Update

Today was a key highlight of my time in Belgium so far. We took a day trip to Ghent, and visited St. Bavo's Cathedral, where we took part in a VR experience that brought the famous Mystic Lamb to life in a way that felt both immersive way. It was a fascinating way to learn about one of the most celebrated works of art up close. After visiting the cathedral, we wandered through Ghent's streets, admiring the medieval architecture, canals, and atmosphere.