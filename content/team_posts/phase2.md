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
## Data model — ER diagrams 
For **each persona**, include a localized data model (an ER diagram) built from the data
their user stories need. Then show one **global ER diagram** that combines them into the
full app. Make sure it includes a place to store the cleaned, pre-processed features used
to train and test the ML models.

 
## Database — first-pass schema (DDL) 
Include a first draft of the SQL `CREATE TABLE` statements for the global data model,
including the table that holds the ML feature data.

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
  <img class="er-thumb" data-target="dlg-database" src="/images/diagrams/Database_Diagram.png" alt="Relational database diagram" style="width: 100%; display: block; cursor: zoom-in;" loading="lazy" />
  <figcaption style="margin-top: 0.45rem; font-size: 0.78rem; text-align: center;">Relational database diagram — click to enlarge, press Esc to close</figcaption>
</figure>

## UI wireframes


### Home Page
<figure style="margin: 1.5rem 0 1.5rem -6rem;">
  <img class="er-thumb" data-target="dlg-home" src="/images/diagrams/Home_Page.png" alt="Home page wireframe" style="width: 100%; display: block; cursor: zoom-in;" loading="lazy" />
  <figcaption style="margin-top: 0.45rem; font-size: 0.78rem; text-align: center;">Home page — click to enlarge, press Esc to close</figcaption>
</figure>

### Household Owner – View 1
<figure style="margin: 1.5rem 0 1.5rem -6rem;">
  <img class="er-thumb" data-target="dlg-household1" src="/images/diagrams/Household_View1.png" alt="Household Owner wireframe — View 1" style="width: 100%; display: block; cursor: zoom-in;" loading="lazy" />
  <figcaption style="margin-top: 0.45rem; font-size: 0.78rem; text-align: center;">Household Owner — View 1 — click to enlarge, press Esc to close</figcaption>
</figure>

### Household Owner – View 2
<figure style="margin: 1.5rem 0 1.5rem -6rem;">
  <img class="er-thumb" data-target="dlg-household2" src="/images/diagrams/Household_View2.png" alt="Household Owner wireframe — View 2" style="width: 100%; display: block; cursor: zoom-in;" loading="lazy" />
  <figcaption style="margin-top: 0.45rem; font-size: 0.78rem; text-align: center;">Household Owner — View 2 — click to enlarge, press Esc to close</figcaption>
</figure>

### Journalist – View 1
<figure style="margin: 1.5rem 0 1.5rem -6rem;">
  <img class="er-thumb" data-target="dlg-journalist1" src="/images/diagrams/Journalist_View1.png" alt="Journalist wireframe — View 1" style="width: 100%; display: block; cursor: zoom-in;" loading="lazy" />
  <figcaption style="margin-top: 0.45rem; font-size: 0.78rem; text-align: center;">Journalist — View 1 — click to enlarge, press Esc to close</figcaption>
</figure>

### Journalist – View 2
<figure style="margin: 1.5rem 0 1.5rem -6rem;">
  <img class="er-thumb" data-target="dlg-journalist2" src="/images/diagrams/Journalist_View2.png" alt="Journalist wireframe — View 2" style="width: 100%; display: block; cursor: zoom-in;" loading="lazy" />
  <figcaption style="margin-top: 0.45rem; font-size: 0.78rem; text-align: center;">Journalist — View 2 — click to enlarge, press Esc to close</figcaption>
</figure>

### Policy Analyst – View 1
<figure style="margin: 1.5rem 0 1.5rem -6rem;">
  <img class="er-thumb" data-target="dlg-policy1" src="/images/diagrams/Policy_AnalystView1.png" alt="Policy Analyst wireframe — View 1" style="width: 100%; display: block; cursor: zoom-in;" loading="lazy" />
  <figcaption style="margin-top: 0.45rem; font-size: 0.78rem; text-align: center;">Policy Analyst — View 1 — click to enlarge, press Esc to close</figcaption>
</figure>

### Policy Analyst – View 2
<figure style="margin: 1.5rem 0 1.5rem -6rem;">
  <img class="er-thumb" data-target="dlg-policy2" src="/images/diagrams/PolicyAnalystView2.png" alt="Policy Analyst wireframe — View 2" style="width: 100%; display: block; cursor: zoom-in;" loading="lazy" />
  <figcaption style="margin-top: 0.45rem; font-size: 0.78rem; text-align: center;">Policy Analyst — View 2 — click to enlarge, press Esc to close</figcaption>
</figure>

## ER Diagrams

### Full ER Diagram
<figure style="margin: 1.5rem 0 1.5rem -6rem;">
  <img class="er-thumb" data-target="dlg-full" src="/images/diagrams/FullERDiagram.png" alt="Full ER diagram for the EU Energy Security Index" style="width: 100%; display: block; cursor: zoom-in;" loading="lazy" />
  <figcaption style="margin-top: 0.45rem; font-size: 0.78rem; text-align: center;">Full ER diagram — click to enlarge, press Esc to close</figcaption>
</figure>

[Download the full ER diagram (PDF)](/pdfs/FullERDiagram.pdf)

### Household Owner ER Diagram
<figure style="margin: 1.5rem 0 1.5rem -6rem;">
  <img class="er-thumb" data-target="dlg-household" src="/images/diagrams/HouseholdOwner.png" alt="Household Owner ER diagram" style="width: 100%; display: block; cursor: zoom-in;" loading="lazy" />
  <figcaption style="margin-top: 0.45rem; font-size: 0.78rem; text-align: center;">Household Owner ER diagram — click to enlarge, press Esc to close</figcaption>
</figure>

[Download the Household Owner ER diagram (PDF)](/pdfs/HouseholdOwner.pdf)

### Journalist ER Diagram
<figure style="margin: 1.5rem 0 1.5rem -6rem;">
  <img class="er-thumb" data-target="dlg-journalist" src="/images/diagrams/JournalistDiagram.png" alt="Journalist ER diagram" style="width: 100%; display: block; cursor: zoom-in;" loading="lazy" />
  <figcaption style="margin-top: 0.45rem; font-size: 0.78rem; text-align: center;">Journalist ER diagram — click to enlarge, press Esc to close</figcaption>
</figure>

[Download the Journalist ER diagram (PDF)](/pdfs/JournalistDiagram.pdf)

### Policy Analyst ER Diagram
<figure style="margin: 1.5rem 0 1.5rem -6rem;">
  <img class="er-thumb" data-target="dlg-policy" src="/images/diagrams/PolicyAnalyst.png" alt="Policy Analyst ER diagram" style="width: 100%; display: block; cursor: zoom-in;" loading="lazy" />
  <figcaption style="margin-top: 0.45rem; font-size: 0.78rem; text-align: center;">Policy Analyst ER diagram — click to enlarge, press Esc to close</figcaption>
</figure>

[Download the Policy Analyst ER diagram (PDF)](/pdfs/PolicyAnalyst.pdf)

<dialog id="dlg-full" class="er-dialog" style="padding: 0; border: none; background: rgba(0,0,0,0.92); max-width: 100vw; max-height: 100vh; width: 100vw; height: 100vh;">
  <div style="position: fixed; top: 1rem; left: 50%; transform: translateX(-50%); color: white; font-size: 0.85rem; background: rgba(0,0,0,0.6); padding: 0.4rem 0.9rem; border-radius: 9999px; pointer-events: none;">Press <kbd style="background: white; color: black; padding: 0.05rem 0.4rem; border-radius: 0.25rem; font-family: inherit;">Esc</kbd> to close</div>
  <img src="/images/diagrams/FullERDiagram.png" alt="Full ER diagram (enlarged)" style="display: block; max-width: 100%; max-height: 100%; object-fit: contain; margin: auto; padding: 2rem;" />
</dialog>

<dialog id="dlg-household" class="er-dialog" style="padding: 0; border: none; background: rgba(0,0,0,0.92); max-width: 100vw; max-height: 100vh; width: 100vw; height: 100vh;">
  <div style="position: fixed; top: 1rem; left: 50%; transform: translateX(-50%); color: white; font-size: 0.85rem; background: rgba(0,0,0,0.6); padding: 0.4rem 0.9rem; border-radius: 9999px; pointer-events: none;">Press <kbd style="background: white; color: black; padding: 0.05rem 0.4rem; border-radius: 0.25rem; font-family: inherit;">Esc</kbd> to close</div>
  <img src="/images/diagrams/HouseholdOwner.png" alt="Household Owner ER diagram (enlarged)" style="display: block; max-width: 100%; max-height: 100%; object-fit: contain; margin: auto; padding: 2rem;" />
</dialog>

<dialog id="dlg-journalist" class="er-dialog" style="padding: 0; border: none; background: rgba(0,0,0,0.92); max-width: 100vw; max-height: 100vh; width: 100vw; height: 100vh;">
  <div style="position: fixed; top: 1rem; left: 50%; transform: translateX(-50%); color: white; font-size: 0.85rem; background: rgba(0,0,0,0.6); padding: 0.4rem 0.9rem; border-radius: 9999px; pointer-events: none;">Press <kbd style="background: white; color: black; padding: 0.05rem 0.4rem; border-radius: 0.25rem; font-family: inherit;">Esc</kbd> to close</div>
  <img src="/images/diagrams/JournalistDiagram.png" alt="Journalist ER diagram (enlarged)" style="display: block; max-width: 100%; max-height: 100%; object-fit: contain; margin: auto; padding: 2rem;" />
</dialog>

<dialog id="dlg-policy" class="er-dialog" style="padding: 0; border: none; background: rgba(0,0,0,0.92); max-width: 100vw; max-height: 100vh; width: 100vw; height: 100vh;">
  <div style="position: fixed; top: 1rem; left: 50%; transform: translateX(-50%); color: white; font-size: 0.85rem; background: rgba(0,0,0,0.6); padding: 0.4rem 0.9rem; border-radius: 9999px; pointer-events: none;">Press <kbd style="background: white; color: black; padding: 0.05rem 0.4rem; border-radius: 0.25rem; font-family: inherit;">Esc</kbd> to close</div>
  <img src="/images/diagrams/PolicyAnalyst.png" alt="Policy Analyst ER diagram (enlarged)" style="display: block; max-width: 100%; max-height: 100%; object-fit: contain; margin: auto; padding: 2rem;" />
</dialog>

<dialog id="dlg-home" class="er-dialog" style="padding: 0; border: none; background: rgba(0,0,0,0.92); max-width: 100vw; max-height: 100vh; width: 100vw; height: 100vh;">
  <div style="position: fixed; top: 1rem; left: 50%; transform: translateX(-50%); color: white; font-size: 0.85rem; background: rgba(0,0,0,0.6); padding: 0.4rem 0.9rem; border-radius: 9999px; pointer-events: none;">Press <kbd style="background: white; color: black; padding: 0.05rem 0.4rem; border-radius: 0.25rem; font-family: inherit;">Esc</kbd> to close</div>
  <img src="/images/diagrams/Home_Page.png" alt="Home page wireframe (enlarged)" style="display: block; max-width: 100%; max-height: 100%; object-fit: contain; margin: auto; padding: 2rem;" />
</dialog>

<dialog id="dlg-household1" class="er-dialog" style="padding: 0; border: none; background: rgba(0,0,0,0.92); max-width: 100vw; max-height: 100vh; width: 100vw; height: 100vh;">
  <div style="position: fixed; top: 1rem; left: 50%; transform: translateX(-50%); color: white; font-size: 0.85rem; background: rgba(0,0,0,0.6); padding: 0.4rem 0.9rem; border-radius: 9999px; pointer-events: none;">Press <kbd style="background: white; color: black; padding: 0.05rem 0.4rem; border-radius: 0.25rem; font-family: inherit;">Esc</kbd> to close</div>
  <img src="/images/diagrams/Household_View1.png" alt="Household Owner wireframe — View 1 (enlarged)" style="display: block; max-width: 100%; max-height: 100%; object-fit: contain; margin: auto; padding: 2rem;" />
</dialog>

<dialog id="dlg-household2" class="er-dialog" style="padding: 0; border: none; background: rgba(0,0,0,0.92); max-width: 100vw; max-height: 100vh; width: 100vw; height: 100vh;">
  <div style="position: fixed; top: 1rem; left: 50%; transform: translateX(-50%); color: white; font-size: 0.85rem; background: rgba(0,0,0,0.6); padding: 0.4rem 0.9rem; border-radius: 9999px; pointer-events: none;">Press <kbd style="background: white; color: black; padding: 0.05rem 0.4rem; border-radius: 0.25rem; font-family: inherit;">Esc</kbd> to close</div>
  <img src="/images/diagrams/Household_View2.png" alt="Household Owner wireframe — View 2 (enlarged)" style="display: block; max-width: 100%; max-height: 100%; object-fit: contain; margin: auto; padding: 2rem;" />
</dialog>

<dialog id="dlg-journalist1" class="er-dialog" style="padding: 0; border: none; background: rgba(0,0,0,0.92); max-width: 100vw; max-height: 100vh; width: 100vw; height: 100vh;">
  <div style="position: fixed; top: 1rem; left: 50%; transform: translateX(-50%); color: white; font-size: 0.85rem; background: rgba(0,0,0,0.6); padding: 0.4rem 0.9rem; border-radius: 9999px; pointer-events: none;">Press <kbd style="background: white; color: black; padding: 0.05rem 0.4rem; border-radius: 0.25rem; font-family: inherit;">Esc</kbd> to close</div>
  <img src="/images/diagrams/Journalist_View1.png" alt="Journalist wireframe — View 1 (enlarged)" style="display: block; max-width: 100%; max-height: 100%; object-fit: contain; margin: auto; padding: 2rem;" />
</dialog>

<dialog id="dlg-journalist2" class="er-dialog" style="padding: 0; border: none; background: rgba(0,0,0,0.92); max-width: 100vw; max-height: 100vh; width: 100vw; height: 100vh;">
  <div style="position: fixed; top: 1rem; left: 50%; transform: translateX(-50%); color: white; font-size: 0.85rem; background: rgba(0,0,0,0.6); padding: 0.4rem 0.9rem; border-radius: 9999px; pointer-events: none;">Press <kbd style="background: white; color: black; padding: 0.05rem 0.4rem; border-radius: 0.25rem; font-family: inherit;">Esc</kbd> to close</div>
  <img src="/images/diagrams/Journalist_View2.png" alt="Journalist wireframe — View 2 (enlarged)" style="display: block; max-width: 100%; max-height: 100%; object-fit: contain; margin: auto; padding: 2rem;" />
</dialog>

<dialog id="dlg-policy1" class="er-dialog" style="padding: 0; border: none; background: rgba(0,0,0,0.92); max-width: 100vw; max-height: 100vh; width: 100vw; height: 100vh;">
  <div style="position: fixed; top: 1rem; left: 50%; transform: translateX(-50%); color: white; font-size: 0.85rem; background: rgba(0,0,0,0.6); padding: 0.4rem 0.9rem; border-radius: 9999px; pointer-events: none;">Press <kbd style="background: white; color: black; padding: 0.05rem 0.4rem; border-radius: 0.25rem; font-family: inherit;">Esc</kbd> to close</div>
  <img src="/images/diagrams/Policy_AnalystView1.png" alt="Policy Analyst wireframe — View 1 (enlarged)" style="display: block; max-width: 100%; max-height: 100%; object-fit: contain; margin: auto; padding: 2rem;" />
</dialog>

<dialog id="dlg-policy2" class="er-dialog" style="padding: 0; border: none; background: rgba(0,0,0,0.92); max-width: 100vw; max-height: 100vh; width: 100vw; height: 100vh;">
  <div style="position: fixed; top: 1rem; left: 50%; transform: translateX(-50%); color: white; font-size: 0.85rem; background: rgba(0,0,0,0.6); padding: 0.4rem 0.9rem; border-radius: 9999px; pointer-events: none;">Press <kbd style="background: white; color: black; padding: 0.05rem 0.4rem; border-radius: 0.25rem; font-family: inherit;">Esc</kbd> to close</div>
  <img src="/images/diagrams/PolicyAnalystView2.png" alt="Policy Analyst wireframe — View 2 (enlarged)" style="display: block; max-width: 100%; max-height: 100%; object-fit: contain; margin: auto; padding: 2rem;" />
</dialog>

<dialog id="dlg-database" class="er-dialog" style="padding: 0; border: none; background: rgba(0,0,0,0.92); max-width: 100vw; max-height: 100vh; width: 100vw; height: 100vh;">
  <div style="position: fixed; top: 1rem; left: 50%; transform: translateX(-50%); color: white; font-size: 0.85rem; background: rgba(0,0,0,0.6); padding: 0.4rem 0.9rem; border-radius: 9999px; pointer-events: none;">Press <kbd style="background: white; color: black; padding: 0.05rem 0.4rem; border-radius: 0.25rem; font-family: inherit;">Esc</kbd> to close</div>
  <img src="/images/diagrams/Database_Diagram.png" alt="Relational database diagram (enlarged)" style="display: block; max-width: 100%; max-height: 100%; object-fit: contain; margin: auto; padding: 2rem;" />
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
Each teammate writes their own post covering: which charts or data-pipeline steps they
personally worked on, which localized data model they were the point person for, and
(optional) a short note about anything notable from the week.
{{< members >}}
{{< member name="Ari Spokony" role="" href="/ari_spokony/" initials="AS" photo="images/team/ari.jpg" >}}
{{< member name="Bobby Bress" role="" href="/bobby_bress/" initials="BB" photo="images/team/bobby.jpg" >}}
{{< member name="Anjali Patel" role="" href="/anjali_patel/" initials="AP" photo="images/team/anjali.jpg" >}}
{{< member name="Rayna Patel" role="" href="/rayna_patel/" initials="RP" photo="images/team/rayna.jpg" >}}
{{< /members >}}
 
