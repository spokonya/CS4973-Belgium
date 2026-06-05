---
title: "Blog Post 3"
date: 2026-06-05
draft: false
description: "Anjali Blog Post3"
slug: "anjali-blog3"
tags: ["ds3000", "Python", "Strousberg", "Belgium"]
authors:
  - "Anjali_Patel"
showAuthorsBadges : false
---

## Data Cleaning and Visulization
![Straousberg/France Trip!](https://www.leshuttle.com/media-library/media/leshuttle/articles/france/lyon/adobestock_320312981.jpeg?width=1536&height=824&rmode=crop&format=webp&quality=60&v=202504230934)

### Deliverable 3 Update
For this phase, I focused on the updates for Model 2, our winter storage stress classifier. I mainly focused on refining as well as feature explanations and the issues we found during model exploration. I also added a proper section justifying the 30% stress threshold, as it is backed by EU policy regulation since 2022 and requiring member states to mandate 90% storage by November 1st. I also decided to continue with only logistic regression as it has the most meaningful results and R2 of 0.66. The biggest thing I caught was a standardization error in our feature importance analysis. Our features have very different scales, so ranking the raw coefficients reflected units rather than influence. I rebuilt the model as a StandardScaler and LogisticRegression pipeline with scaling done inside each CV fold, then re-ran the analysis. I also corrected our cross-validation description, since StratifiedKFold stratifies on the class label rather than by country as we had written, and tested GroupKFold by country before dropping it as too unstable with only 17 groups.

On the frontend, I deployed the trained models into our Streamlit app and integrated the interaction with the pages the CS students had designed. The Gas Storage Risk page lets users pick a country and adjust three sliders, one per model feature, defaulting to that country's most recent AGSI values, the model re-predicts live as they drag, showing risk probability, with their scenario plotted against ten years of real winters. The Country Comparison page ranks all 17 countries as a risk leaderboard based on their latest pre-winter conditions, and Country Snapshot shows each country's ten-year storage history. I also re-saved the final model as the full standardized pipeline with joblib.

### Strousberg/France

I really enjoyed the trip to Strousberg and Luxembourg this weekend! It was a great way to take a break and explore new cities. Strousberg was absolutely beautiful and I loved the mix of historical buildings and also a modernized city. My favorite part was the bridges over the water and the lock bridge. Additionally, we went to Luxembourg which was my first time in the country and I enjoyed seeing all the amazing views and enjoy the nice hotel. 