# Data migration reference

Read this when writing Phase 3. It covers choosing a sync strategy, writing the scripts, the conversions that quietly corrupt data, and reconciliation, with extra rules for ledgers and payments. Nothing here assumes a particular database; the same rules apply to relational, document, key-value and file-based stores.

## Contents
1. Profiling and choosing a sync strategy
2. Script rules
3. Transform pitfalls
4. Money, ledgers and in-flight payments
5. Reconciliation
6. Rehearsal

## 1. Choosing a sync strategy

| Strategy | How it works | Use when | Cost |
|---|---|---|---|
| One-shot copy in a freeze | Stop writes to old, copy everything, switch | Data is small enough that the copy fits an acceptable downtime window (measure it) | Downtime; rollback means copying back anything written to new |
| Backfill + dual-write | Copy history; the application writes to both old and new from then on | You control the write path and coexistence is short | Dual-write bugs: a write succeeds in one and fails in the other. Needs a repair job |
| Backfill + change data capture (CDC) | Copy history; stream changes from the old store's log or triggers into new | Long coexistence, many writers, or you can't touch the old write path | More infrastructure; ordering and idempotency must be handled |
| Reverse sync | New → old after cutover | You need rollback to be possible after new has taken writes | Same problems in the other direction; worth it for money and accounts |

Most databases offer some form of CDC (logical replication, a change stream, triggers, or a dedicated connector). Check what the old store supports before choosing.

Write down which system is the **source of truth for each entity at each stage**. A table of entity × stage is the clearest form.

### Before choosing: profile and clean

Profile the source data first: volumes per entity, null rates, duplicate candidates, invalid formats, orphans, values outside the expected enums. Fix what can be fixed **in the old system** before migrating (merge duplicates, correct formats, archive obsolete records). Cleaning during the transform hides decisions in script code; cleaning at the source leaves them visible and reversible. The profiling output also gives you the migration-duration estimate.

## 2. Script rules

- **Idempotent:** running twice yields the same result. Upsert on a stable key; never blind-insert.
- **Resumable:** process in batches ordered by a stable key, and checkpoint the last key so a crash doesn't restart from zero.
- **Observable:** log counts read, written, skipped and failed per batch. Write rejected rows (with the reason) to a quarantine table or file instead of dropping them.
- **Throttled:** don't take down the old production database while reading from it. Read from a replica or snapshot when possible.
- **Deterministic transforms:** the same input always gives the same output (no "current time" in transforms, so timestamps are preserved, not reset).
- **Versioned with the code** and reviewed like code.

## 3. Transform pitfalls

These cause most "the numbers are slightly off" incidents:

- **Money as floats.** Store minor units (cents) as integers, or a fixed-precision decimal. Convert once, with an explicit rounding rule, and reconcile.
- **Time zones.** Know whether old timestamps are UTC, server-local, or naive. Normalise to UTC, but preserve the original offset where the wall-clock time is the meaning (an appointment at 7pm local time is a local time, not an instant).
- **Phone numbers and emails.** Normalise (E.164 for phones, lower-case for emails), but **detect collisions first**: two old accounts that normalise to the same identifier need a merge decision, not a silent overwrite.
- **Text.** Encoding (latin1 vs utf8), Unicode normalisation, trimmed whitespace, length limits in the new schema truncating data.
- **Nulls vs empty vs missing.** Document-store to SQL migrations (and the reverse) collapse these differently.
- **Enums and status machines.** Map every old value, including legacy ones no longer produced but still present in data. Unmapped value = fail loudly.
- **Soft-deleted and orphaned rows.** Decide whether they migrate. Orphans (foreign-key targets missing) are common in old systems without constraints.
- **Files and blobs.** URLs embedded in records (avatars, generated documents, images) that point at old storage. Either copy and rewrite the URLs, or keep the old storage serving.
- **Derived values.** Prefer recomputing them in the new system and comparing with the old stored value. Any difference is a finding.
- **Auto-increment and sequence gaps.** If the new store generates its own IDs, reserve the old ID range or map explicitly, so nothing new collides with something migrated later.

## 4. Money, ledgers and in-flight payments

- **Migrate the transactions, recompute the balances.** Then compare recomputed balances with the old system's stored balances per account. Every difference must be explained before cutover.
- **Reconcile against the outside world too:** totals per day against payment-provider, bank or settlement reports, not only old database vs new database.
- **In-flight operations at cutover:** payments initiated but not confirmed, refunds pending, payouts queued. Provider callbacks will arrive at whatever URL was registered when the payment started. Either keep the old callback endpoint alive and forwarding, or drain in-flight operations before the switch. List each provider's callback URL and how it is changed (some require a support request and days of lead time).
- **Idempotency keys** from the old system must carry over, so a retried request after cutover doesn't double-charge.
- **Never migrate secrets in plaintext.** Card data shouldn't be there at all (tokens only). Provider and API credentials get rotated on cutover, not copied.

## 5. Reconciliation

Define the queries **before** the first run. Each has a pass criterion.

- Counts per entity (and per status) old vs new.
- Hash of key fields per row, compared in bulk (for example a hash of `id, amount, status, updated_at`), then aggregated per batch so mismatches can be narrowed down.
- Sums: per account, per day, total, to the minor unit, exact match.
- Sample spot checks: N random users end to end (profile, history, balances, owned records) viewed in both systems by a person.
- Referential checks in new: no orphans, uniques hold.

Keep reconciliation running daily throughout coexistence, not only once.

## 6. Rehearsal

- Run the full migration against a recent production snapshot (handled under the same data-protection rules as production).
- Record: wall-clock duration per step, row counts, quarantined rows, reconciliation results.
- The measured duration, plus margin, is the freeze window. Re-measure if data grows significantly before cutover.
- Rehearse the rollback path the same way.
