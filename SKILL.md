---
name: greenfield-migration
description: Plan and run a greenfield rewrite — rebuilding an existing app or service from scratch on a new stack, language, framework or architecture and moving real users, data and traffic onto it without losing behaviour, money or trust. Covers deciding whether a rewrite is warranted, inventorying what the old system actually does, a parity checklist, data migration and reconciliation, identity/auth/session carry-over, running old and new side by side, the cutover runbook, rollback, and decommissioning. Use whenever the user talks about a rewrite, rebuild, "v2", replatforming, porting to a new language, framework, platform, backend or database, moving off a legacy system, sunsetting an old app, a big-bang or phased cutover, or asks "how do we move users to the new system" — even if they never say "greenfield" or "migration". Not for routine dependency/SDK upgrades or a single schema migration inside one system.
---

# Greenfield migration

A rewrite fails in predictable ways: the new system quietly drops behaviour nobody wrote down, data arrives subtly wrong, users get logged out or lose balances, and there is no way back once the switch is flipped. This skill exists to make those failures visible *before* cutover. The output is a plan the team can execute and check off, not an essay.

The skill is independent of language, framework and platform. It applies to a backend service, a web app, a mobile app, a data pipeline or a whole product, and to any old-to-new pairing. Where a reference file goes platform-specific (mobile stores, for example) it says so.

Work through the phases in order. Each one produces a concrete file. Skip a phase only when you can say why it doesn't apply, and write that reason down; the reason is useful to the reader.

## Working principles

- **The old code is the spec.** Read it before trusting docs, tickets or memory. Where you have access to traffic, logs or usage data, read those too.
- **Name real things.** Files, endpoints, tables, jobs, owners. A plan with placeholders is a plan nobody can check. When you can't find something, write `unknown` and list it as an open question rather than inventing a plausible name.
- **Prefer the gradual path.** Default to a rollout that can be reversed per user, per cohort or per route. Big bang needs a stated reason.
- **Measure, don't assert.** Parity, data correctness and readiness are shown by tests, diffs and reconciliation queries, not by "looks fine".
- **Ask when it matters, decide when it doesn't.** Anything that changes money handling, data ownership, security or the order of work is the user's call; ask one question at a time with a recommendation attached. Routine choices with a conventional answer don't need a question.
- **Scale to the system.** A small internal tool may need half a page per phase. A payments or identity system needs all of it, with reconciliation and rollback rehearsed.

If you are working from the user's description alone, with no code to read, still produce the files. Mark every fact you could not verify as an assumption, and put the assumptions at the top of the brief so they are the first thing the team checks.

## Phase 0 — Frame it and challenge it

Read [references/strategy-and-assessment.md](references/strategy-and-assessment.md) for the strategy table, assessment checklist, success criteria and risk register format.

Get these facts first, from the user and the code. Ask only for what you can't find.

- **What is being replaced, by what?** Old stack → new stack, repositories, hosting, who owns each.
- **Why rewrite instead of evolve?** Push on this once. An incremental replacement (strangler fig: route one screen or endpoint at a time to the new system behind the old front door) is usually lower risk. A full rewrite is justified when the old platform itself is the blocker (dead framework, unsupportable runtime, platform-policy or compliance deadline, an architecture that can't hold the next product). Record the reason; if it is weak, say so and propose the incremental route, then follow the user's decision.
- **What can't break?** Money movement, authentication, regulatory records, partner integrations, public URLs and deep links, notification delivery. These get the strictest parity and the slowest rollout.
- **Compliance:** data-protection, payments and retention rules that apply. They constrain hosting, data handling and what can be deleted at the end.
- **Constraints:** deadline and its source, team size and skill gaps on the new stack, freeze windows (a peak sales period, month-end close, a regulatory reporting date), who signs off on go/no-go.
- **Shape of the delivery:** strangler fig, phased by cohort/tenant/region/app version, or big bang. Default to gradual.
- **Baseline and success criteria:** measure today's production numbers (latency, error rate, crash-free rate, conversion, payment success, support volume) *before* building, and state the targets relative to them. These numbers become the rollout gates and the go/no-go criteria later, so get them early.
- **Top risks:** start the risk register now (likelihood, impact, mitigation, contingency, owner).
- **Estimate honestly:** rewrites routinely run 1.5–2x the first estimate. Put the buffer in the plan.

Write `00-brief.md` (one or two pages): goal, why-rewrite and chosen strategy, non-negotiables, compliance, constraints, baseline and success criteria, top risks, decision owners, open assumptions.

## Phase 1 — Inventory what the old system actually does

The old code is the spec, including the bugs users now depend on. Docs and memory are incomplete; read the code and, where you can, the traffic.

Sweep every surface. For a large codebase, split the sweep by surface and run the parts in parallel if your environment supports subagents; otherwise work through the table top to bottom.

| Surface | Look for |
|---|---|
| User-facing | Every screen/route, empty/error/loading states, deep links and public URLs, notifications (push, SMS, email) and their triggers, share links, offline behaviour, accessibility and localisation |
| API / contracts | Every endpoint the clients call (and which client versions call them), webhooks in and out, response shapes clients rely on, error codes clients branch on, rate limits and auth schemes |
| Background | Scheduled jobs, queues and consumers, retries, scheduled reports, cleanup tasks, reconciliation jobs |
| Data | Tables/collections, implicit invariants (uniques, soft deletes, status machines), derived or cached values, files and blobs, retention rules |
| Identity | How users sign up and sign in, session/token lifetimes, password hashing scheme, OTP/2FA, roles and permissions, account linking, API keys issued to others |
| Integrations | Payment providers, identity verification, messaging providers, analytics events (names matter: dashboards depend on them), crash reporting, CRM, feature-flag service, partner APIs and their credentials, allow-listed IPs and callback URLs |
| Operational | Admin and back-office tools, support workflows that read this data, alerts and dashboards, runbooks, exports other teams depend on |

Code isn't the only source. Ask the people who run and support the old system (support agents, operations, the engineer who wrote the billing job) what breaks and what users complain about. Do it early, before they move on; their knowledge is often in no file. Check usage data too: features nobody uses are candidates to drop instead of rebuild.

For each behaviour record **what** it does, **where** (file and line, or endpoint), **who depends on it**, and a **disposition**: `keep` (must match), `change` (deliberately different; say how), or `drop` (say who agreed). Anything not yet classified is a risk, not a default `drop`.

Write `01-inventory.md` using [assets/inventory-template.md](assets/inventory-template.md). Expect this to be the longest file; that is fine.

## Phase 2 — Parity contract

Turn every `keep` and `change` row into something checkable. "Works the same" is not checkable.

- Pick the verification per row: contract test against both systems, golden-file or snapshot comparison, recorded-traffic replay, screenshot diff, scripted manual check, or reconciliation query.
- Prefer tests that run against **both** old and new, so a disagreement shows up as a diff instead of an opinion.
- Mark which rows **block cutover** and which can follow after. Non-negotiables from Phase 0 always block.
- Note anything that can only be verified in production (real payment callbacks, real notification delivery). Those go into the cutover runbook as smoke checks.
- Beyond functional parity, plan the test types that catch rewrite-specific failures: **load** (about 2x expected peak, including known seasonal or campaign spikes), **soak** (sustained load for hours to surface leaks and connection exhaustion), **compatibility** (old client versions, partner APIs, the devices, browsers and OS versions users actually have), and **user acceptance** with real users or staff on real tasks.
- Decide where test data comes from: an anonymised production copy for final validation and performance, a representative subset for debugging, synthetic data for development. Production data in test environments follows the same data-protection rules as production.

Write `02-parity.md`: a checklist table (behaviour, verification method, blocking?, status) plus the test plan and test-data sources.

## Phase 3 — Data migration

Read [references/data-migration.md](references/data-migration.md) before writing this phase. Core rules:

- **Don't move code and data in the same step.** If the new code and the new datastore go live together and numbers are wrong, you can't tell which caused it. Stage it: new code reads a synced copy while the old system stays authoritative, or data moves while the old code still runs. Then switch one thing at a time.
- **Clean before you copy.** Profile the old data (duplicates, invalid values, orphans, formats) and fix what you can at the source, so the new system doesn't inherit the mess or reject it at cutover.
- **Choose the sync strategy explicitly:** one-shot copy during a freeze, backfill plus dual-write, or backfill plus change-data-capture. The longer old and new must coexist, the more you need continuous sync.
- **Keep identifiers stable.** Carry old primary keys and user IDs across (as the key or a mapped column) so links, receipts, support records and analytics still join.
- **Scripts are idempotent and re-runnable**, run first against a production snapshot, with timing measured. The measured time decides the freeze window.
- **Reconcile, don't eyeball:** row counts per entity, checksums of key fields, and for money, sums per account and in total to the smallest unit. Define the reconciliation queries before migrating.
- Decide what happens to records written to the old system *after* the copy.

Write `03-data.md`: entity mapping (old → new, transforms), sync strategy, source of truth per entity per stage, reconciliation queries and pass criteria, rehearsal results.

## Phase 4 — Identity, sessions and clients in the wild

The fastest way to lose users is to log them all out or break the version they already have installed.

- **Credentials:** can the new system verify existing password hashes as-is? If not, verify with the old scheme and rehash on next login. Never force a mass reset without a recorded decision.
- **Sessions and tokens:** can the new system accept existing tokens (shared signing keys, a token-exchange endpoint)? If not, plan a silent re-authentication path.
- **Old clients keep calling.** Installed apps, cached web bundles and partner integrations will hit the old endpoints for months. Decide: keep a compatibility layer, proxy old routes to new, or enforce a minimum version (with the in-app prompt live *before* cutover).
- **Third-party identities:** OAuth client IDs, social sign-in identifiers (often tied to a developer account or app identifier), push tokens, API keys you issued to partners, webhook callback URLs registered with partners.

For mobile apps, read [references/mobile-apps.md](references/mobile-apps.md). App identifiers, signing keys, on-device storage, store review timing and forced updates are where mobile rewrites get hurt.

Write `04-identity.md`.

## Phase 5 — Run old and new side by side

Build confidence with real traffic before users depend on the new system.

- **Shadow or dark launch:** send real reads (or copies of requests) to the new system, compare results, discard its output. Log diffs; triage them against the parity file.
- **Dual-run for money and records:** if the new system computes balances or fees, run it in parallel and reconcile daily before it becomes the source of truth.
- **Cohort rollout:** internal staff → a small percentage → widening (for example 1% → 10% → 25% → 50% → 100%), with a flag that routes a user to old or new and can be flipped back per user. Each step holds long enough to see a full traffic cycle, including the busiest period the business has, not just an hour. The gate metrics and thresholds come from the Phase 0 baseline and success criteria. Write down the threshold that triggers moving back a step.
- **Slice order** (for strangler or phased delivery): read-only surfaces first, then low-risk writes, then core writes, with money movement and partner integrations last.
- **One source of truth at a time** per entity. Two systems both accepting writes for the same record without a defined sync is how data forks.

Write `05-parallel-run.md`: what is shadowed, the diff triage process, cohort steps with gate metrics.

## Phase 6 — Cutover runbook and rollback

Read [assets/runbook-template.md](assets/runbook-template.md). The runbook is a timed, owned, step-by-step script someone can follow at 2am.

- Each step: owner, action, expected result, how to verify, what to do if it fails.
- An explicit **go/no-go checkpoint** before the point of no return, with the criteria from Phase 2.
- **Rollback, per step:** how to undo it and how long that takes. Identify the **one-way doors**, the things that can't be undone (a store release users have installed, emails or SMS sent, money settled, data only written to the new system, DNS TTLs). For each, either make it reversible (reverse sync, keep the old system writable, keep the old binary deployable) or move it as late as possible and say so loudly.
- **Readiness checklist** by area: infrastructure, application, data, testing, operations (dashboards, alerts, on-call), communications, rollback. Every item has an owner and is ticked before the window opens.
- **Rollback triggers**, decided in advance so nobody argues at 2am: critical flow broken, data corruption detected, metrics past threshold, security issue, or the window about to be exceeded. Name the one person who calls it.
- Communications: who tells users, support and partners, before, during and after. Support needs the "what changed" list from Phase 1's `change` rows and should be briefed before launch.
- Schedule the window at the lowest-risk time for *this* business (not a sales peak, a month-end close or a live campaign), with the team on hand.
- Rehearse the full runbook, including rollback, on staging with production-shaped data. Record actual timings.
- If you roll back, run a short review within a day: root cause, plan changes, new date. A rollback that is learnt from isn't a failure of the migration.

Write `06-cutover-runbook.md`.

## Phase 7 — Decommission

The migration isn't finished while the old system can still surprise you. Don't declare victory early; stabilisation takes weeks.

- **Stabilise:** watch closely for the first 24–72 hours (extra support on duty, rollback still available), review metrics daily for the first week, then weekly for the first month. Compare against the Phase 0 success criteria and say plainly which were met.
- Keep the old system read-only for a defined period (typically 30–90 days); know what would make you need it.
- Archive data you're legally or operationally required to keep, with a documented way to query it.
- Remove: credentials, partner allow-lists and callbacks pointing at old infrastructure, DNS, scheduled jobs, billing for old hosting, old app listings or their redirects.
- Confirm traffic to old endpoints is zero (or only from versions you've agreed to stop serving) before shutting them down.
- **Check costs:** confirm old licences, hosting and third-party plans are cancelled, and report actual savings if cost was a driver.
- **Hand over knowledge:** architecture, runbooks, troubleshooting notes and dashboards for the new system, with support and operations trained on it.

Write `07-decommission.md`.

## Output

Put the files in `docs/migration/<short-name>/` in the repository the user is working in, or wherever they ask. Start with `README.md`: one paragraph on the migration, the current phase, the top five open risks, and links to each phase file. Keep it updated as phases progress; it is what someone new reads first.

```
docs/migration/<short-name>/
├── README.md
├── 00-brief.md
├── 01-inventory.md
├── 02-parity.md
├── 03-data.md
├── 04-identity.md
├── 05-parallel-run.md
├── 06-cutover-runbook.md
└── 07-decommission.md
```

Write for the whole team: a product lead should be able to read the brief and README; engineers need the inventory, parity and runbook detail.

When the plan is ready, offer to break the phases into tickets in whatever tracker the team uses, with each blocking parity row and the runbook rehearsal as tickets of their own.

## Partial use

If the user only wants one phase ("just help me inventory the old app", "write the cutover runbook"), do that phase well and list what the other phases will need from it. If the user is mid-migration, read what they already have, slot it into the phase structure, and fill the gaps rather than starting over.
