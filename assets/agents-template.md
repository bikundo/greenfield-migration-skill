# AGENTS.md

Ground rules for anyone, human or agent, working in this repository. They apply to every file in every language here. Read this before changing anything.

## What this repository is

<One paragraph: what the system does, who uses it, what it replaced and why. Link the migration plan: `docs/migration/<name>/README.md`. Name the old system and where it lives while it still exists.>

## Rules

### 1. No inline comments. Use doc blocks.
Never write a comment inside a function body or at the end of a line. Explain a module, type, function or exported value with a doc block above it, in the language's convention (<name the conventions for the languages in this repo>). If code needs an inline comment to be understood, restructure the code until it does not.

### 2. Simple, scalable, maintainable code.
Prefer the design that still holds at ten times the users, tenants and data, not the one that is quickest to type today. When two options differ in effort but only one scales, take the one that scales and say so in the PR. "Scalable" includes operability: observable, idempotent, bounded in memory, safe to run more than one copy of.

### 3. No shortcuts.
No stubs returning fake data, no commented-out code, no skipped or disabled tests, no "fix later" markers, no hard-coded secrets or environment values, no swallowed errors. Out-of-scope work goes in the tracker with an owner, not in the code. Half-finished work is reported as half-finished.

### 4. When in doubt, ask.
Do not resolve an ambiguity by guessing. If a decision would change the data model, an external contract, money handling, tenancy, security, or the order of work, stop and ask <decision owner> one question at a time with a recommendation attached. Routine implementation choices with a conventional answer do not need a question.

### 5. Research. Do not assume.
Before relying on how a protocol, library, API or service behaves, read its specification or documentation and record the source. A plan or PR that depends on an unverified assumption must name the assumption.

### 6. These rules are durable.
Update this file when <decision owner> adds or changes a rule. Do not relax a rule because it is inconvenient for the task at hand.

## Conventions that follow from the rules

<Fill in per stack. Examples of the kind of thing that belongs here:>
- **Code layout:** <project structure, package boundaries, where new code goes>
- **Data:** <migrations policy, money as integer minor units, timestamps as UTC, every tenant-owned table carries the tenant key>
- **Contracts:** <how the API is defined and how clients are generated; frozen external contracts and where they are documented>
- **Testing:** <what must have a test, how CI runs, where parity and reconciliation tests live>
- **Delivery:** <how builds and releases are produced; nothing deployed by hand>

## Where to look

- Migration plan and decision log: `docs/migration/<name>/`
- Behaviour inventory (what the old system did and what was kept, changed, improved, added or dropped): `docs/migration/<name>/01-inventory.md`
- Parity contract and test plan: `docs/migration/<name>/02-parity.md`
- Runbooks and operations: <path>
- How to run locally and run the tests: <path or commands>
