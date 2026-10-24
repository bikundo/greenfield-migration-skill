# Behaviour inventory — <old system> → <new system>

Disposition: `keep` = must match · `change` = deliberately different (describe) · `improve` = same purpose, better implementation (describe) · `add` = new behaviour (name who asked) · `drop` = removed (name who agreed) · `?` = not yet decided (treat as a risk)

For `improve` and `add`, the Notes column says whether it ships with the migration or after it.

## Summary

| Surface | Items | keep | change | improve | add | drop | ? |
|---|---|---|---|---|---|---|---|
| User-facing | | | | | | | |
| API / contracts | | | | | | | |
| Background | | | | | | | |
| Data | | | | | | | |
| Identity | | | | | | | |
| Integrations | | | | | | | |
| Operational | | | | | | | |

## User-facing

| # | Behaviour | Where (file:line / route) | Depends on it | Disposition | Notes |
|---|---|---|---|---|---|
| U1 | | | | | |

## API / contracts

| # | Endpoint / webhook | Callers (incl. client versions) | Contract details clients rely on | Disposition | Notes |
|---|---|---|---|---|---|
| A1 | | | | | |

## Background

| # | Job / trigger | Schedule / trigger | Effect | Disposition | Notes |
|---|---|---|---|---|---|
| B1 | | | | | |

## Data

| # | Entity / table / collection | Rows (approx.) | Invariants & quirks | Disposition | Notes |
|---|---|---|---|---|---|
| D1 | | | | | |

## Identity

| # | Mechanism | Detail (hash scheme, token lifetime, provider IDs) | Disposition | Notes |
|---|---|---|---|---|
| I1 | | | | |

## Integrations

| # | Service | What we send / receive | Credentials & registered URLs | Disposition | Notes |
|---|---|---|---|---|---|
| X1 | | | | | |

## Operational

| # | Tool / process | Who uses it | Disposition | Notes |
|---|---|---|---|---|
| O1 | | | | |

## Surprises found

Behaviour that wasn't documented or that users rely on despite looking like a bug. Each one needs an explicit disposition above.

## Gaps and improvements

Problems found in the old system while sweeping, and new behaviour the rewrite should add. Each links to the row above it affects (or is a new row with disposition `add`).

| # | Gap | Where | Evidence (bug report, metric, code) | Proposed improvement | Ships with migration? | Decided by |
|---|---|---|---|---|---|---|
| G1 | | | | | | |

## Open questions

Things nobody on the team could answer yet, with who is expected to answer and by when.
