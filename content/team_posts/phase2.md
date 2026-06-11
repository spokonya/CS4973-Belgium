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

{{< siteimg src="images/charts/price_history_DEcopy.png" alt="Germany Day Ahead Electricity Prices" width="600" >}}

**EDA Chart 1: Germany Day Ahead Electricty Prices**

Shows how prices evolved continuously over time. Prices held steady around 50–100 EUR/MWh through 2021, spiked to 699 EUR/MWh during the 2022 Ukraine invasion period, then normalized to 75–150 EUR/MWh through 2026, confirming that the current environment is not historically unusual.

{{< siteimg src="images/charts/seasonality_DEcopy.png" alt="Avg Electricity Price by Month" width="600" >}}

**EDA Chart 2: Average Electricty Price by Month**

Compares monthly averages as discrete categories. Summer months (July–September) average 125–160 EUR/MWh while spring (April–May) is cheapest at ~90 EUR/MWh. Note: these figures are inflated by the 2022 crisis peak.

{{< siteimg src="images/charts/yoy_comparison_DEcopy.png" alt="Monthly Avg Price per Year" width="600" >}}

**EDA Chart 3: Monthly Average Price pper Year**

Overlays each year's monthly trajectory for direct comparison. 2022 is a clear outlier peaking at 460 EUR/MWh, while all other years cluster tightly in the 60–130 EUR/MWh range. This answers Marco's question that the current moment is not historically unusual.

{{< siteimg src="images/charts/lr_predictions_DEcopy.png" alt="Linear Regression Prediction vs Actual" width="600" >}}

**ML1 Chart 1: Linear Regression Prediction vs Actual**

Directly evaluates model performance on unseen data. The model tracks the general trend and weekly cycles  across the February–May 2026 test period, with the main weakness being sudden sharp drops. Our R² = 0.265, so the model explains ~27% of the variance. Adjusting this model is somethng we must focus on in Phase 3, along with adding the possibility of selecting other countries for this model, and not Germany alone.

{{< siteimg src="images/charts/lr_coefficients_DEcopy.png" alt="Linear Regression Coefficients" width="600" >}}

**ML1 Chart 2: Linear Regression Coefficients**

Shows which features drive the forecast. `lag_1` dominates by a wide margin, confirming recent price momentum is the strongest signal, while dayofweek and `lag_2` carry negative coefficients reflecting the weekend dip and short-term mean patterns.

## Machine learning

### ML 1
 
**What we built:** 
- A Linear Regression model predicting daily average electricity prices for Germany, directly targeting Lena's user story of wanting a 30-day price outlook to decide whether to lock in a fixed-rate contract
- Features include lag prices from 1, 2, 3, 7, 14, and 30 days prior, 7-day and 30-day rolling averages, and calendar features (month, day of week, year) — all normalized with StandardScaler before training
- Deliberately more complex than a neighborhood-based (kNN) model as required by the assignment, using learned linear weights across 11 engineered features rather than simple distance-based lookup
- Trained on 2021–May 2026 real ENTSO-E data, with the last 90 days held out as a test set

**The plan for two models:** 
you'll ultimately have at least two models on real data;
  parts not yet on real data (like the day-ahead price forecast) currently use simulated
  data, which is allowed for this phase.

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

## ML 2: Winter Gas Storage Stress Model

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

### EDA Chart 1: EU Gas Storage Levels Over Time

Our first EDA chart is a multi-line time series for Germany, France, Italy, and the Netherlands from 2014 to 2024, with the 30% stress threshold marked. We chose a line chart because storage percentage is a continuous value tracked daily, and the yearly fill-and-draw cycle is the most important pattern to surface. A bar chart or scatter plot would hide this seasonal pattern.

This chart answers Lena’s question of whether the current winter is historically unusual. Most years, the lines dip but stay above 30%, while a few winters clearly cross the stress line. This gives her a visual baseline for what “normal” looks like compared to a risky winter.

### EDA Chart 2: Lowest Winter Storage Ever Recorded by Country

Our second EDA chart is a horizontal ranked bar chart showing the single lowest storage point each country has ever recorded. Countries below the 30% threshold are shown as risky, while countries above the threshold are shown as safer.

We chose a ranked bar chart because the question is comparative: who is most at risk historically? Ranking makes the answer instantly readable. This chart answers Sofia’s question of which countries belong at the top of a risk ranking. Almost every country in our dataset has crossed below 30% at least once, with Spain and Poland being the main exceptions.

### EDA Chart 3: Does a Full Start Mean a Safe Winter?

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

### ML Chart 1: A Full Tank Does Not Mean a Safe Winter

- Our first ML chart is a horizontal bar chart of feature importance from the trained random forest. It shows how much each variable contributed to the model’s predictions. The biggest driver was `storage_at_start`, which shows how full storage is when winter begins. The less obvious signals were `storage_trend_30d` and `storage_volatility`, which the model also picked up on.

- This supports our main idea that a full tank does not always mean a safe winter. Energy security also depends on how storage is changing before winter and how stable or unstable that storage pattern is.

## Difficulties

- One difficulty was working with the AGSI API. API rate limits and occasional server issues required us to build a retry loop with backoff to collect data for all 17 countries.

- Another difficulty was that the “stress” label is only a proxy for supply-shock risk. It is useful for our policy analyst persona because it flags risky winters, but it is not a perfect measure of energy insecurity.

- The dataset is also small, with only around 200 rows. This limits how much we can trust more complex models because they are more likely to overfit. Our current features are storage-only, which also limits the model’s ability to explain why a winter becomes risky.

## What’s Left

- Next, we want to adjust the decision threshold to prioritize recall because missing a real stress event is worse than creating a false alarm.

- We also want to build a separate supply-shock vulnerability classifier that adds more features beyond storage, such as import dependence and price exposure. This would make the model more complete and better connected to the full scope of our Zeus Energy Security Index dashboard.


## Data model — ER diagrams 
In this phase, we produced persona-specific ER diagrams for Household Owner, Journalist,
and Energy Trader workflows, then merged those into a global ER model for the full app.
The global model captures user/profile relationships, analytics-facing entities, and
storage for cleaned statistical records used in model training and evaluation.

### Full ER Diagram
<figure style="margin: 1.5rem 0 1.5rem -6rem;">
  {{< siteimg class="er-thumb" data-target="dlg-full" src="images/diagrams/FullERDiagram.png" alt="Full ER diagram for the EU Energy Security Index" style="width: 100%; display: block; cursor: zoom-in;" >}}
  <figcaption style="margin-top: 0.45rem; font-size: 0.78rem; text-align: center;">Full ER diagram — click to enlarge, press Esc to close</figcaption>
</figure>

[Download the full ER diagram (PDF)]({{< relurl "pdfs/FullERDiagram.pdf" >}})

### Household Owner ER Diagram
<figure style="margin: 1.5rem 0 1.5rem -6rem;">
  {{< siteimg class="er-thumb" data-target="dlg-household" src="images/diagrams/HouseholdOwner-v2.png" alt="Household Owner ER diagram" style="width: 100%; display: block; cursor: zoom-in;" >}}
  <figcaption style="margin-top: 0.45rem; font-size: 0.78rem; text-align: center;">Household Owner ER diagram — click to enlarge, press Esc to close</figcaption>
</figure>

[Download the Household Owner ER diagram (PDF)]({{< relurl "pdfs/HouseholdOwner.pdf" >}})

### Journalist ER Diagram
<figure style="margin: 1.5rem 0 1.5rem -6rem;">
  {{< siteimg class="er-thumb" data-target="dlg-journalist" src="images/diagrams/JournalistDiagram-v2.png" alt="Journalist ER diagram" style="width: 100%; display: block; cursor: zoom-in;" >}}
  <figcaption style="margin-top: 0.45rem; font-size: 0.78rem; text-align: center;">Journalist ER diagram — click to enlarge, press Esc to close</figcaption>
</figure>

[Download the Journalist ER diagram (PDF)]({{< relurl "pdfs/JournalistDiagram.pdf" >}})

### Energy Trader ER Diagram
<figure style="margin: 1.5rem 0 1.5rem -6rem;">
  {{< siteimg class="er-thumb" data-target="dlg-trader" src="images/diagrams/EnergyTrader.png" alt="Energy Trader ER diagram" style="width: 100%; display: block; cursor: zoom-in;" >}}
  <figcaption style="margin-top: 0.45rem; font-size: 0.78rem; text-align: center;">Energy Trader ER diagram — click to enlarge, press Esc to close</figcaption>
</figure>

[Download the Energy Trader ER diagram (PDF)]({{< relurl "pdfs/EnergyTrader.pdf" >}})

 
## Database — first-pass schema (DDL) 
From the global ER model, we drafted a first-pass SQL schema with explicit primary/foreign
keys and role-specific entities. This DDL establishes the baseline structure for app data,
including users, personas, content, comments, and statistical records that support ML
feature workflows.

```sql
CREATE TABLE IF NOT EXISTS Stat_Type (
    Stat_Type_ID     INT          NOT NULL AUTO_INCREMENT,
    Cross_Border_Flow VARCHAR(100),
    Imports          VARCHAR(100),
    Storage_Levels   INT,
    Price_Info       INT,
    CONSTRAINT pk_stat_type PRIMARY KEY (Stat_Type_ID)
);

CREATE TABLE IF NOT EXISTS Stats (
    Stat_ID          INT          NOT NULL AUTO_INCREMENT,
    Country          VARCHAR(100),
    Unit             VARCHAR(50),
    Quantity         INT,
    Used_For_Training BOOLEAN     NOT NULL DEFAULT FALSE,
    Stat_Type_ID     INT          NOT NULL,
    CONSTRAINT pk_stats          PRIMARY KEY (Stat_ID),
    CONSTRAINT fk_stats_stat_type_v2 FOREIGN KEY (Stat_Type_ID) REFERENCES Stat_Type (Stat_Type_ID)
);

CREATE TABLE IF NOT EXISTS User (
    User_ID    INT          NOT NULL AUTO_INCREMENT,
    Name       VARCHAR(150) NOT NULL,
    Email      VARCHAR(255) NOT NULL,
    CONSTRAINT pk_user       PRIMARY KEY (User_ID),
    CONSTRAINT uq_user_email UNIQUE (Email)
);

CREATE TABLE IF NOT EXISTS Profile (
    Profile_ID           INT          NOT NULL AUTO_INCREMENT,
    Countries_of_Interest VARCHAR(255),
    User_ID              INT          NOT NULL,
    Position             VARCHAR(150),
    CONSTRAINT pk_profile      PRIMARY KEY (Profile_ID),
    CONSTRAINT fk_profile_user_v2 FOREIGN KEY (User_ID) REFERENCES User (User_ID)
);

CREATE TABLE IF NOT EXISTS Household_Owner (
    Household_Owner_ID  INT NOT NULL AUTO_INCREMENT,
    User_ID             INT NOT NULL,
    CONSTRAINT pk_household_owner      PRIMARY KEY (Household_Owner_ID),
    CONSTRAINT fk_household_owner_user_v2 FOREIGN KEY (User_ID) REFERENCES User (User_ID)
);

CREATE TABLE IF NOT EXISTS Policy_Analyst (
    Analyst_ID  INT NOT NULL AUTO_INCREMENT,
    User_ID     INT NOT NULL,
    CONSTRAINT pk_policy_analyst      PRIMARY KEY (Analyst_ID),
    CONSTRAINT fk_policy_analyst_user_v2 FOREIGN KEY (User_ID) REFERENCES User (User_ID)
);

CREATE TABLE IF NOT EXISTS Journalist (
    Journalist_ID  INT NOT NULL AUTO_INCREMENT,
    User_ID        INT NOT NULL,
    CONSTRAINT pk_journalist      PRIMARY KEY (Journalist_ID),
    CONSTRAINT fk_journalist_user_v2 FOREIGN KEY (User_ID) REFERENCES User (User_ID)
);

CREATE TABLE IF NOT EXISTS Articles (
    Article_ID   INT          NOT NULL AUTO_INCREMENT,
    Link_To_Live VARCHAR(500),
    Theme_Tags   VARCHAR(255),
    User_ID      INT          NOT NULL,
    Date         DATE,
    Title        VARCHAR(300),
    Content      VARCHAR(5000),
    CONSTRAINT pk_articles      PRIMARY KEY (Article_ID),
    CONSTRAINT fk_articles_user_v2 FOREIGN KEY (User_ID) REFERENCES User (User_ID)
);

CREATE TABLE IF NOT EXISTS Comments (
    ID            INT          NOT NULL AUTO_INCREMENT,
    Content       VARCHAR(2000),
    Date          DATE,
    Journalist_ID INT          NOT NULL,
    Analyst_ID    INT          NOT NULL,
    Stat_ID       INT          NOT NULL,
    Visibility    BOOLEAN      NOT NULL DEFAULT TRUE,
    CONSTRAINT pk_comments           PRIMARY KEY (ID),
    CONSTRAINT fk_comments_journalist_v2 FOREIGN KEY (Journalist_ID) REFERENCES Journalist    (Journalist_ID),
    CONSTRAINT fk_comments_analyst_v2  FOREIGN KEY (Analyst_ID)    REFERENCES Policy_Analyst (Analyst_ID),
    CONSTRAINT fk_comments_stat_v2     FOREIGN KEY (Stat_ID)       REFERENCES Stats          (Stat_ID)
);
```

## Relational Database Diagram
<figure style="margin: 1.5rem 0 1.5rem -6rem;">
  {{< siteimg class="er-thumb" data-target="dlg-database" src="images/diagrams/Database_Diagram.png" alt="Relational database diagram" style="width: 100%; display: block; cursor: zoom-in;" >}}
  <figcaption style="margin-top: 0.45rem; font-size: 0.78rem; text-align: center;">Relational database diagram — click to enlarge, press Esc to close</figcaption>
</figure>

## UI wireframes

These are our initial wireframes for the landing page and persona-specific app views.
The layouts and interactions are expected to evolve as development continues.

### Home Page
<figure style="margin: 1.5rem 0 1.5rem -6rem;">
  {{< siteimg class="er-thumb" data-target="dlg-home" src="images/diagrams/Home_Page.png" alt="Home page wireframe" style="width: 100%; display: block; cursor: zoom-in;" >}}
  <figcaption style="margin-top: 0.45rem; font-size: 0.78rem; text-align: center;">Home page — click to enlarge, press Esc to close</figcaption>
</figure>

### Household Owner – View 1
<figure style="margin: 1.5rem 0 1.5rem -6rem;">
  {{< siteimg class="er-thumb" data-target="dlg-household1" src="images/diagrams/Household_View1.png" alt="Household Owner wireframe — View 1" style="width: 100%; display: block; cursor: zoom-in;" >}}
  <figcaption style="margin-top: 0.45rem; font-size: 0.78rem; text-align: center;">Household Owner — View 1 — click to enlarge, press Esc to close</figcaption>
</figure>

### Household Owner – View 2
<figure style="margin: 1.5rem 0 1.5rem -6rem;">
  {{< siteimg class="er-thumb" data-target="dlg-household2" src="images/diagrams/Household_View2.png" alt="Household Owner wireframe — View 2" style="width: 100%; display: block; cursor: zoom-in;" >}}
  <figcaption style="margin-top: 0.45rem; font-size: 0.78rem; text-align: center;">Household Owner — View 2 — click to enlarge, press Esc to close</figcaption>
</figure>

### Journalist – View 1
<figure style="margin: 1.5rem 0 1.5rem -6rem;">
  {{< siteimg class="er-thumb" data-target="dlg-journalist1" src="images/diagrams/Journalist_View1.png" alt="Journalist wireframe — View 1" style="width: 100%; display: block; cursor: zoom-in;" >}}
  <figcaption style="margin-top: 0.45rem; font-size: 0.78rem; text-align: center;">Journalist — View 1 — click to enlarge, press Esc to close</figcaption>
</figure>

### Journalist – View 2
<figure style="margin: 1.5rem 0 1.5rem -6rem;">
  {{< siteimg class="er-thumb" data-target="dlg-journalist2" src="images/diagrams/Journalist_View2.png" alt="Journalist wireframe — View 2" style="width: 100%; display: block; cursor: zoom-in;" >}}
  <figcaption style="margin-top: 0.45rem; font-size: 0.78rem; text-align: center;">Journalist — View 2 — click to enlarge, press Esc to close</figcaption>
</figure>

### Policy Analyst – View 1
<figure style="margin: 1.5rem 0 1.5rem -6rem;">
  {{< siteimg class="er-thumb" data-target="dlg-policy1" src="images/diagrams/Policy_AnalystView1.png" alt="Policy Analyst wireframe — View 1" style="width: 100%; display: block; cursor: zoom-in;" >}}
  <figcaption style="margin-top: 0.45rem; font-size: 0.78rem; text-align: center;">Policy Analyst — View 1 — click to enlarge, press Esc to close</figcaption>
</figure>

### Policy Analyst – View 2
<figure style="margin: 1.5rem 0 1.5rem -6rem;">
  {{< siteimg class="er-thumb" data-target="dlg-policy2" src="images/diagrams/PolicyAnalystView2.png" alt="Policy Analyst wireframe — View 2" style="width: 100%; display: block; cursor: zoom-in;" >}}
  <figcaption style="margin-top: 0.45rem; font-size: 0.78rem; text-align: center;">Policy Analyst — View 2 — click to enlarge, press Esc to close</figcaption>
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

<dialog id="dlg-database" class="er-dialog" style="padding: 0; border: none; background: rgba(0,0,0,0.92); max-width: 100vw; max-height: 100vh; width: 100vw; height: 100vh;">
  <div style="position: fixed; top: 1rem; left: 50%; transform: translateX(-50%); color: white; font-size: 0.85rem; background: rgba(0,0,0,0.6); padding: 0.4rem 0.9rem; border-radius: 9999px; pointer-events: none;">Press <kbd style="background: white; color: black; padding: 0.05rem 0.4rem; border-radius: 0.25rem; font-family: inherit;">Esc</kbd> to close</div>
  {{< siteimg src="images/diagrams/Database_Diagram.png" alt="Relational database diagram (enlarged)" style="display: block; max-width: 100%; max-height: 100%; object-fit: contain; margin: auto; padding: 2rem;" >}}
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
 
