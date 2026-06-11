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

<!--
OUTLINE — fill in each section below. Notes in HTML comments describe what
each section should cover; replace the placeholder prose as you write.
-->

## Project Recap

<!--
- One-paragraph reminder of what Zeus (the EU Energy Security Index) is and the
  problem it solves.
- Recap the three personas and the core question each one brings to the product:
  - Household Owner
  - Journalist
  - Energy Trader
- State the Phase IV goal: what "done" looks like for this phase and how it
  builds on Phases I–III.
-->

Energy security matters to everyone, but the data needed to understand it is scattered across fragmented official sources: ENTSO-E for electricity, GIE for gas storage, and Eurostat for the wider picture. Zeus pulls those signals into a single platform and turns raw European energy data into country-level insight, including day-ahead electricity price forecasts, gas-storage stress relative to historical norms, import dependence, and how each country compares to its neighbors. The goal is to surface the conditions that precede a supply shock, not just report one after it happens.

Zeus is built around three personas, each with its own view of the same platform:

- **Household Owner:** wants to understand their bill and where electricity prices are headed in their country.
- **Journalist:** needs defensible, country-level evidence to back stories on gas and electricity market stress, including which countries face winter storage risk.
- **Energy Trader:** wants a 30-day price path to trade against, a watchlist, and a journal to log forecast calls and review how they resolved.

Two machine-learning models power the analytics:

- **ML1:** 30-day electricity price forecast across 15 EU countries feeding both the Household Owner and Energy Trader personas.
- **ML2:** winter gas-storage stress classifier powering the Journalist risk pages.

In Phase 4, we focused on finalizing the implementation of the website. We reworked our interface design based on the feedback we got, connected the database to the Energy Trader persona so they can log their trade notes, and finalized our two machine learning models to make them as accurate as we could.





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

## ML Models — Fundamental Understanding

<!--
For EACH model: task, input features, the underlying math, and why this model
class is the right fit for the problem.
-->

### ML1 — Day-Ahead Price Forecast (Autoregressive Linear Regression)

_TODO: task (forecast daily avg price), features (lags, rolling means, calendar),
the linear-regression math, why autoregressive linear regression fits._

### ML2 — Winter Storage-Risk Classifier (Logistic Regression)

_TODO: task (predict storage_stress before winter), features (storage_at_start,
trend, volatility), the logistic-regression math, why logistic regression fits._

## ML Implementation in the Architecture

<!--
- Show how the models flow through the system: train → seed-to-DB → serve-with-NumPy.
- Training scripts produce weights/scaler/intercept → seeded into the model tables.
- Flask routes load those parameters and run inference with plain NumPy (no heavy
  ML runtime in the request path).
- List the relevant API routes and the frontend clients that call them.
-->

_TODO: train → seed-to-DB → serve-with-NumPy pipeline, the Flask routes, and the Streamlit clients that consume them._

## Model Verification — Assumptions & Predictive Checks

<!--
CREDIT-EARNING SECTION. This work is done OUTSIDE the web app (notebooks/scripts),
not served in the UI.
- State the modeling assumptions for each model and test them.
- Predictive checks: held-out performance, residual analysis, calibration, etc.
- Be explicit that these checks live outside the deployed app.
-->

_TODO: assumption checks and predictive validation for ML1 and ML2 (performed outside the web app)._

## Reflection & Next Steps

<!--
- What went well, what was hard, what you'd do differently.
- Honest limitations of the current models/architecture.
- Concrete next steps beyond this phase.
-->

_TODO: reflection on Phase IV and next steps._

## Individual posts

<!--
Keep consistent with earlier phases — link each teammate's individual Phase IV update.
-->
{{< members >}}
{{< member name="Ari Spokony" role="" href="/ari_spokony/" initials="AS" photo="images/team/ari.jpg" >}}
{{< member name="Bobby Bress" role="" href="/bobby_bress/" initials="BB" photo="images/team/bobby.jpg" >}}
{{< member name="Anjali Patel" role="" href="/anjali_patel/" initials="AP" photo="images/team/anjali.jpg" >}}
{{< member name="Rayna Patel" role="" href="/rayna_patel/" initials="RP" photo="images/team/rayna.jpg" >}}
{{< /members >}}
