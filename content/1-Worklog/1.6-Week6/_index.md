---
title: "Week 6 Worklog"
date: 2026-10-23
weight: 6
chapter: false
pre: " <b> 1.6. </b> "
---

> In progress. Filled in up to Friday 23/10.

### Week 6 Objectives:

* Start Sprint 2 (20/10–02/11): the events module, the training-data bucket and the nightly pipeline.
* Run acceptance tests C1–C7 on staging for release v0.1.0.
* Prepare Blog 1 and the workshop site for this report.

### Tasks carried out this week:

| Day | Task | Start Date | Completion Date | Reference Material |
| --- | --- | --- | --- | --- |
| Mon | - Add `POST /api/v1/events` to `contracts/openapi.yaml`: login required, body only `{ productId, eventType }` with `VIEW` or `ADD_TO_CART`, `additionalProperties: false` so `userId`, timestamps and `PURCHASE` are rejected <br> - Regenerate the web API types and lint the contract (valid, no new warnings) <br> - Open PR #174 and ask the teammate who owns the contract to review <br> - PR #174 merged into `develop` by the team leader | 18/10 | 19/10 | Issue #173, PR #174 |
| Mon | - Review Dependabot PR #175 (ruff 0.17.0 → 0.17.1 in `data/`): ran ruff, mypy and 28 tests on the PR branch, approved | 19/10 | 19/10 | PR #175 |
| Tue | - Add the training-data S3 bucket in `infra/`: SSE-KMS with the AWS managed key `aws/s3`, bucket key on, public access blocked, TLS required, versioning with 30-day expiry of old versions <br> - 6 CDK unit tests; typecheck, lint, 60 infra tests and `cdk synth` with cdk-nag pass <br> - Read the CI `cdk diff` against staging and explain every change to the reviewer <br> - Open PR #179; the team leader reviewed and merged it into `develop` | 20/10 | 21/10 | Issue #178, PR #179 |
| Wed | - Security review of the GitHub Actions OIDC trust policy (`infra/bootstrap/github-oidc.yaml`): trust policies match `aud` and `sub` exactly, but the deploy roles can assume the CDK bootstrap execution role, which has AdministratorAccess, and dev, staging and prod share one AWS account <br> - The team leader checked the GitHub environment rules and chose to accept the risk | 21/10 | 21/10 | Issue #180 |
| Wed | - Review and approve PR #182 (pre sign-up Lambda timed out on cold start: 512 MB, 5 s); ran the infra tests on the PR branch (22/22) | 21/10 | 21/10 | PR #182 |
| Wed | - Acceptance tests C1–C7 on staging for v0.1.0: 5/7 passed <br> - Opened bug #184 (detail page had no specifications or image) and #185 (cart icon shows no item count) <br> - Reviewed and approved the fix PR #190, then re-tested on staging: specifications now shown | 21/10 | 21/10 | Issues #184, #185, PR #190 |
| Wed | - Update the threat model: new risk C6 (deploy role is account-wide admin, accepted, from #180) and E9 (fake messages on the `order-confirmed` topic could create fake purchases); record the team decisions of 17/10 and owners | 21/10 | 21/10 | Issue #186, PR #187 |
| Wed | - Data generator follows the team leader's review: real product catalog by default, upper-case event types like the API, one ISO 8601 UTC time format, a 10% group of new users with no training history, UTF-8 output on Windows <br> - 10 new tests (38 in total); re-ran both offline evaluations on the real catalog: models still beat the bestseller baseline on 4 seeds | 21/10 | 21/10 | Issue #188, PR #189 |
| Thu | - Set up my laptop for integration tests: WSL 2, Docker Desktop (auto-start off to save RAM), AWS CLI | 22/10 | 22/10 | Docker docs |
| Thu | - Update threat model items O6, O8 and B6 at a teammate's request | 22/10 | 22/10 | Issue #197, PR #198 |
| Thu | - Events module (`POST /api/v1/events`): hexagonal domain, application and DynamoDB adapter; `userId` replaced by an HMAC-SHA256 hash with the key from SSM Parameter Store; event IDs are ULIDs; items expire after 90 days with DynamoDB TTL <br> - Unit tests and integration tests against DynamoDB Local in Docker | 22/10 | 22/10 | Issue #199, PR #200 |
| Thu | - Nightly pipeline core in Python: versioned `recs` rows, the ACTIVE pointer that switches all users at once, a version worse than the baseline is not activated, "For you" scores per user | 22/10 | 22/10 | Issue #201, PR #202 |
| Thu | - Infrastructure for the events module in CDK: Lambda, `events` table, API route, read access to only the one SSM key <br> - Read the CI `cdk diff` against staging: found a pre sign-up change that would have been reverted (fixed by the team leader's release merge-back) and the AWS account ID being visible in the CI summary (low risk, reported) | 22/10 | 22/10 | Issue #203, PR #204 |
| Thu | - Write down the C1–C7 results in the release checklist | 22/10 | 22/10 | Issue #218, PR #220 |
| Thu | - Move the `userId` hash function to a shared place so the recommendation API hashes the user exactly like the events module (no cross-module import) | 22/10 | 22/10 | Issue #222, PR #225 |
| Thu | - Review the team leader's recommendation API core (PR #224) and agree on the `recs` format: column `productId`, up to 20 rows per anchor, a `BESTSELLER` type with a 30/90/365-day fallback and a `basis` field | 22/10 | 22/10 | PR #213, PR #224 |
| Fri | - Change the pipeline to that `recs` format and add the bestseller job; record the cross-module table access in ADR-0017 | 22/10 | | `recs` format PR |
| Fri | - Create this workshop site from the FCAJ template, deploy it on GitHub Pages, write the worklog in English and Vietnamese | 23/10 | 23/10 | [fcj-workshop](https://github.com/Netanii/fcj-workshop) |

### Week 6 Achievements:

* The events module is merged with its infrastructure: the website can record views and add-to-cart events, and stored events never contain the real user ID.
* The nightly pipeline core is merged; the recommendation API and the pipeline now share one hash function and one table format.
* Release v0.1.0 reached production after my acceptance tests on staging; bug #184 that I found was fixed and re-tested.
* [Complete at the end of the week.]
