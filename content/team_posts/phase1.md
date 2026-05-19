---
title: "Project - Phase I"
date: 2026-05-18
draft: false
description: "EU Energy Security Index: Phase I team update"
slug: "phase1post"
tags: ["project", "phase1", "energy", "Personas"]
authors:
  - "Ari_Spokony"
  - "Bobby_Bress"
  - "Anjali_Patel"
  - "Rayna_Patel"
showAuthorsBadges: false
---

# Phase I — Team Update

We are a group of students participating in a study abroad program in Belgium. In this post, we'll be covering our Phase I progress on the EU Energy Security Index. It is a dashboard designed to help users understand energy security risks across European Union countries. We'll walk through our project concept, the problem we're solving, and the foundation we've built so far.

## At a glance


|                   |                                                             |
| ----------------- | ----------------------------------------------------------- |
| **Project** | EU Energy Security Index |
| **Problem** | EU countries lack a unified, accessible view of their energy security risk and import dependence|
| **Solution** | An interactive per-country dashboard that tracks energy imports, gas storage, electricity prices, and supply shock risk across the EU  |
| **Primary users** | Households · Journalists · Policy analysts |

  
---

## Project description

Energy security is extremely important for countries across the world. Having sufficient information regarding energy on a country by country view could help make critical policy decisions based on energy. A dashboard that could make it easy to see information on different European Union countries could display which countries are most dependent on energy imports and which ones are at the greatest risk in case of an energy supply shock. This could help policy analysts and government officials in making policy decisions regarding energy security for their country. It could also assist journalists writing about energy policy and citizens looking to understand their energy bills. We want to build this for our project and include information like each country's dependence on others for energy imports, their current gas storage level vs historical norms, an electricity price forecast, a risk score in case of a supply shock, and how each country compares to its neighbors. We would use machine learning for predicting the electricity prices of the countries and classifying each country’s vulnerability to a supply shock.

### Why now

The Russian invasion of Ukraine in 2022 exposed how vulnerable European countries are to energy supply disruptions. It triggered an energy crisis that greatly increased electricity and gas prices. Since then, the EU has attempted to diversify its energy sources and build up gas reserves. But, there is still a great risk of future shocks. It is a critical time for people to have access to clear energy security data to help aid in this.

### What we're building


- **Energy import dependence tracker** — visualize how reliant each EU country is on external sources for gas, oil, and electricity
- **Gas storage monitor** — compare each country's current storage levels against historical averages and EU benchmarks
- **Electricity price forecast** — machine learning model predicting short term electricity prices by country
- **Supply shock risk score** — ML classifier that rates each country's vulnerability to an energy supply disruption
- **Country comparison tool** — side by side view of how a country stacks up against its neighbors across all key metrics

### Planned machine learning


| Model                     | Purpose                                  | Inputs (draft)                                  | Output                     |
| ------------------------- | ---------------------------------------- | ----------------------------------------------- | -------------------------- |
| **ML #1: Price forecast** | Short-horizon electricity price forecast | *[e.g. historical prices, demand, gas storage]* | *[e.g. 30-day outlook]*    |
| **ML #2: Winter stress**  | Supply-shock vulnerability classifier    | *[e.g. import dependence, storage, weather]*    | *[Per-country risk score]* |


---

## User personas


### Persona 1 — Household Owner

| | |
| :--- | :--- |
| **Name** | *Lena Müller* |
| **Role** | *Household Owner* |
| **Age** | *30* |
| **Location** | *Hamburg, Germany* |
| **Headshot** | <img src="https://media.easy-peasy.ai/27feb2bb-aeb4-4a83-9fb6-8f3f2a15885e/540357c1-24ab-4ab9-942d-e00789905eb2_medium.webp" alt="Lena" width="150"> |
| **Description** | *Her current electric bill is 180 euros per month. Her bill jumped 60% in winter 2022 and has since been changing depending on the electric news. She has no economics or finance background. She's comfortable with apps, but doesn't trust the contract offers that her utility company send every few months.* |
| **Goals** | *Lena would like to be able to anticipate whether energy bills are about to rise and decide whether to lock in a fixed-rate contract.* |

**User stories**

1. As Lena, I would like see how electricity prices in Germany are forecasted to move over the next 30 days, so that I can decide whether to lock in a fixed-rate contract now or wait.

2. As Lena, I want to see how current gas storage levels compare to recent winters, so that I can judge whether the upcoming winter is likely to be a high-bill one.

3. As Lena, I want a plain-language summary explaining the forecast, so that I can understand what's driving the prediction without needing an economics background.

4. As Lena, I want to compare Germany's situation to its neighbors, so that I can understand whether the price pressure is local or EU-wide.

---

### Persona 2: Energy journalist


|                 |                      |
| --------------- | -------------------- |
| **Name**        | *[e.g. Marco Frite]* |
| **Role**        | Energy journalist    |
| **Description** | *[…]*                |
| **Goals**       | *[…]*                |


**User stories**

1. *[…]*
2. *[…]*
3. *[…]*
4. *[…]*
5. *[optional]*

---

### Persona 3: Policy analyst


|                 |                         |
| --------------- | ----------------------- |
| **Name**        | *[e.g. Sofia Anderson]* |
| **Role**        | Policy analyst          |
| **Description** | *[…]*                   |
| **Goals**       | *[…]*                   |


**User stories**

1. *[…]*
2. *[…]*
3. *[…]*
4. *[…]*
5. *[optional]*

---

## Data sources

### 1. ENTSO-E: Electricity transparency


|                        |                                                                                                                                                                                                                                                    |
| ---------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Link**               | [ENTSO-E Transparency Platform](https://transparency.entsoe.eu/?appState=%7B%22sa%22%3A%5B%5D%2C%22st%22%3A%22BZN%22%2C%22mm%22%3Atrue%2C%22ma%22%3Afalse%2C%22sp%22%3A%22CLOSED%22%2C%22dt%22%3Anull%2C%22df%22%3Anull%2C%22tz%22%3A%22CET%22%7D) |
| **What we use it for** | Real-time and predictive electricity data: day-ahead prices, demand forecasts, fuel-specific generation, cross-border flows, and system unavailabilities across EU bidding zones.                                                                  |


**API access**

1. Create an account on the transparency platform.
2. Email [transparency@entsoe.eu](mailto:transparency@entsoe.eu) with the subject `Restful API access` and the body containing the email you used to sign up (approval typically takes 1–3 days).
3. After approval, generate a security key from your account ([directions](https://transparencyplatform.zendesk.com/hc/en-us/articles/12845911031188-How-to-get-security-token)).

**Integration notes**

- Python helper: [entsoe-py](https://github.com/EnergieID/entsoe-py) *(still evaluating for our pipeline)*
- **Manual of Procedures (MoP):** [transparencyplatform.zendesk.com](https://transparencyplatform.zendesk.com), the authoritative reference for data items and publication standards.
- **EIC directory:** [Energy Identification Codes](https://www.entsoe.eu/data/energy-identification-codes-eic/), the canonical list of bidding zone and control area codes for Postgres reference tables.
- **REST API guide:** available via the platform Help page (parameters and document types).
- **Postman collection:** [interactive API docs](https://documenter.getpostman.com/view/7009892/2s93JtP3F6) for query testing before development.
- **Backup:** published CSVs via the [File Library Guide](https://transparencyplatform.zendesk.com/hc/en-us/articles/35960137882129-File-Library-Guide).

**Persona relevance**

- **Household (Lena):** Day-ahead pricing and demand forecasts power the 30-day outlook she uses to decide whether to lock in a fixed-rate contract.
- **Journalist (Marco):** Unified view of current prices and cross-border flows across EU bidding zones for country-level comparisons and article snapshots.
- **Policy analyst (Sofia):** Fuel-specific generation and system unavailability data feed vulnerability metrics and risk scores in policy briefings.

---

### 2. GIE (AGSI): Gas storage & LNG


|                        |                                                                                                                                                           |
| ---------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Link**               | [AGSI (Gas Infrastructure Europe)](https://agsi.gie.eu/)                                                                                                  |
| **What we use it for** | Underground gas storage and LNG terminal data: storage levels vs. capacity, injection/withdrawal rates, LNG inventory, send-out rates, and slot bookings. |


**AGSI (Aggregated Gas Storage Inventory)**

Daily data on underground gas storage in Europe and neighboring countries (UK, Ukraine). Per country, company, and facility: current gas in storage, working gas volume (total capacity), injection rate, withdrawal rate, and percent full. Historical data back to 2011.

**LNG (liquefied natural gas)**

Same structure for LNG terminals (seaport import points): LNG inventory, send-out rates, and slot bookings. Post-2022, LNG replaced a large share of Russian pipeline gas, so this source is critical for supply-shock context.

**API access**

Request API access from the [account page](https://agsi.gie.eu/account).

**Persona relevance**

- **Household (Lena):** Real-time storage vs. five-year average helps judge whether an upcoming winter is likely to mean higher bills.
- **Journalist (Marco):** Country-level storage and LNG send-out rates highlight outliers and EU-wide comparisons for reporting.
- **Policy analyst (Sofia):** Storage trajectories are a core input to the winter stress risk score and the primary variable for supply-shock scenario modeling.

---

### 3. Eurostat: Energy statistics


|                        |                                                                                                                                                            |
| ---------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Link**               | [Eurostat databrowser](http://ec.europa.eu/eurostat/databrowser/)                                                                                          |
| **What we use it for** | Harmonized cross-EU structural and historical indicators: household energy prices, import dependence, energy balances, and renewables share.               |
| **Access**             | No API key required; responses are JSON. Most recent published reference period: **2025S2**. Datasets update twice daily at 11:00 and 23:00 Brussels time. |


**Datasets we plan to use**


| Code          | Description                  |
| ------------- | ---------------------------- |
| `nrg_pc_204`  | Household electricity prices |
| `nrg_ti_eh`   | Electricity imports          |
| `nrg_ti_gas`  | Gas imports                  |
| `Nrg_bal_c`   | Annual energy balance        |
| `nrg_pc_202`  | Household gas prices         |
| `nrg_ind_ren` | Renewables share             |
| `sdg_07_30`   | Import dependency            |
| `nrg_cb_e`    | Monthly electricity balance  |


**Persona relevance**

- **Household (Lena):** `nrg_pc_204` and `nrg_pc_202` provide a historical baseline to see whether her bill trajectory is unusual compared to other households in Germany.
- **Journalist (Marco):** Harmonized cross-EU definitions let him compare countries without reconciling differing methodologies.
- **Policy analyst (Sofia):** Pre-computed indicators such as `sdg_07_30` (import dependency) and `nrg_ind_ren` (renewables share) replace manual Excel compilation and map directly to structural features in risk rankings.

---

### 4. Open-Meteo: Weather


|                        |                                                                                                                                        |
| ---------------------- | -------------------------------------------------------------------------------------------------------------------------------------- |
| **Link**               | [open-meteo.com](https://open-meteo.com/)                                                                                              |
| **What we use it for** | Free weather forecasts and historical weather (temperature, wind speed, solar radiation) as scenario inputs for price and risk models. |
| **Access**             | No API key required ([read the docs](https://open-meteo.com/en/docs)).                                                                 |


**Persona relevance**

- **Household (Lena):** Weather forecasts are a leading indicator of what potential energy bills could look like, especially during winter.
- **Policy analyst (Sofia):** Wind speed and solar radiation are variables she can use in supply-shock scenarios.

---

## Phase I deliverables checklist


| Deliverable                                 | Owner(s)      | Status                |
| ------------------------------------------- | ------------- | --------------------- |
| Project description (150–250 words)         | Bobby         | Done                |
| User personas (3) + user stories (4–5 each) | Rayna, Anjali | Done                |
| Data sources (2+)                           | Ari           | Done                 |
| Team blog post (this page)                  | All           | Done                 |
| Individual blog posts                       | All           | Done                  |


---

## Individual reflections




| Team member  | Post                      |
| ------------ | ------------------------- |
| Ari Spokony  | *[link or "coming soon"]* |
| Bobby Bress  | *[link or "coming soon"]* |
| Anjali Patel | *[link or "coming soon"]* |
| Rayna Patel  | *[link or "coming soon"]* |


---

## Next steps



- *Finalize Data sources*
- *Choose which data is relevant to our project*
- *Proof of Concept*

