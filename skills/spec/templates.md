# Spec templates

Copy verbatim, then fill. Keep sections in this order.

## Plan doc

Write to `.claude/plans/<YYYY-MM-DD>-<ticket>/plan.md`. Probe output and other
evidence sit next to it in the same directory.

```markdown
# <Title> — <ticket-id>

**Status:** <STATE> — <one line: what it waits on>
**Repo:** <repo + base branch>  ·  **Source:** <linear/github/url/pasted>
**Branch:** <branchName or "none yet">
**Mode:** <prototype | production> — <one line why>

## Context sources
- <Granola: meeting title, date, link> — <fact it contributed>
- <Slack/GitHub/doc link> — <fact it contributed>
- <link> — could not open (<reason>)

## Requirements (nodded <YYYY-MM-DD>)

  R1  WHEN <trigger>, the system SHALL <response>.
  R2  IF <unwanted trigger>, THEN the system SHALL <response>.

Done when
  <one deciding command>
  <condition it must hold under>
  <one thing that must not change>

Out of scope
  <line>
  <line>

Stop when
  <termination condition>

Assumptions
  <defaults the human agreed to>

## Current state (verified)
- `path/to/file.ext:LINE` — <what exists and why it matters>
**Existing tooling:** <what/where — or "none found">
**Precedent:** `path:line` — <sibling code that solves the same shape>

## Blocks (nodded <YYYY-MM-DD>)

| # | Files | Change | Precedent | Proof | Serves |
|---|---|---|---|---|---|
| 1 | `src/a.ts:12-40` | <1-2 lines> | `src/x.ts:30` | `npx tsc --noEmit -p <path>` | R1 |
| 2 | `src/b.ts` (new) | <1-2 lines> | `src/y.ts:10` | `curl -s localhost:3000/...` | R1, R2 |

Estimated production lines: ~<N> (≤300)

## Test blocks (for /execute test)

| # | Spec file | Cases | Serves |
|---|---|---|---|
| T1 | `src/a.spec.ts` (extend) | <given/when/then, one line each> | R1 |

## Evidence
<links to probe outputs in this directory — or omit>

## Open questions
<[NEEDS CLARIFICATION: ...] — ask one at a time, strike as answered; "none" at READY_TO_EXECUTE>
```

## Research ticket (ANSWERED)

Same header and Requirements section (Done when = the questions to answer),
then replace Blocks and Test blocks with:

```markdown
## Findings
<one short paragraph per Done-when question, each claim cited `path:line` or to an evidence file>
```
