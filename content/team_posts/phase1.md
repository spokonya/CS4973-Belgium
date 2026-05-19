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
