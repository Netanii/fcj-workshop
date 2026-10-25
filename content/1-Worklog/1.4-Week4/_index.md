---
title: "Week 4 Worklog"
date: 2026-10-11
weight: 4
chapter: false
pre: " <b> 1.4. </b> "
---

### Week 4 Objectives:

* Write the business documents for my part of shop-ai: how recommendations work and how behaviour events are recorded.
* Turn my synthetic data script into a proper Python package in the team repository, with tests running in CI.

### Tasks carried out this week:

| Day | Task | Start Date | Completion Date | Reference Material |
| --- | --- | --- | --- | --- |
| Tue | - Went to the office <br> - Read ADR-0004, ADR-0009, ADR-0012, ADR-0016 and the team's business-document template <br> - Check `contracts/openapi.yaml` for an events route and a recommendations route (neither existed yet) | 06/10 | 09/10 | `docs/adr/`, `docs/business/_template.md` |
| Fri | - Write `docs/business/recommendation.md`: where recommendations appear, how "Frequently bought together" (association rules) and "For you" (item-based collaborative filtering) are computed, the bestseller baseline, the `recs` output table, evaluation with precision@10 and coverage <br> - Write `docs/business/events.md`: event types, the rule that only the ordering module creates `PURCHASE`, HMAC anonymisation of `userId` <br> - Propose the key design of the `events` and `recs` tables in `dynamodb-design.md` | 09/10 | 10/10 | Issue #106, PR #107 |
| Sat | - Open and merge PR #107 into `develop` | 10/10 | 10/10 | PR #107 |
| Sun | - Package the synthetic data generator with uv (`pyproject.toml`, `uv.lock`), add a `--products` option to read the team's real product catalog <br> - Write 11 pytest tests: same seed gives the same data, no time leakage between train and test, cancelled orders create no `PURCHASE` event <br> - Enable the `uv` block in `.github/dependabot.yml` <br> - Open and merge PR #109 (CI runs ruff, mypy, pytest for `data/`) | 10/10 | 11/10 | Issue #108, PR #109 |
| Sun | - Merge three Dependabot PRs (#110, #111, #115) <br> - Two of them changed `pnpm-lock.yaml` and the lockfile broke after merging both; a teammate fixed it in #119 | 11/10 | 11/10 | PR #110, #111, #115, #119 |

### Week 4 Achievements:

* The team now has written business rules for recommendations and events, so the API, the data pipeline and the security work can be built against the same document.
* `data/` is a real Python package: one command (`uv run data`) generates reproducible synthetic data, and CI checks it with ruff, mypy and 11 tests on every pull request.
* Lesson learned: Dependabot PRs that touch the same lockfile must be merged one at a time, re-checking CI after each merge. The team rule since then is "merge Dependabot PRs one by one".

### Pending / next week:

* Agree with the team leader on the version and ACTIVE pointer design of the `recs` table.
* Write threat model v0 in `docs/security/`.
* Build the bestseller baseline and association rules offline and compare them with precision@10.
