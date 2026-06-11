---
title: "Blog Post 4"
date: 2026-06-11
draft: false
description: "Anjali Blog Post4"
slug: "anjali-blog4"
tags: ["ds3000", "Python", "C-Mine", "Belgium"]
authors:
  - "Anjali_Patel"
showAuthorsBadges : false
---

## Data Cleaning and Visulization
![Escape Room](https://www.amctheatres.com/movies/saving-private-ryan-1015)

### Deliverable 4 Update

#### Backend
On the backend I cleaned up the features and ran a bunch of checks before trusting the model. The main thing I caught was a multicollinearity issue with our interaction term. The raw storage_at_start × storage_volatility feature was 0.95-correlated with storage_volatility and pushed the VIF up to 43.6, which made the coefficients pretty much useless. I fixed it by mean-centering both features before multiplying, which brought every VIF back down to around 1 without losing the interaction. The rest of the assumptions held up fine too: the outcome is binary with a 103/81 class balance, the correlation matrix didn't show any other redundant features, and the Box-Tidwell test passed for all four features, so none of them needed a transform. For the predictive side I kept the 5-fold stratified cross-validation, looked at precision and recall instead of just accuracy, and compared logistic regression against a random forest before sticking with logistic regression.

#### Frontend
For this phase I focused on refining Model 2's frontend, the Gas Storage Risk page, and the backend behind it. On the frontend I cleaned up the what-if tool where a user picks one of the 17 countries and adjusts three inputs. I rewrote the input labels and feature explanations to be clearer and more technical, especially the storage volatility one, where I explained what a high vs low value actually means. The bottom of the page used to be a single block on why we chose 30%, so I split it into three side-by-side sections: why 30%, where Europe's gas comes from, and how countries store it, which makes the page easier to read. I also kept the scatter plot that shows the main point of the model, that a full tank at the start of winter doesn't mean a safe one, with the 30% stress line, the selected country highlighted, and the user's current scenario dropped in so they can see where it lands.


### Escape Room and C-Mine
I got to participate in the Saving Private Ryan escape room which is one of my favorite War movies, so it was exciting trying to escape it. It was fun solving puzzles with peers that I didn't know well beforehand. Additionally, it was super cool to explore the C-mine and see what Genk was built upon. IMO, I think the coal mines should open back up again and be exploited. 