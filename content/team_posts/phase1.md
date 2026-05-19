---
title: "Project - Phase I"
date: 2026-05-18
draft: false
description: "EU Energy Security Index — Phase I team update"
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

<!-- Short intro: team name + semester/program + sentence on what this post covers -->

# Phase I — Team Update

We are a group of students participating in a study abroad program in Belgium. In this post, we'll be covering our Phase I progress on the EU Energy Security Index. It is a dashboard designed to help users understand energy security risks across European Union countries. We'll walk through our project concept, the problem we're solving, and the foundation we've built so far.

---

## At a glance

| | |
| :--- | :--- |
| **Project** | EU Energy Security Index |
| **Problem** | EU countries lack a unified, accessible view of their energy security risk and import dependence|
| **Solution** | An interactive per-country dashboard that tracks energy imports, gas storage, electricity prices, and supply shock risk across the EU  |
| **Primary users** | Households · Journalists · Policy analysts |

---

## Project description

<!-- Owner: Bobby | Target: 150–250 words | Source: ideas.md → "Project Description" -->

Energy security is extremely important for countries across the world. Having sufficient information regarding energy on a country by country view could help make critical policy decisions based on energy. A dashboard that could make it easy to see information on different European Union countries could display which countries are most dependent on energy imports and which ones are at the greatest risk in case of an energy supply shock. This could help policy analysts and government officials in making policy decisions regarding energy security for their country. It could also assist journalists writing about energy policy and citizens looking to understand their energy bills. We want to build this for our project and include information like each country's dependence on others for energy imports, their current gas storage level vs historical norms, an electricity price forecast, a risk score in case of a supply shock, and how each country compares to its neighbors. We would use machine learning for predicting the electricity prices of the countries and classifying each country’s vulnerability to a supply shock.

### Why now

<!-- Optional: 1 short paragraph on post-2022 energy security context -->

The Russian invasion of Ukraine in 2022 exposed how vulnerable European countries are to energy supply disruptions. It triggered an energy crisis that greatly increased electricity and gas prices. Since then, the EU has attempted to diversify its energy sources and build up gas reserves. But, there is still a great risk of future shocks. It is a critical time for people to have access to clear energy security data to help aid in this.

### What we're building

<!-- Bullet or short list: dashboard views + scenario inputs -->

- **Energy import dependence tracker** — visualize how reliant each EU country is on external sources for gas, oil, and electricity
- **Gas storage monitor** — compare each country's current storage levels against historical averages and EU benchmarks
- **Electricity price forecast** — machine learning model predicting short term electricity prices by country
- **Supply shock risk score** — ML classifier that rates each country's vulnerability to an energy supply disruption
- **Country comparison tool** — side by side view of how a country stacks up against its neighbors across all key metrics

### Planned machine learning

| Model | Purpose | Inputs (draft) | Output |
| :--- | :--- | :--- | :--- |
| **ML #1 — Price forecast** | Short-horizon electricity price forecast | *[e.g. historical prices, demand, gas storage]* | *[e.g. 30-day outlook]* |
| **ML #2 — Winter stress** | Supply-shock vulnerability classifier | *[e.g. import dependence, storage, weather]* | *[Per-country risk score]* |

---

## User personas

<!-- Owners: Rayna & Anjali | 3 personas × 4–5 user stories each | Source: ideas.md → "User Personas" -->

### Persona 1 — Household owner

| | |
| :--- | :--- |
| **Name** | *[e.g. Lena Müller]* |
| **Role** | Household owner |
| **Description** | *[Age, location, bill context, background, tech comfort]* |
| **Goals** | *[Bullet or short paragraph]* |

**User stories**

1. *[As a …, I want …, so that …]*
2. *[…]*
3. *[…]*
4. *[…]*
5. *[optional]*

---

### Persona 2 — Energy journalist

| | |
| :--- | :--- |
| **Name** | *[e.g. Marco Frite]* |
| **Role** | Energy journalist |
| **Description** | *[…]* |
| **Goals** | *[…]* |

**User stories**

1. *[…]*
2. *[…]*
3. *[…]*
4. *[…]*
5. *[optional]*

---

### Persona 3 — Policy analyst

| | |
| :--- | :--- |
| **Name** | *[e.g. Sofia Anderson]* |
| **Role** | Policy analyst |
| **Description** | *[…]* |
| **Goals** | *[…]* |

**User stories**

1. *[…]*
2. *[…]*
3. *[…]*
4. *[…]*
5. *[optional]*

---

## Data sources

<!-- Owner: Ari | Minimum 2 sources (we have 4) | Source: ideas.md → "Data Sources" -->

### 1. ENTSO-E — Electricity transparency

| | |
| :--- | :--- |
| **Link** | *[URL]* |
| **What we use it for** | *[Day-ahead prices, demand, generation, cross-border flows, etc.]* |
| **Access** | *[Account + API token steps, or note if pending]* |
| **Integration notes** | *[e.g. entsoe-py, EIC codes, Postman collection]* |

**Persona relevance**

- **Household:** *[…]*
- **Journalist:** *[…]*
- **Policy analyst:** *[…]*

---

### 2. GIE (AGSI) — Gas storage & LNG

| | |
| :--- | :--- |
| **Link** | *[URL]* |
| **What we use it for** | *[Storage levels, % full, injection/withdrawal, LNG send-out]* |
| **Access** | *[API request status]* |

**Persona relevance**

- **Household:** *[…]*
- **Journalist:** *[…]*
- **Policy analyst:** *[…]*

---

### 3. Eurostat — Energy statistics

| | |
| :--- | :--- |
| **Link** | *[Databrowser / API base URL]* |
| **What we use it for** | *[Import dependency, household prices, balances, renewables]* |
| **Access** | *[No key required — note update schedule if relevant]* |

**Datasets we plan to use**

| Code | Description | Status |
| :--- | :--- | :--- |
| `nrg_pc_204` | Household electricity prices | *[planned / in use]* |
| `nrg_ti_eh` | Electricity imports | *[…]* |
| `nrg_ti_gas` | Gas imports | *[…]* |
| `Nrg_bal_c` | Annual energy balance | *[…]* |
| `nrg_pc_202` | Household gas prices | *[…]* |
| `nrg_ind_ren` | Renewables share | *[…]* |
| `sdg_07_30` | Import dependency | *[…]* |
| `nrg_cb_e` | Monthly electricity balance | *[…]* |

**Persona relevance**

- **Household:** *[…]*
- **Journalist:** *[…]*
- **Policy analyst:** *[…]*

---

### 4. Open-Meteo — Weather

| | |
| :--- | :--- |
| **Link** | *[URL]* |
| **What we use it for** | *[Temperature, wind, solar — scenario inputs]* |
| **Access** | *[No key required]* |

**Persona relevance**

- **Household:** *[…]*
- **Journalist:** *[…]*
- **Policy analyst:** *[…]*

---

## Phase I deliverables checklist

| Deliverable | Owner(s) | Status |
| :--- | :--- | :--- |
| Project description (150–250 words) | Bobby | *[ ]* |
| User personas (3) + user stories (4–5 each) | Rayna, Anjali | *[ ]* |
| Data sources (2+) | Ari | *[ ]* |
| Team blog post (this page) | All | *[ ]* |
| Individual blog posts | All | *[link as published]* |

---

## Individual reflections

<!-- Each teammate: link to your post in content/<name>/ when ready -->

| Team member | Post |
| :--- | :--- |
| Ari Spokony | *[link or "coming soon"]* |
| Bobby Bress | *[link or "coming soon"]* |
| Anjali Patel | *[link or "coming soon"]* |
| Rayna Patel | *[link or "coming soon"]* |

---

## Next steps

<!-- Short bullets: what happens after Phase I -->

- *[e.g. API access, schema design, first data pull]*
- *[…]*
- *[…]*
