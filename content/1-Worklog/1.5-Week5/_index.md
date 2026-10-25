---
title: "Week 5 Worklog"
date: 2026-10-18
weight: 5
chapter: false
pre: " <b> 1.5. </b> "
---

### Week 5 Objectives:

* Finalise the `recs` table design with the team leader.
* Deliver threat model v0 and a data classification table.
* Build both recommendation models offline and measure them against the bestseller baseline.

### Tasks carried out this week:

| Day | Task | Start Date | Completion Date | Reference Material |
| --- | --- | --- | --- | --- |
| Wed | - Went to the office <br> - Discuss the `recs` design with the team leader: one shared ACTIVE pointer per recommendation type instead of one pointer per user or product <br> - Read the v0.1.0 release checklist and confirm my acceptance tests C1–C7 (product browsing) | 14/10 | 14/10 | `docs/releases/v0.1.0.md` |
| Thu | - Update the `recs` design: pointer item `pk=META`, `sk=ACTIVE#FOR_YOU` / `ACTIVE#ALSO_BOUGHT`, zero-padded rank, numeric `ttl` (DynamoDB TTL needs epoch seconds), 7-day TTL with an extension rule so the active version never expires | 15/10 | 16/10 | Issue #165, PR #166 |
| Fri | - Write threat model v0 with STRIDE for three flows (sign-in, ordering, event ingestion) and a data classification table <br> - Check every "already in place" claim against the real code (Cognito settings, CloudFront, throttling, CI security checks) <br> - New findings: no CSP header on CloudFront while tokens sit in JavaScript-readable cookies; stock can be held by fake COD orders; the ordering role could change prices; the API Gateway URL bypasses WAF | 16/10 | 16/10 | Issue #167, PR #168 |
| Fri | - Implement association rules (support, confidence, lift) and the bestseller baseline in pure Python <br> - Write an offline evaluation: time-based train/test split, precision@10, hit rate@10, coverage <br> - 10 new tests | 16/10 | 16/10 | Issue #169, PR #170 |
| Sat | - Implement item-based collaborative filtering (cosine similarity). Each user–product pair keeps the maximum event weight instead of a sum, so spamming views cannot inflate a score <br> - Evaluate against the baseline on four random seeds; 7 new tests | 17/10 | 17/10 | Issue #171, PR #172 |
| Sat | - Send five design questions to the team (events route, how ordering reports purchases, HMAC key storage, data bucket encryption, threat model follow-ups); the team leader answered all of them the same evening | 17/10 | 17/10 | Team chat |
| Sun | - Prepare the `POST /api/v1/events` contract change (PR opened on 19/10, see week 6) | 18/10 | 18/10 | Issue #173 |

### Week 5 Achievements:

* Merged 4 pull requests (#166, #168, #170, #172), all with green CI.
* Both recommendation types now run offline and beat the bestseller baseline on the synthetic data (seed 42):

| Recommendation | Method | precision@10 | coverage |
| --- | --- | --- | --- |
| Frequently bought together | Bestseller baseline | 0.0200 | 0.106 |
| | Association rules | 0.0495 | 0.654 |
| For you | Bestseller baseline | 0.0207 | 0.164 |
| | Item-based CF | 0.0435 | 0.942 |

* Known limit: the synthetic data has the co-purchase rules built in, so these numbers show that the pipeline works, not that the model is better in real life. On one seed (2026) item-based CF had a slightly lower hit rate than the baseline, which is written down in `data/README.md`.
* Threat model v0 gives the team a shared list of risks; the team leader turned its open points into owners and dates on 17/10.

### Pending / next week:

* Merge the events route into the API contract after review.
* Write the infrastructure PR for the training-data bucket (SSE-KMS with the AWS managed key, bucket key on).
* Run acceptance tests C1–C7 on staging once the `release/v0.1.0` branch is cut (planned 16/10, not cut yet at the end of this week).
* Write Blog 1 (due 26/10).
