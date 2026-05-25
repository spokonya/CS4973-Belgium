---
title: "Project - Phase II"
date: 2026-05-25
draft: false
description: "EU Energy Security Index: Phase II team update"
slug: "phase2post"
tags: ["project", "phase2", "energy", "Personas"]
showAuthorsBadges: false
showTableOfContents: true
---

# Phase II — Team Update

Open with a short paragraph: one sentence on what the EU Energy Security Index is,
then a few sentences summarizing what this phase delivered — you pulled real data and
cleaned it, explored it with summary statistics and charts, built your first
machine-learning model on real data, drafted the app's data model and database schema,
and made initial UI wireframes.
 
## At a glance
{{< glance
project="EU Energy Security Index"
problem="EU countries lack a unified view of energy security risk and import dependence"
solution="Per-country dashboard for imports, gas storage, electricity prices, and supply-shock risk"
users="Households, Journalists, Policy analysts"
>}}
 
## Updates since Phase I
Say clearly whether your plan changed, and address each of these even if the answer is
"no change" (the graders specifically ask):
 
- Did any **persona** change, or did you add or edit any **user stories**?
- Did you find or confirm any **new datasets** for the ML? Mention that you secured
  **ENTSO-E** access, that **Eurostat** and **Open-Meteo** are your keyless real-data
  sources for now, and that **AGSI** (gas storage) access is still pending.
- Any change in **scope or modeling approach** worth noting.
## Real data curation
Walk through your data pipeline from raw API to clean files:
 
### ML 1
- **Where it came from and how:** which sources you pulled, that you pulled via API into
  Python (e.g. as JSON).
- **Cleaning (pandas/numpy):** the steps you ran — matching country codes (Eurostat uses
  "EL" for Greece → "GR"), handling missing values, lining up time periods, turning daily
  weather into yearly features.
- **Saved to CSV** so you don't have to re-run the API every time.
- **Exploratory analysis:** report your **single-variable** summary stats (averages,
  spread) and your **two-variable** stats (how things relate — e.g. electricity price
  rises with gas price and import dependence, falls with renewables share).
### ML 2
- **Where it came from and how:** which sources you pulled, that you pulled via API into
  Python (e.g. as JSON).
- **Cleaning (pandas/numpy):** the steps you ran — matching country codes (Eurostat uses
  "EL" for Greece → "GR"), handling missing values, lining up time periods, turning daily
  weather into yearly features.
- **Saved to CSV** so you don't have to re-run the API every time.
- **Exploratory analysis:** report your **single-variable** summary stats (averages,
  spread) and your **two-variable** stats (how things relate — e.g. electricity price
  rises with gas price and import dependence, falls with renewables share).
## Data visualizations
### ML 1
Show each chart, and for every one write two things: **why** you chose that chart type,
and **what it tells you about one of your Phase I big questions.** Your charts and the
question each answers:
 
- Import-dependence ranking → which countries rely most on imports.
- Price drivers (scatter) → what pushes household prices up.
- Correlation heatmap → which factors move together.
- Storage vs. historical norm → is this winter unusual? (Lena)
- Price distribution → where one country sits in the EU range. (Marco)
- PCA country map → which countries are most similar.
- Risk ranking → countries ordered from most to least at-risk. (Sofia)
### ML 2
Show each chart, and for every one write two things: **why** you chose that chart type,
and **what it tells you about one of your Phase I big questions.** Your charts and the
question each answers:
 
- Import-dependence ranking → which countries rely most on imports.
- Price drivers (scatter) → what pushes household prices up.
- Correlation heatmap → which factors move together.
- Storage vs. historical norm → is this winter unusual? (Lena)
- Price distribution → where one country sits in the EU range. (Marco)
- PCA country map → which countries are most similar.
- Risk ranking → countries ordered from most to least at-risk. (Sofia)
## Machine learning
### ML 1
Describe your modeling work and be honest about its state:
 
- **What you built:** at least one supervised model trained on real cleaned data — your
  high-price classifier — and note it's deliberately **more complex than a neighborhood
  (kNN) model**, which the assignment requires. kNN is used only as a baseline and for the
  "similar countries" feature. Also mention PCA and the risk ranking.
- **The plan for two models:** you'll ultimately have at least two models on real data;
  parts not yet on real data (like the day-ahead price forecast) currently use simulated
  data, which is allowed for this phase.
- **How well it worked:** your results, using the fair "leave-one-country-out" test, and
  what the scores mean.
- **Difficulties:** e.g. API approval wait times, no ready-made "supply-shock" label, etc.
- **What's left:** run everything on real data, pick the final model, add gas-storage
  features once AGSI is approved, build the shock classifier in Phase III.
### ML 2
Describe your modeling work and be honest about its state:
 
- **What you built:** at least one supervised model trained on real cleaned data — your
  high-price classifier — and note it's deliberately **more complex than a neighborhood
  (kNN) model**, which the assignment requires. kNN is used only as a baseline and for the
  "similar countries" feature. Also mention PCA and the risk ranking.
- **The plan for two models:** you'll ultimately have at least two models on real data;
  parts not yet on real data (like the day-ahead price forecast) currently use simulated
  data, which is allowed for this phase.
- **How well it worked:** your results, using the fair "leave-one-country-out" test, and
  what the scores mean.
- **Difficulties:** e.g. API approval wait times, no ready-made "supply-shock" label, etc.
- **What's left:** run everything on real data, pick the final model, add gas-storage
  features once AGSI is approved, build the shock classifier in Phase III.
## Data model — ER diagrams  (CS teammates)
For **each persona**, include a localized data model (an ER diagram) built from the data
their user stories need. Then show one **global ER diagram** that combines them into the
full app. Make sure it includes a place to store the cleaned, pre-processed features used
to train and test the ML models.

 
## Database — first-pass schema (DDL)  (CS teammates)
Include a first draft of the SQL `CREATE TABLE` statements for the global data model,
including the table that holds the ML feature data.
 
## UI wireframes
Show **two screens for each persona** plus **one home/landing page**. Low-fidelity is
fine (hand-drawn is OK). Do **not** include login, password-reset, or account-creation
screens.
 
## Individual posts
Each teammate writes their own post covering: which charts or data-pipeline steps they
personally worked on, which localized data model they were the point person for, and
(optional) a short note about anything notable from the week.
{{< members >}}
{{< member name="Ari Spokony" role="" href="/ari_spokony/" initials="AS" photo="images/team/ari.jpg" >}}
{{< member name="Bobby Bress" role="" href="/bobby_bress/" initials="BB" photo="images/team/bobby.jpg" >}}
{{< member name="Anjali Patel" role="" href="/anjali_patel/" initials="AP" photo="images/team/anjali.jpg" >}}
{{< member name="Rayna Patel" role="" href="/rayna_patel/" initials="RP" photo="images/team/rayna.jpg" >}}
{{< /members >}}
 
