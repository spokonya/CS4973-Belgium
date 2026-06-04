---
title: "Bobby Bress Phase 3 Blog Post"
date: 2026-06-04
draft: false
description: "Reflection on Phase 3 contributions: journalist UI, ML integration, REST API design, and the chocolate factory visit"
slug: "bress-phase3-blog"
tags: ["UI", "journalist", "ENTSO-E", "REST API", "ML", "Belgium"]
authors:
  - "Bobby_Bress"
showAuthorsBadges : false
---

### Phase 3 Blog Post

For this section, I built the user interface pages for the journalist. I made them based on the wireframes that we made last time, but had to make a variety of changes based on the available data and a minor change in our ML models. The journalist section has 3 sections: the country snapshot, the country comparison, and the gas storage risk. The country storage page accesses ENTSO-e data via an api that pulls real time data and displays it. It shows electricity price, electricity demand, renewables share, and import dependence. There is also a chart for each country that shows where their energy comes from. The country comparison page shows the same variables, but in a comparison where there is a chart and a table that show the data. The gas storage risk page lets you input variables for storage level entering winter, change in storage over October, and storage volatility over the last 90 days and outputs wether the country is at risk and gives a risk probability. This is the machine learning model that is used for the journalist. Aside from the user interface, I worked with Ari to help brainstorm what the REST API table should look like.

Outside of the project, I really enjoyed the chocolate factory visit. It was a breath of fresh air while working on our projects, and I had a great time making the chocolate in the factory. I also thought it was great that we were able to take them home with us.
