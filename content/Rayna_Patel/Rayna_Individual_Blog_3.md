---
title: "Week 3 Blog Post"
date: 2026-06-04
draft: false
description: "Rayna's Third Blog Post"
slug: "rayna-week3"
tags: ["DS3000", "Leuven", "Linear Algebra", "Belgium"]
authors:
  - "Rayna_Patel"
showAuthorsBadges : false
---

# Phase 3 Blog Post

## My Contributions to Phase III
My contribution to Phase III was implementing feedback and improving the model through feature engineering, the addition of 14 EU countries, and creating a single shared model with one-hot encoded country features, overall increasing R² from 0.265 to 0.608.

### Model Updates

On the feature engineering side, I removed `year`, switched lag features from selecting a few lags to consecutive lags (1 through 7), replaced raw `month` and `dayofweek` numbers with one-hot encoding and `drop_first=True`, and added `rolling_7d_std` and `price_vs_7d_avg` to capture price volatility and comparison relative to recent trends. These changes together pushed R² from 0.265 to 0.320.

The biggest change to ML1 this phase was expanding from Germany alone to 15 EU countries. I pulled data for all 15 countries from the ENTSO-E API, combined them into a single dataset of 29,565 rows, and retrained the model with country as a one-hot encoded feature. This change alone pushed R² from 0.320 to 0.608.

### EDA and Visualizations

I added a scatter plot of electricity price vs day of year colored by year to explore whether a quadratic, cubic, or square root seasonal relationship existed in the data. The chart showed each year has a distinctly different shape, with 2022 being an outlier compared to other years which trend up, down, or stay flat, confirming that no single mathematical curve fits the seasonal pattern.

I also added residual plots to validate the linear regression model assumptions. The residuals are roughly randomly distributed around zero with no apparent biases.

### What's Next for ML1

- Connect the model to the frontend in Phase 4

## Belgium Update

This week we explored Strasbourg and visited Eurostat in Luxembourg.Also, in Brussels, we had a guest speaker session on EU digital and industrial policy which is currently my favorite guest speaker of the entire program. The speaker was very engaging and the meeting was more discussion based which allowed for us to hear different perspectives from one another. We also took part in a chocolate making workshop. Regardless of being allergic, this was one of the most fun afternoons of the trip.

{{< figure src="https://upload.wikimedia.org/wikipedia/commons/thumb/6/65/Gerwerstub_1572%2C_maison_des_tanneurs%2C_Strasbourg_%282014%29.jpg/1920px-Gerwerstub_1572%2C_maison_des_tanneurs%2C_Strasbourg_%282014%29.jpg?_=20230622195150" alt="Strasbourg" caption="Strasbourg — Wikimedia Commons" >}}