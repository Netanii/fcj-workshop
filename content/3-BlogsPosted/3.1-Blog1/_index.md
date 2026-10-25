---
title: "Blog 1"
date: 2026-10-23
weight: 1
chapter: false
pre: " <b> 3.1. </b> "
---

# Building co-purchase recommendations on Lambda + DynamoDB instead of Amazon Personalize

*Nguyễn Trọng Nhân · shop-ai project, First Cloud AI Journey 2026*

My team is building shop-ai, a serverless computer-parts shop on AWS. My part is product recommendations: "Frequently bought together" on the product page and "For you" on the home page. At first the team planned to use Amazon Personalize. This post explains why we dropped it, how we built a simple model ourselves, and what we measured.

## 1. Why not Amazon Personalize

Personalize is a ready-made recommendation service: you feed it data and it returns recommendations. When the team read the pricing carefully (recorded in ADR-0016 of our repository on 01/10/2026), we found three problems:

1. **You pay in three places:** data ingestion, training hours, and the campaign that serves real-time recommendations.
2. **A campaign is billed for at least 1 request per second**, even when nobody calls it. We estimated about $0.54 per hour with a v2 recipe. If someone forgets to turn it off, a month can reach about $394, while the budget for the whole project is under $15.
3. **Personalize is only available on the Paid plan**, and all three of our accounts are on the Free plan.

Our mentor's advice: if we only need product recommendations at low cost, we should build a simple model ourselves on Lambda and DynamoDB. Our architecture already had a place for this: recommendations are computed in a nightly batch and written to a table, and the API only reads that table.

> The prices above are the numbers the team recorded when making the decision. Check the AWS pricing page before reusing them, because prices can change.

## 2. Architecture

```text
EventBridge Scheduler (every night)
        │
        ▼
Python Lambda: read events and orders → validate data → compute recommendations
        │
        ▼
DynamoDB recs table: write a new version → compare with the baseline → switch the ACTIVE pointer
        │
        ▼
API GET /api/v1/recs/... (reads the pointer, 60-second cache) → widget on the website
```

Two important design decisions:

- **Write by version, switch with one pointer.** Each run writes a new version of the recommendations (the version number is the run date, for example `20261027`). Each recommendation type has exactly one pointer item (`pk=META`, `sk=ACTIVE#FOR_YOU`, `ACTIVE#ALSO_BOUGHT` or `ACTIVE#BESTSELLER`) that says which version is live. Switching to a new version or rolling back means updating that one item, for every user at once. If a job stops halfway, the website never mixes two versions.
- **A version worse than the baseline is not activated.** The job measures itself; if the new version is not as good as the bestseller list, the old version stays.

## 3. Three ways to recommend

**Baseline: bestsellers.** Count the orders of each product, overall and per product category. When the website is new there are few orders, so the job uses the shortest window with at least 5 orders: 30 days, then 90 days, then 365 days. If none has enough, it ranks the newest products, and the pointer records how the list was ranked (`basis`) so the API knows. This list is used for new users with no history, and it is the benchmark for the two models below.

**"Frequently bought together": association rules.** For each pair of products A and B in the same order:

- support = orders with both A and B / all orders
- confidence(A→B) = orders with both A and B / orders with A
- lift(A→B) = confidence(A→B) / share of orders with B

A lift above 1 means people who buy A buy B more often than usual. I keep pairs that appear in at least 2 orders with a confidence of at least 0.05, then rank them by lift.

**"For you": item-based collaborative filtering.** Each product is a vector of scores per user: a view is worth 1, add to cart 3, a purchase 5. Two products are similar when the same people are interested in them, measured with cosine similarity. A user's recommendations are the products most similar to what they viewed, added to cart or bought, minus the products they already interacted with.

A small but important security detail: each (user, product) pair keeps **only the largest weight**, not the sum. Someone who views the same product 50 times still gives it only 1 point. An attacker therefore cannot push a product into recommendations just by spamming views. This is one mitigation for the "recommendation poisoning" risk in our threat model.

**No heavy libraries.** Lambda limits the unzipped code package to 250 MB, so all three methods use only the Python standard library, without pandas or scikit-learn. At demo scale (about 100 products, 600 users) the computation takes less than a second.

## 4. Evaluation

To evaluate fairly, train and test must not be split at random. They are **split by time**: the model learns only from data before a cut-off and is scored on data after it. This matches the real task of predicting the future from the past.

Three metrics:

- **precision@10:** out of 10 recommendations, how many are correct on average.
- **hit rate@10:** the share of recommendation lists with at least one correct item.
- **coverage:** the percentage of the catalog that is ever recommended. The baseline recommends almost the same list to everyone, so its coverage is very low.

Results on synthetic data (seed 42; 104 products; 896 training orders, 199 test orders):

| Recommendation | Method | precision@10 | hit rate@10 | coverage |
|---|---|---|---|---|
| Frequently bought together | Bestseller baseline | 0.0200 | 0.1905 | 0.106 |
| | Association rules | 0.0495 | 0.3667 | 0.654 |
| For you | Bestseller baseline | 0.0207 | 0.1957 | 0.164 |
| | Item-based CF | 0.0435 | 0.3804 | 0.942 |

Both models reach about twice the precision@10 of the baseline and recommend far more of the catalog. precision@10 looks low because each order has only 2–3 products, so each test case has only 1–2 correct answers among 10 recommendations.

## 5. Honest limits

- **The data is synthetic and has co-purchase patterns built in**, for example people who buy a graphics card often add a power supply. A model that finds those patterns proves the pipeline works, not that the model is better in real life.
- **Results vary between seeds.** With seeds 42, 1 and 7, item-based CF beats the baseline on all three metrics. With seed 2026, precision is still higher (0.0540 vs 0.0490) but hit rate is slightly lower (0.40 vs 0.42). Only 92 users are scored, so the numbers move easily.
- **The thresholds (2 orders, confidence 0.05, weights 1/3/5) are temporary.** The team will set them again once the website has real event data.

## 6. Done so far and next steps

Done:

- The `events` module records views and add-to-cart through `POST /api/v1/events`, and anonymises `userId` with HMAC before storing it (PRs #200, #204).
- The nightly pipeline core: writes the `recs` table by version and switches the ACTIVE pointer (PR #202).

Next:

1. Run the pipeline on Lambda on an EventBridge Scheduler schedule, with alarms when the job fails or does not run.
2. Record purchases (`PURCHASE`) from confirmed orders through an SQS queue.
3. A recommendation-poisoning experiment: create fake accounts that spam views, and measure the target product's rank before and after turning on the defences.

My biggest lesson: you do not need an expensive AI service to get useful recommendations. Start with a simple baseline, measure carefully, and replace it with something more complex only when the numbers show it is better.

*Code and numbers: the `data/` folder in [hhieuann/shop-ai](https://github.com/hhieuann/shop-ai), PRs #170, #172 and #202.*
