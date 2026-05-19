---
title: "Project - Phase I"
date: 2026-05-18
draft: false
description: "EU Energy Security Index: Phase I team update"
slug: "phase1post"
tags: ["project", "phase1", "energy", "Personas"]
showAuthorsBadges: false
showTableOfContents: true
---

# Phase I — Team Update

We are a group of students participating in a study abroad program in Belgium. In this post, we'll be covering our Phase I progress on the EU Energy Security Index. It is a dashboard designed to help users understand energy security risks across European Union countries. We'll walk through our project concept, the problem we're solving, and the foundation we've built so far.

## At a glance

{{< glance
  project="EU Energy Security Index"
  problem="EU countries lack a unified, accessible view of energy security risk and import dependence"
  solution="Per-country dashboard for imports, gas storage, electricity prices, and supply-shock risk"
  users="Households, Journalists, Policy analysts"
>}}

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

## User personas


### Persona 1 — Household Owner

{{< persona name="Lena Müller" role="Household Owner" age="30" location="Hamburg, Germany" initials="LM" type="household" photo="images/team/Lena.jpg" >}}
Her electric bill is about 180 euros per month. She wants to anticipate bill changes and decide when to lock in a fixed-rate contract. It jumped roughly 60% during winter 2022 and still moves with the news cycle. She has no economics background, uses apps comfortably, and does not fully trust the contract offers her utility sends every few months.

**User stories**

1. As Lena, I want to see how electricity prices in Germany are forecast to move over the next 30 days, so I can decide whether to lock in a fixed-rate contract now or wait.
2. As Lena, I want to see how current gas storage levels compare to recent winters, so I can judge whether the upcoming winter is likely to be a high-bill season.
3. As Lena, I want a plain-language summary of the forecast, so I can understand what is driving the prediction without an economics background.
4. As Lena, I want to compare Germany's situation to its neighbors, so I can tell whether price pressure is local or EU-wide.
{{< /persona >}}

### Persona 2 — Energy Journalist

{{< persona name="Marco Frite" role="Energy Journalist" age="30" location="Brussels, Belgium" initials="MF" type="journalist" photo="images/team/marco.jpg" >}}
He writes two to three articles a week. He needs country-level energy facts quickly and wants to spot EU-wide outliers for reporting. on gas markets, electricity prices, and policy decisions. Today he jumps between ENTSO-E, gas storage portals, and Eurostat—each with its own format.

**User stories**

1. As Marco, I want current electricity prices, gas storage, and import dependence for each EU country in one place, so I can gather facts without bouncing between five sources.
2. As Marco, I want to compare a country's indicators to its neighbors, so I know which countries deserve attention in a story.
3. As Marco, I want a country snapshot I can screenshot or export, so figures in my article stay accurate without manual copy-paste.
4. As Marco, I want to compare today's prices and risk score to the same date in prior years, so I can frame whether the moment is historically unusual.
{{< /persona >}}

### Persona 3 — Policy Analyst

{{< persona name="Sofia Anderson" role="Policy Analyst" age="47" location="Brussels, Belgium" initials="SA" type="policy" photo="images/team/sofia.jpg" >}}
Senior researcher writing memos. She needs to identify vulnerable member states and model supply-shock scenarios for briefing memos. for MEPs and national energy councils. Comfortable with models and needs exportable, defensible comparisons—not headline noise.

**User stories**

1. As Sofia, I want to rank EU countries by winter stress risk, so I can prioritize which member states need attention in briefings.
2. As Sofia, I want to adjust gas storage and weather scenarios by country, so I can model different supply-shock conditions for policymakers.
3. As Sofia, I want to see which features drive each country's risk score, so I can tie recommendations to evidence.
4. As Sofia, I want to export country comparison data, so I can drop tables into memos and presentations without retyping numbers.
{{< /persona >}}

## Data Sources

### 1. ENTSO-E: Electricity transparency

{{< datasource name="ENTSO-E Transparency Platform" link="https://transparency.entsoe.eu/" use="Real-time and predictive electricity data: day-ahead prices, demand forecasts, fuel-specific generation, cross-border flows, and system unavailabilities across EU bidding zones." >}}

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
{{< /datasource >}}

### 2. GIE (AGSI): Gas storage & LNG

{{< datasource name="AGSI (Gas Infrastructure Europe)" link="https://agsi.gie.eu/" use="Underground gas storage and LNG terminal data: storage levels vs. capacity, injection/withdrawal rates, LNG inventory, send-out rates, and slot bookings." >}}

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
{{< /datasource >}}

### 3. Eurostat: Energy statistics

{{< datasource name="Eurostat databrowser" link="http://ec.europa.eu/eurostat/databrowser/" use="Harmonized cross-EU structural and historical indicators: household energy prices, import dependence, energy balances, and renewables share. No API key required; datasets update twice daily (11:00 and 23:00 Brussels time)." >}}

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
{{< /datasource >}}

### 4. Open-Meteo: Weather

{{< datasource name="Open-Meteo" link="https://open-meteo.com/" use="Free weather forecasts and historical weather (temperature, wind speed, solar radiation) as scenario inputs for price and risk models. No API key required." >}}

**Persona relevance**

- **Household (Lena):** Weather forecasts are a leading indicator of what potential energy bills could look like, especially during winter.
- **Policy analyst (Sofia):** Wind speed and solar radiation are variables she can use in supply-shock scenarios.
{{< /datasource >}}

## Individual reflections

{{< members >}}
{{< member name="Ari Spokony" role="Data sources" href="/ari_spokony/blog1/" initials="AS" photo="images/team/ari.jpg" >}}
{{< member name="Bobby Bress" role="Project description" href="/bobby_bress/bress-belgium-blog/" initials="BB" photo="images/team/bobby.jpg" >}}
{{< member name="Anjali Patel" role="Personas & user stories" href="/anjali_patel/anjali-intro/" initials="AP" photo="images/team/anjali.jpg" >}}
{{< member name="Rayna Patel" role="Personas & user stories" href="/rayna_patel/rayna-intro/" initials="RP" photo="images/team/rayna.jpg" >}}
{{< /members >}}

