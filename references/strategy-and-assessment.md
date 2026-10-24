# Strategy, assessment and risk

Read this in Phase 0 to choose the migration shape and set up the brief: the strategy table, the assessment checklist, success criteria, the risk register and estimating.

## Contents
1. Strategy options
2. Assessing the current system
3. Target state and gap analysis
4. Success criteria
5. Risk register
6. Estimating and cadence

## 1. Strategy options

A "greenfield rewrite" can still be delivered in different ways. Choose deliberately and record why.

| Strategy | What changes | Good fit | Main risk |
|---|---|---|---|
| Rehost (lift and shift) | Where it runs; code mostly unchanged | Infrastructure deadline, hosting cost, platform end of life | Carries every old problem across; little value beyond the move |
| Replatform | Runtime, datastore or managed services; application logic mostly kept | The old platform is the pain, but the logic is sound | Hidden coupling to old platform behaviour (SQL dialects, file paths, time zones, locale) |
| Strangler fig | New system grows beside the old one, behind a shared front door (proxy, API gateway, or app shell); routes move over one at a time | Business-critical, many integrations, low risk tolerance, long-lived product | Coexistence lasts a long time; needs data sync and discipline about which side owns what |
| Phased by cohort, tenant or region | Whole new system, but users move in groups | Users or tenants can be separated cleanly (by customer, organisation, region, or app version) | Running both systems in production at once; cross-cohort interactions (a user in cohort A transacting with an account in cohort B) |
| Big bang | Everyone switches in one event | Small, simple system; acceptable downtime window; tests cover behaviour well | No gradual rollback; every unknown shows up at once |

Rough guide:
- High criticality (money, identity) or more than a handful of integrations → strangler fig or phased. Big bang needs a written reason.
- The old system is small and well tested, and a maintenance window is acceptable → big bang can be the cheapest honest option.
- A mobile app rewrite is usually **phased by app version** in practice, because users update over weeks. Plan the backend for both versions at once (see `mobile-apps.md`).

When using strangler or phased delivery, order the slices by risk:
1. Read-only, low-stakes surfaces (catalogues, content, profile views)
2. Reporting and analytics read paths
3. Low-risk writes (preferences, profile edits)
4. Core writes (orders, bookings, submissions, the records the business runs on)
5. Money movement and partner integrations, last, once the new platform has a track record

## 2. Assessing the current system

Capture, briefly, in the brief or a short `00-assessment.md`:

- **Stack and hosting:** languages, frameworks and versions, datastores, infrastructure, third-party services.
- **Shape:** monolith, services or batch; number of integrations; data volume; transaction volume; user counts (total, daily active, peak concurrent).
- **Criticality:** what breaks for whom if it is down for an hour, and what the revenue or regulatory exposure is.
- **Compliance:** data-protection law where users live (GDPR, CCPA, or the local data-protection act), PCI-DSS scope if cards are touched, financial-services rules, sector rules (health, education, public sector), retention obligations. These shape data handling, hosting location and the decommission phase.
- **Health:** test coverage, documentation quality, known tech debt and security issues, who still understands which parts.
- **Baseline metrics:** measure *before* building, from real production data: latency (p50/p95) on key endpoints and screens, error rate, availability, crash-free rate and start-up time for apps, conversion and payment success rates, support ticket volume. Without a baseline, "the new one is fine" can't be shown or disproved.
- **Why now:** the migration driver (end of support, cost, performance, velocity, compliance, a product the old architecture can't hold).

## 3. Target state and gap analysis

Describe the target stack and architecture in a few lines, then list gaps:

| Capability | Today | Target | Gap / work needed | Priority |
|---|---|---|---|---|
| e.g. Auth | custom JWT | managed provider | token migration, hash compatibility | High |

Include **people gaps**: skills the team needs for the target stack, who is learning or hiring, and which experts on the old system must be interviewed before they move on. Knowledge that is only in someone's head is a migration risk. Capture it in the inventory early.

## 4. Success criteria

Define how you'll know the migration worked, in three groups, each with a measurable target relative to the baseline:

- **Technical:** error rate, latency, availability, crash-free rate no worse than baseline (or a stated improvement), sustained at peak load.
- **Business:** conversion, payment success, retention, support tickets per thousand users not worse; the specific improvements the rewrite was meant to unlock.
- **Project:** date, budget, scope agreed in the inventory (`keep`/`change`/`drop`), old system decommissioned and its costs stopped.

Put the thresholds in the brief. Phase 5's rollout gates and Phase 6's go/no-go use the same numbers.

## 5. Risk register

Keep a short register in the README (top five visible, the rest linked). One row per risk:

| Risk | Likelihood | Impact | Mitigation (before) | Contingency (if it happens) | Owner |
|---|---|---|---|---|---|

Common rows to consider (keep only those that apply, made specific):
- Undocumented dependency discovered late (the most common cause of slipped cutovers) → deeper inventory, traffic analysis, shadow run.
- Data loss or corruption → backups with tested restore, idempotent scripts, reconciliation, reverse sync.
- Performance regression under real load → load test at about 2x expected peak plus a soak test, gradual traffic increase.
- Integration failure (payment provider, messaging, identity verification, partner API) → sandbox tests, keep the old integration live during transition, circuit breakers.
- Users stuck or confused → in-app messaging, support briefed with the `change` list, gradual rollout.
- Key person unavailable → interviews recorded early, runbook ownership shared, rehearsal run by someone other than the author.
- Cutover overruns the window → rehearsed runbook with measured timings, hard go/no-go time.

## 6. Estimating and cadence

- Rewrites routinely take well over the first estimate. A common rule of thumb is 1.5–2x. Build that buffer into the plan explicitly rather than hoping, and treat the inventory size as the best early predictor.
- Status cadence: weekly to stakeholders while planning and building; daily (or more) around cutover.
