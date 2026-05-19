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

*[Team introduction — 2–3 sentences]*

---

## At a glance

| | |
| :--- | :--- |
| **Project** | EU Energy Security Index |
| **Problem** | *[One-line problem statement]* |
| **Solution** | *[One-line solution — interactive per-country EU dashboard]* |
| **Primary users** | Households · Journalists · Policy analysts |

---

## Project description

<!-- Owner: Bobby | Target: 150–250 words | Source: ideas.md → "Project Description" -->

*[Paste final project description here]*

### Why now

<!-- Optional: 1 short paragraph on post-2022 energy security context -->

*[Context paragraph — optional]*

### What we're building

<!-- Bullet or short list: dashboard views + scenario inputs -->

- *[Dashboard feature 1]*
- *[Dashboard feature 2]*
- *[Dashboard feature 3]*

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

### Persona 2 — Energy Journalist

| | |
| :--- | :--- |
| **Name** | *Marco Frite* |
| **Role** | *Energy Journalist* |
| **Age** | *30* |
| **Location** | *Brussels, Belgium* |
| **Description** | *Writes 2-3 articles a week on gas markets, electricity prices, and polcy decisions* |
| **Goals** | *Gather current country-level energy data fast. Find newsworthy comparisons and outliers across the EU. Although, our data is scattered across ENTSO-E, gas storage portals, and Eurostat. Each source has its own format* |

**User stories**

1. As Marco, I want to view current electricity prices, gas storage levels, and import dependence for each EU country in one place, so that I can quickly gather facts for a story without bouncing between five different sources

2. As Marco, I want to see how a country's energy indicators compare to its neighbors to see what countries to highlight in reports

3. As Marco, I want a country snapshot I can screenshot or export, so that I can include accurate, current figures in my article without copy-pasting numbers manually

4. As Marco, I want to see how a country's current prices and risk score compare to the same date in previous years, so that I can put today's

---

### Persona 3 — Policy analyst

| | |
| :--- | :--- |
| **Name** | *Sofia Andereson* |
| **Role** | *Policy Analyst* |
| **Age** | *47* |
| **Location** | *Brussels, Belgium* |
| **Description** | *Senior researcher who writes briefing memos for MEPs and national energy councils. Highly educated and comfortable using models* |
| **Goals** | *Identify the most vulnerable EU countries to brief policymakers. Model supply shock scenarios for memos.* |

**User stories**

1. As Sofia, I want to rank EU countries by their winter stress risk score, so that I can identify which member states need attention in briefings

2. As Sofia, I want to adjust gas storage levels and weather scenarios for each country, so that I can model different supply shock conditions for policymakers

3. As Sofia, I want to see which features are driving each country's risk score, so that I can write policy recommendations 

4. As Sofia, I want to export country comparison data, so that I can include it in policy memos and presentations without retyping numbers

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

- *DS3000:* Complete data collection, cleaning, and EDA/Data Vizualization, Build POC of ML Models in Jupyter
- *CS3200:* Localized data model generation for personas, integrate localized data mdels into Global Data Model
- *Together:* Draft Wireframes of POC
