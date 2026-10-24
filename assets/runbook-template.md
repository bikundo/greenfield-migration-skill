# Cutover runbook — <migration name>

- **Window:** <date, start–end, timezone>  ·  **Freeze starts:** <time>
- **Incident lead / go-no-go owner:** <name>  ·  **Comms owner:** <name>
- **Channel:** <where the team coordinates>  ·  **Status page / user comms:** <where>
- **Rehearsed on:** <date, environment, total duration measured>

## Readiness checklist

Every item has an owner and is ticked before the window opens.

- **Infrastructure:** target environment provisioned · network, firewall and security rules · TLS certificates · DNS records prepared (not switched) · load balancers / gateways configured
- **Application:** release built and deployed to target (dark) · config and secrets set for production · store builds approved and held for release (mobile)
- **Data:** initial load done · reconciliation passed · delta sync running and tested · final-sync steps scripted
- **Testing:** blocking parity rows green · load and soak results within targets · user-acceptance sign-off · security scan passed
- **Operations:** dashboards for gate metrics · alerts wired to on-call · runbooks written · escalation path known
- **Comms:** stakeholders, partners and users notified · support briefed on what changed · status page ready
- **Rollback:** procedure written · rehearsed on staging with timings · triggers agreed · rollback owner named

## Pre-flight (T-7d → T-1h)

| # | When | Owner | Action | Done when |
|---|---|---|---|---|
| P1 | T-7d | | Users, support and partners notified | |
| P2 | T-1d | | All blocking parity rows green (link 02-parity.md) | |
| P3 | T-1d | | Rollback rehearsed; durations recorded | |
| P4 | T-1h | | Backups/snapshots taken and restore tested | |

## Cutover steps

| # | Time | Owner | Action | Expected result | Verify by | If it fails | Reversible? (how, how long) |
|---|---|---|---|---|---|---|---|
| C1 | | | | | | | |

### Go / no-go checkpoint — before step C<n> (point of no return)

Proceed only if all are true:
- [ ] Reconciliation queries pass (counts, sums, hashes)
- [ ] Smoke checks pass (sign in, the core user flow, a real payment if applicable, a real notification)
- [ ] Error rate / latency within <threshold>
- [ ] <other criteria from 02-parity.md>

Decision recorded by <owner> at <time>: GO / NO-GO.

## One-way doors

Steps that can't be undone, and what was done to delay or soften each one.

| Step | Why it is irreversible | Mitigation |
|---|---|---|
| | | |

## Rollback plan

Trigger conditions (any one), agreed in advance:
- A critical flow is broken (sign-in, the core user flow, payment, …)
- Data corruption or reconciliation failure detected
- Gate metrics past threshold for <duration> (from 00-brief.md)
- Security issue discovered
- The issue can't be fixed inside the window

Decision owner: <name>. After any rollback: review within 24h (root cause, plan changes, new date).

| # | Owner | Action | Duration (rehearsed) | Verify by |
|---|---|---|---|---|
| R1 | | | | |

## Post-cutover (T+0 → T+14d)

| # | When | Owner | Check |
|---|---|---|---|
| Q1 | T+1h | | Smoke checks repeated with real users |
| Q2 | Daily | | Reconciliation report |
| Q3 | T+14d | | Decision: proceed to decommission (07-decommission.md) |
