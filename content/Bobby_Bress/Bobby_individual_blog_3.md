---
title: "Bobby Bress Phase 3 Blog Post"
date: 2026-06-04
draft: false
description: "Reflection on Phase 3 contributions: journalist UI, ML integration, REST API design, and the chocolate factory visit"
slug: "bress-phase3-blog"
tags: ["UI", "journalist", "gas storage", "REST API", "ML", "Belgium"]
authors:
  - "Bobby_Bress"
showAuthorsBadges : false
---

### Phase 3 Blog Post

For this section, I built the user interface pages for the journalist. I made them based on the wireframes that we made last time, but had to make a variety of changes based on the available data and a shift in how we framed our ML models around gas storage. The journalist section has 3 sections: the country snapshot, the country comparison, and the gas storage risk. The country snapshot page shows ten years of gas storage history for a country at a time. It displays the current storage level, the number of stressed winters on record, the lowest winter level on record, and the 30% stress threshold. There is also a time-series chart that shows how storage fills and drains each year, with the stress line marked, and a "context for your story" panel that gives the journalist EU policy background and curated external links to follow. The country comparison page runs the gas storage risk model across every country we have data for and ranks them by risk probability. It flags which countries the model predicts could fall below 30% storage this winter and shows them all in a horizontal bar chart sorted by risk. The gas storage risk page lets you input variables for storage level entering winter, change in storage over October, and storage volatility over the last 90 days, and outputs whether the country is at risk along with a risk probability. This is the machine learning model that is used for the journalist. Aside from the user interface, I worked with Ari to help brainstorm what the REST API table should look like.

Outside of the project, I really enjoyed the chocolate factory visit. It was a breath of fresh air while working on our projects, and I had a great time making the chocolate in the factory. I also thought it was great that we were able to take them home with us.
