---
title: "Zeus - Phase II"
date: 2026-05-25
draft: false
description: "EU Energy Security Index: Phase II team update"
slug: "phase2post"
tags: ["project", "phase2", "energy", "Personas"]
showAuthorsBadges: false
showTableOfContents: true
---

# Project Description

The EU Energy Security Index is our per-country dashboard for energy security risk, import dependence, gas storage, and electricity prices. In Phase II we moved from planning to execution: live API data, exploratory charts, a first supervised model on real prices, a global ER model with SQL DDL, and persona-specific UI wireframes—while refining personas around read/write interactions and securing ENTSO-E access.
 
## Updates since Phase I
Since our last update we have had a couple of changes to our project based on the feedback we received this past week as well as our own internal discussions as a team. The most notable changes are documented below:

- We reevaluated our personas through the lens of how they would interact with our site through CRUD operations, specifically focusing on interactions beyond just read. This meant thinking about how the analyst and journalist would want to save past searches to pick up where they left off, how a household owner could enter their own expenditure to compare against the national average, and similar write-oriented interactions.
- AGSI and ENTSO-E API access is confirmed. We have working programmatic access to both sources Phase I feedback asked us to confirm
- We redrafted our project description based on the feedback from our previous update, which is shown on our team posts landing page as well as at the top of this blog post. The main changes had to do with making it more action-oriented and redefining the solution as more than just a single dashboard.
- API access to ENTSO-E was granted last week, which allowed us to pull in data and clean it directly rather than relying solely on CSV downloads.

## User personas

We refined Lena and Marco against the flows we actually built in Phase III (saved preferences, spend input, forecast API, country snapshots, etc.). Sofia remains in scope for later phases; this update centers on household and journalist delivery.

### Persona 1 — Household Owner

{{< persona name="Lena Müller" role="Household Owner" age="30" location="Hamburg, Germany" initials="LM" type="household" photo="images/team/Lena.jpg" >}}
Since Phase I, Lena's persona has been grounded in a concrete billing intake flow. She now enters her utility provider, monthly bill amount, tariff type, and billing cycle dates through a profile form on her dashboard. This gives the dashboard enough context to count down to her next payment due date and frame the 30-day electricity price forecast directly against what she is currently paying. The plain-language summary remains central to her experience — she has no economics background and needs the forecast translated into a simple contract decision.

**User stories**

1. As Lena, I want to see how electricity prices in Germany are forecast to move over the next 30 days, so I can decide whether to lock in a fixed-rate contract now or wait.
2. As Lena, I want to see how current gas storage levels compare to recent winters, so I can judge whether the upcoming winter is likely to be a high-bill season.
3. As Lena, I want a plain-language summary of the forecast, so I can understand what is driving the prediction without an economics background.
4. As Lena, I want to compare Germany's situation to its neighbors, so I can tell whether price pressure is local or EU-wide.

{{< /persona >}}

---

### Persona 2 — Energy Journalist

{{< persona name="Marco Frite" role="Energy Journalist" age="30" location="Brussels, Belgium" initials="MF" type="journalist" photo="images/team/marco.jpg" >}}
Since Phase I, Marco's persona has been sharpened around two problems: the need to cite figures accurately at a specific point in time, and the need to track story leads without losing context between sessions. The country snapshot feature addresses the first — Marco can freeze a country's indicators and ML outputs as a timestamped record rather than relying on whatever the dashboard shows at publication time. Saved notes address the second by offering the ability to attach notes to any data visualization, and edit or delete them as a story develops or goes cold.

**User stories**

1. As Marco, I want current electricity prices, gas storage, and import dependence for each EU country in one place, so I can gather facts without bouncing between five sources.
2. As Marco, I want to compare a country's indicators to its neighbors, so I know which countries deserve attention in a story.
3. As Marco, I want a country snapshot I can screenshot or export, so figures in my article stay accurate without manual copy-paste.
4. As Marco, I want to compare today's prices and risk score to the same date in prior years, so I can frame whether the moment is historically unusual.

{{< /persona >}}

---

### Persona 3 — Energy Trader

{{< persona name="Niels Becker" role="Energy Trader" age="34" location="Amsterdam, Netherlands" initials="NB" type="trader" photo="images/team/niels.jpg" >}}
Niels trades electricity on the EPEX day-ahead market from a desk in Amsterdam. He positions his book 24 to 72 hours out and uses the 30-day price forecast as a directional input to that process. He works fast, trusts quantitative outputs, and keeps a running log of the decisions he makes against the data he acted on so he can evaluate forecast quality over time. Unlike Lena, who checks prices a few times a year, Niels interacts with the forecast daily and needs the dashboard scoped tightly to the markets he is actively trading.

**User stories**

1. As Niels, I want to manage a watchlist of bidding zones so my dashboard shows only the markets I am actively trading, without noise from the full EU view.
2. As Niels, I want to see the 30-day price forecast and gas storage stress risk score side by side for each watched zone, so I can assess both the price direction and the supply-side risk driving it in one view.
3. As Niels, I want to set a price alert on a bidding zone so the dashboard flags me when the forecast crosses a threshold I define, without me having to check manually throughout the day.
5. As Niels, I want to add a post-trade outcome annotation to a past trade note, so I can record whether the forecast called it correctly and build a personal track record over time.
6. As Niels, I want to browse my trade note history filtered by bidding zone and date range, so I can review patterns in how I have been responding to the forecast across different markets.

{{< /persona >}}

## Real data curation
 
### ML 1
**Where it came from and how:** 
- Pulled hourly day-ahead electricity prices for the Germany-Luxembourg bidding zone (DE_LU) from the ENTSO-E Transparency Platform via the entsoe-py Python library
- Date range: January 1, 2021 through May 25, 2026 — 64,267 hourly records pulled directly into a pandas Series

**Cleaning (pandas/numpy):** 
- Renamed columns to timestamp and price_eur_mwh for clarity
- Parsed timezone-aware timestamps using dd:mm:yyyy format and converted to Europe/Berlin timezone
- Sorted chronologically and checked for missing values (found zero gaps, no filling required)
- Extracted time features from timestamps: hour, day of week, month, year, and whether or not it's a weekend
- Aggregated hourly records into daily averages, reducing the dataset to 1,971 rows (one per day)

**Saved to CSV** 
- Saved both the full hourly cleaned file (prices_hourly_DE.csv) and the daily averaged file (prices_daily_DE.csv) to avoid re-running the API

**Exploratory analysis:** 
- **Single-variable:** daily average prices ranged from -53.87 to 699.44 EUR/MWh, with a mean of 126.51 and standard deviation of 99.48 (high variance driven by the 2022 energy crisis). The median of 95.88 EUR/MWh is more representative of typical conditions outside the crisis period. Negative prices occur briefly when renewable generation exceeds demand.
- **Two-variable:** the relationship between price and calendar month is not linear and varies across years in a loosely seasonal and irregular cyclical pattern. For example, the 2022 energy crisis distorts the loose seasonal pattern. We used one-hot encoding for month to let the model learn each month's effect independently rather than assuming any fixed relationship. Recent price history proved the most predictive overall. This is showed through the model's coefficient analysis and feature importance, which show lag_1 (electricity prices from the previous day) as the dominant feature by a wide margin.


## Data visualizations
### ML 1

{{< rawhtml >}}
<figure class="siteimg-figure">
<iframe src="/CS4973-Belgium/interactive_charts/price_history_DE.html" width="100%" height="450" frameborder="0" style="border:none;"></iframe>
<figcaption class="siteimg-figure__caption">Figure 1: Germany day-ahead electricity prices — continuous price history from 2021 through 2026</figcaption>
</figure>
{{< /rawhtml >}}

**EDA Chart 1: Germany Day Ahead Electricty Prices**

Shows how prices evolved continuously over time. Prices held steady around 50–100 EUR/MWh through 2021, spiked to 699 EUR/MWh during the 2022 Ukraine invasion period, then normalized to 75–150 EUR/MWh through 2026, confirming that the current environment is not historically unusual.

{{< rawhtml >}}
<figure class="siteimg-figure">
<iframe src="/CS4973-Belgium/interactive_charts/seasonality_DE.html" width="100%" height="450" frameborder="0" style="border:none;"></iframe>
<figcaption class="siteimg-figure__caption">Figure 2: Average electricity price by month — seasonal monthly averages for Germany</figcaption>
</figure>
{{< /rawhtml >}}

**EDA Chart 2: Average Electricty Price by Month**

Compares monthly averages as discrete categories. Summer months (July–September) average 125–160 EUR/MWh while spring (April–May) is cheapest at ~90 EUR/MWh. Note: these figures are inflated by the 2022 crisis peak.

{{< rawhtml >}}
<figure class="siteimg-figure">
<iframe src="/CS4973-Belgium/interactive_charts/yoy_comparison_DE.html" width="100%" height="450" frameborder="0" style="border:none;"></iframe>
<figcaption class="siteimg-figure__caption">Figure 3: Monthly average price per year — year-over-year monthly comparison for Germany</figcaption>
</figure>
{{< /rawhtml >}}

**EDA Chart 3: Monthly Average Price per Year**

Overlays each year's monthly trajectory for direct comparison. 2022 is a clear outlier peaking at 460 EUR/MWh, while all other years cluster tightly in the 60–130 EUR/MWh range. This answers Marco's question that the current moment is not historically unusual.

{{< rawhtml >}}
<figure class="siteimg-figure">
<iframe src="/CS4973-Belgium/interactive_charts/lr_predictions_DE.html" width="100%" height="450" frameborder="0" style="border:none;"></iframe>
<figcaption class="siteimg-figure__caption">Figure 4: Linear regression prediction vs actual — model performance on the 90-day test set</figcaption>
</figure>
{{< /rawhtml >}}

**ML1 Chart 1: Linear Regression Prediction vs Actual**

Directly evaluates model performance on unseen data. The model tracks the general trend and weekly cycles  across the February–May 2026 test period, with the main weakness being sudden sharp drops. Our R² = 0.265, so the model explains ~27% of the variance. Adjusting this model is somethng we must focus on in Phase 3, along with adding the possibility of selecting other countries for this model, and not Germany alone.

{{< rawhtml >}}
<figure class="siteimg-figure">
<iframe src="/CS4973-Belgium/interactive_charts/lr_coefficients_DE.html" width="100%" height="450" frameborder="0" style="border:none;"></iframe>
<figcaption class="siteimg-figure__caption">Figure 5: Linear regression coefficients — feature weights driving the price forecast</figcaption>
</figure>
{{< /rawhtml >}}

**ML1 Chart 2: Linear Regression Coefficients**

Shows which features drive the forecast. `lag_1` dominates by a wide margin, confirming recent price momentum is the strongest signal, while dayofweek and `lag_2` carry negative coefficients reflecting the weekend dip and short-term mean patterns.

## Machine learning

### ML 1
 
**What we built:** 
- A Linear Regression model predicting daily average electricity prices for Germany, directly targeting Lena's user story of wanting a 30-day price outlook to decide whether to lock in a fixed-rate contract
- Features include lag prices from 1, 2, 3, 7, 14, and 30 days prior, 7-day and 30-day rolling averages, and calendar features (month, day of week, year) — all normalized with StandardScaler before training
- Deliberately more complex than a neighborhood-based (kNN) model as required by the assignment, using learned linear weights across 11 engineered features rather than simple distance-based lookup
- Trained on 2021–May 2026 real ENTSO-E data, with the last 90 days held out as a test set

**How well it worked:** 
- On the 90-day test set (roughly February–May 2026), the model tracked the general price trend closely and correctly captured weekly cycles — prices consistently dipping on weekends and recovering on weekdays — confirming it learned real market patterns
- `lag_1` (yesterday's price) was by far the strongest predictor, followed by the 7-day rolling mean and `lag_3`, confirming that recent price momentum dominates
- `rolling_30d_mean` carried a negative coefficient, reflecting mean reversion — when the 30-day average is elevated the model expects a slight pullback
- The model struggled with sudden sharp spikes, which is expected behavior for a linear model — MAE, RMSE, and R² to be inserted once final run completes

**30-day forecast**
- The model accepts any user-specified start date and iteratively predicts 30 days forward, rolling lag features with each step
- Forecasts seeded from real historical prices when the start date is within the dataset, and from iterative predictions when projecting into the future
- Results are saved to CSV and visualized as an interactive Plotly chart — this function will be exposed as a REST API endpoint in Phase III for Lena to interact with on the dashboard

**Difficulties:** 
- ENTSO-E API approval took several days, delaying the start of data collection
- The 2022 crisis creates a significant statistical outlier in the training data, inflating the mean and standard deviation in ways the model has to account for
- No ready-made label for "high price risk" exists in the data — thresholds have to be defined manually for any future classifier work

**What's left:** 
- Adjust the model to be able to change the country in Phase III
- Improve the model to improve the R² score by adding gas storage and electricity demand as features
- Insert final MAE, RMSE, and R² scores once the model finishes its latest run
- Connect the forecast function to the React frontend via a REST API endpoint in Phase III
- Add gas storage and electricity demand as additional features once AGSI access is approved
- Train ML2 — the supply-shock vulnerability classifier targeting Sofia's user stories

.......

## ML 2 — Winter Storage Stress

Our second machine learning model focuses on predicting whether a European country’s gas storage will drop below 30% full during the winter. This directly connects to Sofia, our policy analyst persona, because the model helps identify which countries may be at risk of a supply shortage before winter begins.

### Data Collection

Our data was pulled from daily country-level gas storage data from the AGSI Transparency Platform through the AGSI API, which was authenticated with a personal API key. We collected data for 17 different European countries from 2014 through 2024.

### Cleaning

For cleaning, we used pandas and NumPy. We parsed `gasDayStart` as a proper datetime column and converted `full`, which represents storage percentage, into a numeric value. We also dropped any rows with missing date or storage values.

We kept only the columns that mattered for our model:

- Country
- Date
- Storage percentage full
- Gas in storage
- Trend direction

I also decided to map November through March to a single “winter year” so the winter season would not be split across two calendar years. For example, December 2018 and February 2019 are both counted as part of Winter 2018.

For each country-winter pair, we took the minimum storage percentage during winter and labeled it `storage_stress = 1` if it dropped below 30%. If it stayed above 30%, we labeled it `storage_stress = 0`. We also filtered the dataset to only include country-winters with at least 90 days of data so partial winters would not skew the label.

### Why We Chose a 30% Threshold

- We chose 30% as our stress threshold because it is both policy-related and physically meaningful. After the 2022 gas crisis, the EU set a 90% storage mandate by November 1. Since then, many EU policy analysts have treated around 28-30% as the level to start worrying about. There is also a physical reason behind this threshold. As storage empties, gas pressure drops, which slows the rate at which gas can be pulled out.
- The main issue is not just running out of gas. The bigger concern is whether storage can keep up with demand during periods of high need
- To make sure 30% was a reasonable option, we also tested 25%, 35%, and 40%. At 40%, about one in five winters got flagged, which created too many alerts for the model to actually be useful. Because of this, 30% gave us a better balance between catching real risk and avoiding too many false alarms.

### Saved Data

- We saved both `agsi_raw.csv`, which contains the raw API pull, and `agsi_clean.csv`, which contains the cleaned and typed data. This way, we do not have to re-hit the API every time we iterate on the model.

## Exploratory Analysis

- For our exploratory analysis, we looked at both single-variable and two-variable patterns in the gas storage data. On the single-variable side, storage percentage ranges from almost 0% to 100% across countries and years, but country averages vary widely. For example, Spain consistently runs higher than Poland or Belgium, which likely reflects differences in capacity, geography, and import strategy.

- For the two-variable analysis, the strongest signal was something we expected to be obvious but actually was not: starting winter near 90-100% full does not guarantee a safe winter. Several country-winters started above 90% and still dropped below the 30% stress line. This helped justify why a predictive model is useful.

### EDA Chart 1

Our first EDA chart is a multi-line time series for Germany, France, Italy, and the Netherlands from 2014 to 2024, with the 30% stress threshold marked. We chose a line chart because storage percentage is a continuous value tracked daily, and the yearly fill-and-draw cycle is the most important pattern to surface. A bar chart or scatter plot would hide this seasonal pattern.

This chart answers Lena’s question of whether the current winter is historically unusual. Most years, the lines dip but stay above 30%, while a few winters clearly cross the stress line. This gives her a visual baseline for what “normal” looks like compared to a risky winter.

### EDA Chart 2

Our second EDA chart is a horizontal ranked bar chart showing the single lowest storage point each country has ever recorded. Countries below the 30% threshold are shown as risky, while countries above the threshold are shown as safer.

We chose a ranked bar chart because the question is comparative: who is most at risk historically? Ranking makes the answer instantly readable. This chart answers Sofia’s question of which countries belong at the top of a risk ranking. Almost every country in our dataset has crossed below 30% at least once, with Spain and Poland being the main exceptions.

### EDA Chart 3

Our third EDA chart is a scatter plot of storage percentage at the start of winter on the x-axis versus the minimum storage percentage reached during winter on the y-axis. Each point represents a country-winter pair and is colored based on whether that winter ended in storage stress.

We chose a scatter plot because the main question is about the relationship between two continuous variables across many country-winter pairs. This is the chart that justifies the project: several red “stress” points sit above 90% on the x-axis, meaning a country started winter nearly full and still dropped into stress.

If a full start guaranteed a safe winter, then no model would be needed.

## Machine Learning Model

- Our model is a supervised classifier that predicts whether a country’s gas storage will drop below 30% full during winter. It directly targets Sofia’s user story of identifying countries at risk of a supply shortfall before winter begins.

- We built three features using only data available before November 1 of each winter, so the model cannot cheat by looking at the outcome:

- `storage_at_start`: average storage in the 30 days leading into winter
- `storage_trend_30d`: change in storage across those 30 days
- `storage_volatility`: standard deviation across the prior 90 days

- The model was trained on roughly 200 country-winter rows across 17 countries and 10 winters.

- We used 5-fold GroupKFold, grouping rows by winter year. This means each fold holds out a chunk of winters entirely. For example, if 2018 is in the test set, every country’s 2018 data goes to the test set and none of it appears in training. We did not use StratifiedKFold because our rows are not fully independent.

### Model Plan

- We trained two candidate models, logistic regression and random forest, and compared them instead of committing to one immediately. Logistic regression gives coefficients that are easier to explain in the dashboard, while random forest can capture interactions between storage level, trend, and volatility that a linear model might miss.

### Model Results

- The logistic regression model performed better overall. It reached 68% accuracy, with stress-class precision of 0.63 and recall of 0.59.

- The random forest underperformed at 61% accuracy and caught only 40 out of 75 stress winters, giving it a recall of 0.53. Since we only have 176 usable rows after filtering, the simpler linear model seems to generalize better. This suggests that the relationship between our current features and winter stress is mostly linear, and that we do not have enough data yet to justify the complexity of a random forest.

- The random forest still gave us a useful ranking of what matters. Storage at the start of winter was the strongest signal, followed by the 30-day storage trend and storage volatility.

### ML Chart 2

- Our first ML chart is a horizontal bar chart of feature importance from the trained random forest. It shows how much each variable contributed to the model’s predictions. The biggest driver was `storage_at_start`, which shows how full storage is when winter begins. The less obvious signals were `storage_trend_30d` and `storage_volatility`, which the model also picked up on.

- This supports our main idea that a full tank does not always mean a safe winter. Energy security also depends on how storage is changing before winter and how stable or unstable that storage pattern is.

## Difficulties

- One difficulty was working with the AGSI API. API rate limits and occasional server issues required us to build a retry loop with backoff to collect data for all 17 countries.
- Standardizing the data and three inputs as they were on largely different scales

## What’s Left

- Next, we want to adjust the decision threshold to prioritize recall because missing a real stress event is worse than creating a false alarm.

- We also want to build a separate supply-shock vulnerability classifier that adds more features beyond storage, such as import dependence and price exposure. This would make the model more complete and better connected to the full scope of our Zeus Energy Security Index dashboard.


## Data model — ER diagrams 
In this phase, we produced persona-specific ER diagrams for Household Owner, Journalist,
and Energy Trader workflows, then merged those into a global ER model for the full app.
The global model captures user/profile relationships, analytics-facing entities, and
storage for cleaned statistical records used in model training and evaluation.

### Full ER Diagram
<figure style="margin: 1.5rem 0;">
  {{< siteimg class="er-thumb" data-target="dlg-full" src="images/diagrams/FullERDiagram.png" alt="Full ER diagram for the EU Energy Security Index" style="width: 100%; display: block; cursor: zoom-in;" >}}
  <figcaption style="margin-top: 0.45rem; font-size: 0.78rem; text-align: center;">Figure 6: Full ER diagram — global model for the EU Energy Security Index — click to enlarge, press Esc to close</figcaption>
</figure>

[Download the full ER diagram (PDF)]({{< relurl "pdfs/FullERDiagram.pdf" >}})

### Household Owner ER Diagram
<figure style="margin: 1.5rem 0;">
  {{< siteimg class="er-thumb" data-target="dlg-household" src="images/diagrams/HouseholdOwner-v2.png" alt="Household Owner ER diagram" style="width: 100%; display: block; cursor: zoom-in;" >}}
  <figcaption style="margin-top: 0.45rem; font-size: 0.78rem; text-align: center;">Figure 7: Household Owner ER diagram — entities and relationships for Lena's workflow — click to enlarge, press Esc to close</figcaption>
</figure>

[Download the Household Owner ER diagram (PDF)]({{< relurl "pdfs/HouseholdOwner.pdf" >}})

### Journalist ER Diagram
<figure style="margin: 1.5rem 0;">
  {{< siteimg class="er-thumb" data-target="dlg-journalist" src="images/diagrams/JournalistDiagram-v2.png" alt="Journalist ER diagram" style="width: 100%; display: block; cursor: zoom-in;" >}}
  <figcaption style="margin-top: 0.45rem; font-size: 0.78rem; text-align: center;">Figure 8: Journalist ER diagram — entities and relationships for Marco's workflow — click to enlarge, press Esc to close</figcaption>
</figure>

[Download the Journalist ER diagram (PDF)]({{< relurl "pdfs/JournalistDiagram.pdf" >}})

### Energy Trader ER Diagram
<figure style="margin: 1.5rem 0;">
  {{< siteimg class="er-thumb" data-target="dlg-trader" src="images/diagrams/EnergyTrader.png" alt="Energy Trader ER diagram" style="width: 100%; display: block; cursor: zoom-in;" >}}
  <figcaption style="margin-top: 0.45rem; font-size: 0.78rem; text-align: center;">Figure 9: Energy Trader ER diagram — entities and relationships for Niels's workflow — click to enlarge, press Esc to close</figcaption>
</figure>

[Download the Energy Trader ER diagram (PDF)]({{< relurl "pdfs/EnergyTrader.pdf" >}})

 
## Database schema (DDL)

The schema below matches the final `database-files/` init scripts in the Zeus-Energy_Security_Index app repo. Only `CREATE TABLE` definitions are shown — seed and model-weight `INSERT` statements are in the repo but omitted here.

### 01_zeus_database.sql

```sql
-- Creates the application database (matches api/.env DB_NAME=Zeus).
CREATE DATABASE IF NOT EXISTS Zeus;
USE Zeus;
```

### 02_zeus_core.sql

```sql
-- =============================================================
-- ZEUS CORE — users + household profiles (Household Owner persona)
-- Matches: Home.py, user_routes.py, household_routes.py,
--          41_Household_Persona_Info.py
-- =============================================================
USE Zeus;
-- Mock demo users (no passwords). One row per dropdown option on Home.
-- email, country, and language are seeded and edited on the Persona Info page.
-- Seed data is loaded from 08_mockaroo_data.sql (runs after all schemas).
CREATE TABLE IF NOT EXISTS users (
    user_id      INT          NOT NULL AUTO_INCREMENT,
    display_name VARCHAR(100) NOT NULL,
    persona      ENUM('household_owner', 'journalist', 'energy_trader') NOT NULL,
    first_name   VARCHAR(50),
    email        VARCHAR(255),
    country      VARCHAR(100),
    language     VARCHAR(50),
    CONSTRAINT pk_users PRIMARY KEY (user_id)
);

-- Billing preferences per household_owner user (Persona Info billing form)
CREATE TABLE IF NOT EXISTS household_profiles (
    profile_id          INT           NOT NULL AUTO_INCREMENT,
    user_id             INT           NOT NULL,
    utility_provider    VARCHAR(100)  NOT NULL,
    monthly_bill_amount DECIMAL(10, 2) NOT NULL,
    bill_due_date       DATE          NOT NULL,
    billing_frequency   ENUM('Weekly', 'Monthly', 'Quarterly', 'Annually') NOT NULL,
    avg_monthly_kwh     DECIMAL(10, 2) NOT NULL,
    tariff_type         ENUM('Fixed rate', 'Variable rate', 'Time-of-use') NOT NULL,
    notes               TEXT,
    CONSTRAINT pk_household_profiles PRIMARY KEY (profile_id),
    CONSTRAINT uq_household_profiles_user UNIQUE (user_id),
    CONSTRAINT fk_household_profiles_user
        FOREIGN KEY (user_id) REFERENCES users (user_id) ON DELETE CASCADE
);
```

### 03_gas_storage_schema.sql

```sql
-- =============================================================
-- GAS STORAGE — journalist pages (Country Snapshot, Comparison, Risk)
-- Daily + winter rows: 06_gas_storage_data.sql (loaded on db init)
-- =============================================================
USE Zeus;
CREATE TABLE IF NOT EXISTS gas_storage_daily (
    storage_id     BIGINT       NOT NULL AUTO_INCREMENT,
    country_code   CHAR(2)      NOT NULL,
    gas_day        DATE         NOT NULL,
    full_pct       DECIMAL(6, 2) NOT NULL,
    gas_in_storage DECIMAL(12, 4),
    trend          DECIMAL(8, 2),
    CONSTRAINT pk_gas_storage_daily PRIMARY KEY (storage_id),
    CONSTRAINT uq_gas_storage_daily_country_day UNIQUE (country_code, gas_day),
    INDEX idx_gas_storage_daily_country_day (country_code, gas_day)
);

-- storage_at_start / storage_trend_30d / storage_volatility must stay DOUBLE —
-- DECIMAL rounding would change logistic-regression predictions.
CREATE TABLE IF NOT EXISTS gas_storage_winters (
    winter_id          INT          NOT NULL AUTO_INCREMENT,
    country_code       CHAR(2)      NOT NULL,
    winter_year        SMALLINT     NOT NULL,
    min_winter_full    DECIMAL(6, 2) NOT NULL,
    days               INT,
    storage_stress     TINYINT      NOT NULL,
    storage_at_start   DOUBLE       NOT NULL,
    storage_trend_30d  DOUBLE       NOT NULL,
    storage_volatility DOUBLE       NOT NULL,
    CONSTRAINT pk_gas_storage_winters PRIMARY KEY (winter_id),
    CONSTRAINT uq_gas_storage_winters_country_year UNIQUE (country_code, winter_year),
    INDEX idx_gas_storage_winters_country (country_code)
);

-- Logistic regression weights (features: storage_at_start, storage_trend_30d, storage_volatility, vol_x_start)
-- vol_x_start is the interaction term storage_at_start * storage_volatility
-- Source: datasets/apsi/apsi.ipynb — logreg.fit(X, y) on all winter rows
-- Inputs are standardized at prediction time: x_scaled = (x - mean) / std
CREATE TABLE IF NOT EXISTS gas_storage_model (
    model_id                   INT    NOT NULL,
    intercept                  DOUBLE NOT NULL,
    weight_storage_at_start    DOUBLE NOT NULL,
    weight_storage_trend_30d   DOUBLE NOT NULL,
    weight_storage_volatility  DOUBLE NOT NULL,
    weight_vol_x_start         DOUBLE NOT NULL,
    mean_storage_at_start      DOUBLE NOT NULL,
    mean_storage_trend_30d     DOUBLE NOT NULL,
    mean_storage_volatility    DOUBLE NOT NULL,
    mean_vol_x_start           DOUBLE NOT NULL,
    std_storage_at_start       DOUBLE NOT NULL,
    std_storage_trend_30d      DOUBLE NOT NULL,
    std_storage_volatility     DOUBLE NOT NULL,
    std_vol_x_start            DOUBLE NOT NULL,
    CONSTRAINT pk_gas_storage_model PRIMARY KEY (model_id)
);
```

### 04_zeus_persona_features.sql

```sql
-- =============================================================
-- OPTIONAL PERSONA FEATURES
-- Household: saved EU energy news articles
-- Journalist: frozen snapshot payloads + private journalist notes
-- =============================================================
USE Zeus;
-- Household Owner — bookmark articles from GET /news/eu-energy
CREATE TABLE IF NOT EXISTS saved_articles (
    article_id  INT           NOT NULL AUTO_INCREMENT,
    user_id     INT           NOT NULL,
    title       VARCHAR(300)  NOT NULL,
    link        VARCHAR(500)  NOT NULL,
    source_name VARCHAR(150),
    description TEXT,
    pub_date    DATETIME,
    saved_at    DATETIME      NOT NULL DEFAULT CURRENT_TIMESTAMP,
    CONSTRAINT pk_saved_articles PRIMARY KEY (article_id),
    CONSTRAINT fk_saved_articles_user
        FOREIGN KEY (user_id) REFERENCES users (user_id) ON DELETE CASCADE,
    INDEX idx_saved_articles_user (user_id)
);

-- Journalist — save a frozen indicator + ML output bundle for citation
CREATE TABLE IF NOT EXISTS snapshots (
    snapshot_id  INT          NOT NULL AUTO_INCREMENT,
    user_id      INT          NOT NULL,
    country_code CHAR(2)      NOT NULL,
    label        VARCHAR(150),
    payload      JSON         NOT NULL,
    created_at   DATETIME     NOT NULL DEFAULT CURRENT_TIMESTAMP,
    CONSTRAINT pk_snapshots PRIMARY KEY (snapshot_id),
    CONSTRAINT fk_snapshots_user
        FOREIGN KEY (user_id) REFERENCES users (user_id) ON DELETE CASCADE,
    INDEX idx_snapshots_user (user_id)
);

-- Journalist — private notes tied to a country (optional) or general notes
CREATE TABLE IF NOT EXISTS notes (
    note_id      INT           NOT NULL AUTO_INCREMENT,
    user_id      INT           NOT NULL,
    country_code CHAR(2),
    content      VARCHAR(2000) NOT NULL,
    context      JSON,
    created_at   DATETIME      NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at   DATETIME      NOT NULL DEFAULT CURRENT_TIMESTAMP
        ON UPDATE CURRENT_TIMESTAMP,
    CONSTRAINT pk_notes PRIMARY KEY (note_id),
    CONSTRAINT fk_notes_user
        FOREIGN KEY (user_id) REFERENCES users (user_id) ON DELETE CASCADE,
    INDEX idx_notes_user (user_id)
);
```

### 05_price_prediction.sql

```sql
-- =============================================================
-- PRICE PREDICTION — journalist Price Forecast page
-- Daily prices: seeded from datasets/entsoe/data/clean/prices_daily_ALL.csv
-- Model weights: seeded from datasets/entsoe/models/lr_price_forecast_ALL.pkl
--                and      datasets/entsoe/models/lr_scaler_params_ALL.json
-- =============================================================
USE Zeus;
-- Daily electricity prices for all countries (source: ENTSO-E)
CREATE TABLE IF NOT EXISTS price_daily (
    price_id          BIGINT        NOT NULL AUTO_INCREMENT,
    price_date        DATE          NOT NULL,
    country           CHAR(2)       NOT NULL,
    avg_price_eur_mwh DOUBLE        NOT NULL,
    CONSTRAINT pk_price_daily PRIMARY KEY (price_id),
    CONSTRAINT uq_price_daily_date_country UNIQUE (price_date, country),
    INDEX idx_price_daily_country_date (country, price_date)
);

-- Linear regression weights stored as a single wide row, one column per feature.
CREATE TABLE IF NOT EXISTS price_model_weights (
    model_id                INT    NOT NULL AUTO_INCREMENT,
    intercept               DOUBLE NOT NULL,
    weight_lag_1            DOUBLE, weight_lag_2            DOUBLE, weight_lag_3            DOUBLE,
    weight_lag_4            DOUBLE, weight_lag_5            DOUBLE, weight_lag_6            DOUBLE,
    weight_lag_7            DOUBLE,
    weight_rolling_7d_mean  DOUBLE, weight_rolling_30d_mean DOUBLE,
    weight_rolling_7d_std   DOUBLE, weight_price_vs_7d_avg  DOUBLE,
    weight_month_2          DOUBLE, weight_month_3          DOUBLE, weight_month_4          DOUBLE,
    weight_month_5          DOUBLE, weight_month_6          DOUBLE, weight_month_7          DOUBLE,
    weight_month_8          DOUBLE, weight_month_9          DOUBLE, weight_month_10         DOUBLE,
    weight_month_11         DOUBLE, weight_month_12         DOUBLE,
    weight_dow_1            DOUBLE, weight_dow_2            DOUBLE, weight_dow_3            DOUBLE,
    weight_dow_4            DOUBLE, weight_dow_5            DOUBLE, weight_dow_6            DOUBLE,
    weight_country_BE       DOUBLE, weight_country_BG       DOUBLE, weight_country_CZ       DOUBLE,
    weight_country_DE       DOUBLE, weight_country_ES       DOUBLE, weight_country_FR       DOUBLE,
    weight_country_HR       DOUBLE, weight_country_HU       DOUBLE, weight_country_LV       DOUBLE,
    weight_country_NL       DOUBLE, weight_country_PL       DOUBLE, weight_country_PT       DOUBLE,
    weight_country_RO       DOUBLE, weight_country_SK       DOUBLE,
    CONSTRAINT pk_price_model_weights PRIMARY KEY (model_id)
);

-- Model means and st. devs
CREATE TABLE IF NOT EXISTS price_model_scaler (
    scaler_id       INT NOT NULL AUTO_INCREMENT,
    feature_name    VARCHAR(50) NOT NULL,
    feature_mean    DOUBLE NOT NULL,
    feature_std     DOUBLE NOT NULL,
    CONSTRAINT pk_price_model_scaler PRIMARY KEY (scaler_id)
);

```

### 07_energy_trader_schema.sql

```sql
-- =============================================================
-- ENERGY TRADER (persona 3) — watchlist, price alerts, trade journal
-- Matches: 52_My_Markets.py, 53_Trade_Journal.py
-- Price forecast ML1 data lives in 05_price_prediction.sql
-- =============================================================
USE Zeus;
-- Bidding zones the trader is actively watching (My Markets watchlist)
CREATE TABLE IF NOT EXISTS trader_watchlist (
    watchlist_id INT      NOT NULL AUTO_INCREMENT,
    user_id      INT      NOT NULL,
    country_code CHAR(2)  NOT NULL,
    added_at     DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
    CONSTRAINT pk_trader_watchlist PRIMARY KEY (watchlist_id),
    CONSTRAINT uq_trader_watchlist_user_country UNIQUE (user_id, country_code),
    CONSTRAINT fk_trader_watchlist_user
        FOREIGN KEY (user_id) REFERENCES users (user_id) ON DELETE CASCADE,
    INDEX idx_trader_watchlist_user (user_id)
);

-- Per-zone forecast threshold alerts (My Markets price alerts)
CREATE TABLE IF NOT EXISTS trader_price_alerts (
    alert_id     INT                              NOT NULL AUTO_INCREMENT,
    user_id      INT                              NOT NULL,
    country_code CHAR(2)                          NOT NULL,
    threshold    DECIMAL(10, 2)                   NOT NULL,
    direction    ENUM('above', 'below')           NOT NULL,
    created_at   DATETIME                         NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at   DATETIME                         NOT NULL DEFAULT CURRENT_TIMESTAMP
        ON UPDATE CURRENT_TIMESTAMP,
    CONSTRAINT pk_trader_price_alerts PRIMARY KEY (alert_id),
    CONSTRAINT uq_trader_price_alerts_user_country UNIQUE (user_id, country_code),
    CONSTRAINT fk_trader_price_alerts_user
        FOREIGN KEY (user_id) REFERENCES users (user_id) ON DELETE CASCADE,
    INDEX idx_trader_price_alerts_user (user_id)
);

-- Trade journal entries — forecast calls, rationale, and outcome annotation
CREATE TABLE IF NOT EXISTS trader_trade_notes (
    note_id        INT                                           NOT NULL AUTO_INCREMENT,
    user_id        INT                                           NOT NULL,
    trade_date     DATE                                          NOT NULL,
    country_code   CHAR(2)                                       NOT NULL,
    direction      ENUM('Long', 'Short', 'Hedge')                NOT NULL,
    forecast_call  VARCHAR(300),
    note           TEXT,
    outcome        ENUM('Pending', 'Forecast correct', 'Forecast wrong')
                   NOT NULL DEFAULT 'Pending',
    outcome_note   VARCHAR(500),
    created_at     DATETIME                                      NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at     DATETIME                                      NOT NULL DEFAULT CURRENT_TIMESTAMP
        ON UPDATE CURRENT_TIMESTAMP,
    CONSTRAINT pk_trader_trade_notes PRIMARY KEY (note_id),
    CONSTRAINT fk_trader_trade_notes_user
        FOREIGN KEY (user_id) REFERENCES users (user_id) ON DELETE CASCADE,
    INDEX idx_trader_trade_notes_user_date (user_id, trade_date)
);

```

## Relational Database Diagram

The final schema splits into two layers: **core identity and persona features** (everything keyed off `users`) and **model and time-series tables** (trained weights, scaler parameters, and the historical data the API reads at prediction time).

### Core identity and persona features

<figure style="margin: 1.5rem 0;">
  {{< siteimg class="er-thumb" data-target="dlg-db-core" src="images/phase4/db_core_persona.png" alt="Database schema — users and persona feature tables" style="width: 100%; display: block; cursor: zoom-in;" >}}
  <figcaption style="margin-top: 0.45rem; font-size: 0.78rem; text-align: center;">Figure 10: Core identity and persona features (users hub with household profiles, trader watchlists, alerts, trade notes, saved articles, snapshots, and notes) — click to enlarge, press Esc to close</figcaption>
</figure>

### Model and time-series tables

<figure style="margin: 1.5rem 0;">
  {{< siteimg class="er-thumb" data-target="dlg-db-model" src="images/phase4/db_model_tables.png" alt="Database schema — price and gas storage model tables" style="width: 100%; display: block; cursor: zoom-in;" >}}
  <figcaption style="margin-top: 0.45rem; font-size: 0.78rem; text-align: center;">Figure 11: Model and time-series tables (price weights/scaler, daily price and storage history, gas storage model, and winter feature rows) — click to enlarge, press Esc to close</figcaption>
</figure>

## UI wireframes

These are our initial wireframes for the landing page and persona-specific app views.
The layouts and interactions are expected to evolve as development continues.

### Home Page
<figure style="margin: 1.5rem 0;">
  {{< siteimg class="er-thumb" data-target="dlg-home" src="images/diagrams/Home_Page.png" alt="Home page wireframe" style="width: 100%; display: block; cursor: zoom-in;" >}}
  <figcaption style="margin-top: 0.45rem; font-size: 0.78rem; text-align: center;">Figure 12: Home page wireframe — landing page and persona selection — click to enlarge, press Esc to close</figcaption>
</figure>

### Household Owner – View 1
<figure style="margin: 1.5rem 0;">
  {{< siteimg class="er-thumb" data-target="dlg-household1" src="images/diagrams/Household_View1.png" alt="Household Owner wireframe — View 1" style="width: 100%; display: block; cursor: zoom-in;" >}}
  <figcaption style="margin-top: 0.45rem; font-size: 0.78rem; text-align: center;">Figure 13: Household Owner wireframe — View 1 — click to enlarge, press Esc to close</figcaption>
</figure>

### Household Owner – View 2
<figure style="margin: 1.5rem 0;">
  {{< siteimg class="er-thumb" data-target="dlg-household2" src="images/diagrams/Household_View2.png" alt="Household Owner wireframe — View 2" style="width: 100%; display: block; cursor: zoom-in;" >}}
  <figcaption style="margin-top: 0.45rem; font-size: 0.78rem; text-align: center;">Figure 14: Household Owner wireframe — View 2 — click to enlarge, press Esc to close</figcaption>
</figure>

### Journalist – View 1
<figure style="margin: 1.5rem 0;">
  {{< siteimg class="er-thumb" data-target="dlg-journalist1" src="images/diagrams/Journalist_View1.png" alt="Journalist wireframe — View 1" style="width: 100%; display: block; cursor: zoom-in;" >}}
  <figcaption style="margin-top: 0.45rem; font-size: 0.78rem; text-align: center;">Figure 15: Journalist wireframe — View 1 — click to enlarge, press Esc to close</figcaption>
</figure>

### Journalist – View 2
<figure style="margin: 1.5rem 0;">
  {{< siteimg class="er-thumb" data-target="dlg-journalist2" src="images/diagrams/Journalist_View2.png" alt="Journalist wireframe — View 2" style="width: 100%; display: block; cursor: zoom-in;" >}}
  <figcaption style="margin-top: 0.45rem; font-size: 0.78rem; text-align: center;">Figure 16: Journalist wireframe — View 2 — click to enlarge, press Esc to close</figcaption>
</figure>

### Policy Analyst – View 1
<figure style="margin: 1.5rem 0;">
  {{< siteimg class="er-thumb" data-target="dlg-policy1" src="images/diagrams/Policy_AnalystView1.png" alt="Policy Analyst wireframe — View 1" style="width: 100%; display: block; cursor: zoom-in;" >}}
  <figcaption style="margin-top: 0.45rem; font-size: 0.78rem; text-align: center;">Figure 17: Policy Analyst wireframe — View 1 — click to enlarge, press Esc to close</figcaption>
</figure>

### Policy Analyst – View 2
<figure style="margin: 1.5rem 0;">
  {{< siteimg class="er-thumb" data-target="dlg-policy2" src="images/diagrams/PolicyAnalystView2.png" alt="Policy Analyst wireframe — View 2" style="width: 100%; display: block; cursor: zoom-in;" >}}
  <figcaption style="margin-top: 0.45rem; font-size: 0.78rem; text-align: center;">Figure 18: Policy Analyst wireframe — View 2 — click to enlarge, press Esc to close</figcaption>
</figure>

<dialog id="dlg-full" class="er-dialog" style="padding: 0; border: none; background: rgba(0,0,0,0.92); max-width: 100vw; max-height: 100vh; width: 100vw; height: 100vh;">
  <div style="position: fixed; top: 1rem; left: 50%; transform: translateX(-50%); color: white; font-size: 0.85rem; background: rgba(0,0,0,0.6); padding: 0.4rem 0.9rem; border-radius: 9999px; pointer-events: none;">Press <kbd style="background: white; color: black; padding: 0.05rem 0.4rem; border-radius: 0.25rem; font-family: inherit;">Esc</kbd> to close</div>
  {{< siteimg src="images/diagrams/FullERDiagram.png" alt="Full ER diagram (enlarged)" style="display: block; max-width: 100%; max-height: 100%; object-fit: contain; margin: auto; padding: 2rem;" >}}
</dialog>

<dialog id="dlg-household" class="er-dialog" style="padding: 0; border: none; background: rgba(0,0,0,0.92); max-width: 100vw; max-height: 100vh; width: 100vw; height: 100vh;">
  <div style="position: fixed; top: 1rem; left: 50%; transform: translateX(-50%); color: white; font-size: 0.85rem; background: rgba(0,0,0,0.6); padding: 0.4rem 0.9rem; border-radius: 9999px; pointer-events: none;">Press <kbd style="background: white; color: black; padding: 0.05rem 0.4rem; border-radius: 0.25rem; font-family: inherit;">Esc</kbd> to close</div>
  {{< siteimg src="images/diagrams/HouseholdOwner-v2.png" alt="Household Owner ER diagram (enlarged)" style="display: block; max-width: 100%; max-height: 100%; object-fit: contain; margin: auto; padding: 2rem;" >}}
</dialog>

<dialog id="dlg-journalist" class="er-dialog" style="padding: 0; border: none; background: rgba(0,0,0,0.92); max-width: 100vw; max-height: 100vh; width: 100vw; height: 100vh;">
  <div style="position: fixed; top: 1rem; left: 50%; transform: translateX(-50%); color: white; font-size: 0.85rem; background: rgba(0,0,0,0.6); padding: 0.4rem 0.9rem; border-radius: 9999px; pointer-events: none;">Press <kbd style="background: white; color: black; padding: 0.05rem 0.4rem; border-radius: 0.25rem; font-family: inherit;">Esc</kbd> to close</div>
  {{< siteimg src="images/diagrams/JournalistDiagram-v2.png" alt="Journalist ER diagram (enlarged)" style="display: block; max-width: 100%; max-height: 100%; object-fit: contain; margin: auto; padding: 2rem;" >}}
</dialog>

<dialog id="dlg-trader" class="er-dialog" style="padding: 0; border: none; background: rgba(0,0,0,0.92); max-width: 100vw; max-height: 100vh; width: 100vw; height: 100vh;">
  <div style="position: fixed; top: 1rem; left: 50%; transform: translateX(-50%); color: white; font-size: 0.85rem; background: rgba(0,0,0,0.6); padding: 0.4rem 0.9rem; border-radius: 9999px; pointer-events: none;">Press <kbd style="background: white; color: black; padding: 0.05rem 0.4rem; border-radius: 0.25rem; font-family: inherit;">Esc</kbd> to close</div>
  {{< siteimg src="images/diagrams/EnergyTrader.png" alt="Energy Trader ER diagram (enlarged)" style="display: block; max-width: 100%; max-height: 100%; object-fit: contain; margin: auto; padding: 2rem;" >}}
</dialog>

<dialog id="dlg-home" class="er-dialog" style="padding: 0; border: none; background: rgba(0,0,0,0.92); max-width: 100vw; max-height: 100vh; width: 100vw; height: 100vh;">
  <div style="position: fixed; top: 1rem; left: 50%; transform: translateX(-50%); color: white; font-size: 0.85rem; background: rgba(0,0,0,0.6); padding: 0.4rem 0.9rem; border-radius: 9999px; pointer-events: none;">Press <kbd style="background: white; color: black; padding: 0.05rem 0.4rem; border-radius: 0.25rem; font-family: inherit;">Esc</kbd> to close</div>
  {{< siteimg src="images/diagrams/Home_Page.png" alt="Home page wireframe (enlarged)" style="display: block; max-width: 100%; max-height: 100%; object-fit: contain; margin: auto; padding: 2rem;" >}}
</dialog>

<dialog id="dlg-household1" class="er-dialog" style="padding: 0; border: none; background: rgba(0,0,0,0.92); max-width: 100vw; max-height: 100vh; width: 100vw; height: 100vh;">
  <div style="position: fixed; top: 1rem; left: 50%; transform: translateX(-50%); color: white; font-size: 0.85rem; background: rgba(0,0,0,0.6); padding: 0.4rem 0.9rem; border-radius: 9999px; pointer-events: none;">Press <kbd style="background: white; color: black; padding: 0.05rem 0.4rem; border-radius: 0.25rem; font-family: inherit;">Esc</kbd> to close</div>
  {{< siteimg src="images/diagrams/Household_View1.png" alt="Household Owner wireframe — View 1 (enlarged)" style="display: block; max-width: 100%; max-height: 100%; object-fit: contain; margin: auto; padding: 2rem;" >}}
</dialog>

<dialog id="dlg-household2" class="er-dialog" style="padding: 0; border: none; background: rgba(0,0,0,0.92); max-width: 100vw; max-height: 100vh; width: 100vw; height: 100vh;">
  <div style="position: fixed; top: 1rem; left: 50%; transform: translateX(-50%); color: white; font-size: 0.85rem; background: rgba(0,0,0,0.6); padding: 0.4rem 0.9rem; border-radius: 9999px; pointer-events: none;">Press <kbd style="background: white; color: black; padding: 0.05rem 0.4rem; border-radius: 0.25rem; font-family: inherit;">Esc</kbd> to close</div>
  {{< siteimg src="images/diagrams/Household_View2.png" alt="Household Owner wireframe — View 2 (enlarged)" style="display: block; max-width: 100%; max-height: 100%; object-fit: contain; margin: auto; padding: 2rem;" >}}
</dialog>

<dialog id="dlg-journalist1" class="er-dialog" style="padding: 0; border: none; background: rgba(0,0,0,0.92); max-width: 100vw; max-height: 100vh; width: 100vw; height: 100vh;">
  <div style="position: fixed; top: 1rem; left: 50%; transform: translateX(-50%); color: white; font-size: 0.85rem; background: rgba(0,0,0,0.6); padding: 0.4rem 0.9rem; border-radius: 9999px; pointer-events: none;">Press <kbd style="background: white; color: black; padding: 0.05rem 0.4rem; border-radius: 0.25rem; font-family: inherit;">Esc</kbd> to close</div>
  {{< siteimg src="images/diagrams/Journalist_View1.png" alt="Journalist wireframe — View 1 (enlarged)" style="display: block; max-width: 100%; max-height: 100%; object-fit: contain; margin: auto; padding: 2rem;" >}}
</dialog>

<dialog id="dlg-journalist2" class="er-dialog" style="padding: 0; border: none; background: rgba(0,0,0,0.92); max-width: 100vw; max-height: 100vh; width: 100vw; height: 100vh;">
  <div style="position: fixed; top: 1rem; left: 50%; transform: translateX(-50%); color: white; font-size: 0.85rem; background: rgba(0,0,0,0.6); padding: 0.4rem 0.9rem; border-radius: 9999px; pointer-events: none;">Press <kbd style="background: white; color: black; padding: 0.05rem 0.4rem; border-radius: 0.25rem; font-family: inherit;">Esc</kbd> to close</div>
  {{< siteimg src="images/diagrams/Journalist_View2.png" alt="Journalist wireframe — View 2 (enlarged)" style="display: block; max-width: 100%; max-height: 100%; object-fit: contain; margin: auto; padding: 2rem;" >}}
</dialog>

<dialog id="dlg-policy1" class="er-dialog" style="padding: 0; border: none; background: rgba(0,0,0,0.92); max-width: 100vw; max-height: 100vh; width: 100vw; height: 100vh;">
  <div style="position: fixed; top: 1rem; left: 50%; transform: translateX(-50%); color: white; font-size: 0.85rem; background: rgba(0,0,0,0.6); padding: 0.4rem 0.9rem; border-radius: 9999px; pointer-events: none;">Press <kbd style="background: white; color: black; padding: 0.05rem 0.4rem; border-radius: 0.25rem; font-family: inherit;">Esc</kbd> to close</div>
  {{< siteimg src="images/diagrams/Policy_AnalystView1.png" alt="Policy Analyst wireframe — View 1 (enlarged)" style="display: block; max-width: 100%; max-height: 100%; object-fit: contain; margin: auto; padding: 2rem;" >}}
</dialog>

<dialog id="dlg-policy2" class="er-dialog" style="padding: 0; border: none; background: rgba(0,0,0,0.92); max-width: 100vw; max-height: 100vh; width: 100vw; height: 100vh;">
  <div style="position: fixed; top: 1rem; left: 50%; transform: translateX(-50%); color: white; font-size: 0.85rem; background: rgba(0,0,0,0.6); padding: 0.4rem 0.9rem; border-radius: 9999px; pointer-events: none;">Press <kbd style="background: white; color: black; padding: 0.05rem 0.4rem; border-radius: 0.25rem; font-family: inherit;">Esc</kbd> to close</div>
  {{< siteimg src="images/diagrams/PolicyAnalystView2.png" alt="Policy Analyst wireframe — View 2 (enlarged)" style="display: block; max-width: 100%; max-height: 100%; object-fit: contain; margin: auto; padding: 2rem;" >}}
</dialog>

<dialog id="dlg-db-core" class="er-dialog" style="padding: 0; border: none; background: rgba(0,0,0,0.92); max-width: 100vw; max-height: 100vh; width: 100vw; height: 100vh;">
  <div style="position: fixed; top: 1rem; left: 50%; transform: translateX(-50%); color: white; font-size: 0.85rem; background: rgba(0,0,0,0.6); padding: 0.4rem 0.9rem; border-radius: 9999px; pointer-events: none;">Press <kbd style="background: white; color: black; padding: 0.05rem 0.4rem; border-radius: 0.25rem; font-family: inherit;">Esc</kbd> to close</div>
  {{< siteimg src="images/phase4/db_core_persona.png" alt="Core identity and persona features (enlarged)" style="display: block; max-width: 100%; max-height: 100%; object-fit: contain; margin: auto; padding: 2rem;" >}}
</dialog>

<dialog id="dlg-db-model" class="er-dialog" style="padding: 0; border: none; background: rgba(0,0,0,0.92); max-width: 100vw; max-height: 100vh; width: 100vw; height: 100vh;">
  <div style="position: fixed; top: 1rem; left: 50%; transform: translateX(-50%); color: white; font-size: 0.85rem; background: rgba(0,0,0,0.6); padding: 0.4rem 0.9rem; border-radius: 9999px; pointer-events: none;">Press <kbd style="background: white; color: black; padding: 0.05rem 0.4rem; border-radius: 0.25rem; font-family: inherit;">Esc</kbd> to close</div>
  {{< siteimg src="images/phase4/db_model_tables.png" alt="Model and time-series tables (enlarged)" style="display: block; max-width: 100%; max-height: 100%; object-fit: contain; margin: auto; padding: 2rem;" >}}
</dialog>

<script>
(function() {
  document.querySelectorAll('.er-thumb').forEach(function(thumb) {
    thumb.addEventListener('click', function() {
      var dlg = document.getElementById(thumb.dataset.target);
      if (dlg) dlg.showModal();
    });
  });
})();
</script>

## Individual posts
Each teammate also published an individual update documenting their direct Phase II
contributions, including data/visualization tasks and the data-model areas they led.
{{< members >}}
{{< member name="Ari Spokony" role="" href="/ari_spokony/" initials="AS" photo="images/team/ari.jpg" >}}
{{< member name="Bobby Bress" role="" href="/bobby_bress/" initials="BB" photo="images/team/bobby.jpg" >}}
{{< member name="Anjali Patel" role="" href="/anjali_patel/" initials="AP" photo="images/team/anjali.jpg" >}}
{{< member name="Rayna Patel" role="" href="/rayna_patel/" initials="RP" photo="images/team/rayna.jpg" >}}
{{< /members >}}
 
