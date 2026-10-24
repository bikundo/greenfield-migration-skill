# Behaviour inventory — <old system> → <new system>

Disposition: `keep` = must match · `change` = deliberately different (describe) · `drop` = removed (name who agreed) · `?` = not yet decided (treat as a risk)

## Summary

| Surface | Items | keep | change | drop | ? |
|---|---|---|---|---|---|
| User-facing | | | | | |
| API / contracts | | | | | |
| Background | | | | | |
| Data | | | | | |
| Identity | | | | | |
| Integrations | | | | | |
| Operational | | | | | |

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

## Open questions

Things nobody on the team could answer yet, with who is expected to answer and by when.
