---
title: "Week 3 Worklog"
date: 2026-10-04
weight: 3
chapter: false
pre: " <b> 1.3. </b> "
---

### Week 3 Objectives:

* Understand the team project (shop-ai: a serverless electronics shop on AWS with AI-based co-purchase recommendations) and define my role: data, recommendations, and data & AI security.
* Prepare the data and security foundations for the recommendation feature: synthetic data, data model, and a first threat model.

### Tasks carried out this week:

| Day | Task | Start Date | Completion Date | Reference Material |
| --- | --- | --- | --- | --- |
| Mon | - Design and write a Python synthetic data generator: product catalog, 4 customer personas, co-purchase rules, noise, time-based train/test split <br> - Extend it with realistic cart behavior: cancelled orders, items removed from the cart, items saved in the cart for later | 28/09 | 02/10 | `generate_synthetic.py` |
| Tue | - Study the cost model of Amazon Personalize (real-time campaign minimum charge vs. batch inference) and the Free plan limitation <br> - 01/10: following the mentor's feedback, the team decided not to use Personalize and to build its own model on Lambda + DynamoDB (ADR-0016) | 29/09 | 01/10 | Amazon Personalize pricing page, ADR-0016 |
| Wed | - Read the team project plan and the review notes of 30/09 <br> - Clarify my scope with the team: data, recommendations, data & AI security | 30/09 | 30/09 | Repository README and `docs/project-plan.md` |
| Thu | - Went to the office <br> - Write the first draft of the STRIDE threat model for the recommendation pipeline: 6 trust boundaries, threats and proposed mitigations <br> - Plan the data-poisoning experiment for week 5 | 01/10 | 01/10 | `threat-model.md`, Microsoft STRIDE |
| Fri | - Design the data model (ERD) with dbdiagram.io: users, products, orders, order_items, cart_items, events, recs <br> - Define event types and the rule that only the order service can create `purchase` events <br> - Share the schema with teammates for the API contract | 02/10 | 02/10 | dbdiagram.io |

### Week 3 Achievements:

* Understood the project architecture and my responsibilities in the team.
* Understood the cost risk of Amazon Personalize; the team replaced it with a self-built model (association rules, item-based collaborative filtering, bestseller baseline) running on Lambda + DynamoDB (ADR-0016).
* Built a reproducible synthetic data generator (fixed random seed) that produces:
  * products, users, orders, order items, cart items;
  * interaction events in a format compatible with Amazon Personalize;
  * a time-based train/test split to avoid data leakage.
* Designed a 7-table data model and agreed on the event rules with teammates, including protection against fake `purchase` events.
* Wrote a first threat model covering the browser-to-API boundary, the training-data bucket, Personalize, and the recommendation table, with data poisoning and IDOR as the main risks I will test.

### Pending / next week:

* Build the rule-based baseline (support, confidence, lift) and evaluation metrics (precision@k, recall@k, coverage).
* Write the HMAC-based `user_id` anonymization function.
