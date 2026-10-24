# greenfield-migration

An agent skill for planning and running a greenfield rewrite: rebuilding an existing app or service from scratch on a new stack and moving real users, data and traffic onto it without losing behaviour, money or trust.

It is written for [Claude Code](https://docs.anthropic.com/en/docs/claude-code) and any other agent that reads the `SKILL.md` convention, but the content is plain Markdown. A person can follow it without an agent at all.

The skill is independent of language, framework, platform and industry. It has been used for backend-plus-frontend rewrites and mobile app ports; the same phases apply to a data pipeline or a whole product.

## What it produces

A set of files under `docs/migration/<name>/` in your repository, one per phase, that a team can execute and check off:

| Phase | File | What it answers |
|---|---|---|
| 0 | `00-brief.md` | Why rewrite, what can't break, strategy, baseline, success criteria, top risks |
| 1 | `01-inventory.md` | Everything the old system actually does, with a keep / change / improve / add / drop disposition per behaviour, plus the gaps found along the way |
| 2 | `02-parity.md` | How each kept behaviour is verified, and which rows block cutover |
| 3 | `03-data.md` | Entity mapping, sync strategy, reconciliation queries, rehearsal results |
| 4 | `04-identity.md` | Credentials, sessions, old clients, third-party identities |
| 5 | `05-parallel-run.md` | Shadow traffic, dual-run, cohort rollout with gate metrics |
| 6 | `06-cutover-runbook.md` | Timed, owned steps, go/no-go, one-way doors, rollback |
| 7 | `07-decommission.md` | Stabilisation, archive, shutdown, cost check, handover |

Plus a `README.md` that tracks the current phase and the top open risks.

## Install

**Claude Code** (personal, available in every project):

```sh
git clone https://github.com/<you>/greenfield-migration-skill ~/.claude/skills/greenfield-migration
```

**Claude Code** (shared with a team, checked into one repository):

```sh
git clone https://github.com/<you>/greenfield-migration-skill .claude/skills/greenfield-migration
```

**Claude.ai:** zip the repository and upload it as a skill in your settings, or install the `.skill` file from the latest release.

**Other agents:** point the agent at `SKILL.md`. The references and templates are linked from it with relative paths.

## Use

Describe the rewrite in your own words. The skill triggers on phrases like "rewrite", "v2", "replatform", "port to", "move off the legacy system", "how do we migrate users". Then:

- Point it at the old code if you have it. The old code is treated as the spec, but not as the ceiling: the skill looks for bugs, security and performance gaps and missing behaviour while it reads, and asks you which improvements ship with the migration.
- Answer its questions on money handling, data ownership and the order of work. It asks one at a time and attaches a recommendation.
- Ask for a single phase if that is all you need ("just inventory the old app").
- Expect it to hold the new code to a standard: doc blocks instead of inline comments, no stubs or skipped tests, designs that still work at ten times the load, and documentation read before behaviour is assumed. It writes those rules into your repository's contributor guide so they outlive the migration.

## Layout

```
SKILL.md                              the workflow, phase by phase
references/
  strategy-and-assessment.md          strategy options, assessment checklist, success criteria, risk register
  data-migration.md                   sync strategies, script rules, transform pitfalls, money, reconciliation
  mobile-apps.md                      store identifiers, on-device storage, old versions, store rollout, OTA
assets/
  inventory-template.md               the behaviour inventory table
  runbook-template.md                 the cutover runbook
```

## Contributing

Issues and pull requests are welcome. Keep additions general: a rule that only holds for one stack or one industry belongs in a reference file with a clear "read this when" line at the top, not in `SKILL.md`.

## Licence

MIT. See [LICENSE](LICENSE).
