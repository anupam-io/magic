---
name: execute review
description: >-
  Fourth step of /execute. Reviews the whole diff against the project's own
  standards (CLAUDE.md, lint/tsconfig/prettier config, ARCHITECTURE.md, the
  precedents the plan cited) and fixes what it finds in place: naming,
  structure, duplication with existing helpers, dead code, convention drift.
  Stops when every R1..Rn and Done-when in the plan still holds and no finding
  is open. Appends to the same changes log. Use when the user says
  "/execute review <plan path>" or right after /execute docs finishes.
disable-model-invocation: true
---

<objective>
The diff reads as if the repo's most consistent contributor wrote it. Behaviour
is unchanged: every requirement that passed before this step passes after it.
</objective>

<inputs>
- A plan path with its `changes.md` from the earlier steps. Missing →
  stop and point at `/execute <plan path>`.
- Standards, in priority order: repo `CLAUDE.md` (+ `CLAUDE.local.md`),
  `ARCHITECTURE.md` / `FUNCTIONS.md` / `TYPES.md` if present, lint, tsconfig,
  prettier config, then the `Precedent` column of the plan's Blocks table.
  Only standards the repo states or demonstrates. Never personal taste.
</inputs>

<scope>
Edits allowed: every file `changes.md` lists, plus any test for those files.
Pulling the diff onto an existing helper may touch the call site only.
Out of scope, always: files the diff did not touch, docs, `.env*`, git, new
dependencies, public signatures the plan did not name.
</scope>

<loop>
1. READ the standards in order. Note each rule as one line with its source.
2. DIFF: `git diff` plus `git diff --stat`. Read every changed hunk.
3. FIND: for each hunk, check the rules and the repo's precedent. Also check:
   a helper that already does ~80% of a new function; a near-copy of
   existing code; scaffolding, unused imports, dead branches; comments that
   state the what; `||` vs `??` per the field's valid values; missing
   timeouts on I/O; `SELECT *`; unscoped tenant queries.
   Record each as a finding: `path:line`, rule + source, fix in one line.
4. SHOW the findings table before editing. None → step 7.
5. FIX each finding in place, smallest diff, one finding at a time.
6. PROVE: format and lint the changed files (the repo's prettier and eslint),
   then run the repo's build command once. No tests here; session 1 proved
   the behaviour and CI runs the suite. Red → revert the fix that broke it,
   log it as "declined: breaks build". Then the loop repeats from step 2
   until a pass adds no new finding. Max 3 passes; then stop and log what is
   open.
7. LOG: append `## Review` to `changes.md` using the template. Print it.
</loop>

<changes-log-template>
```markdown
## Review
Standards read: `CLAUDE.md`, `.eslintrc.cjs`, `tsconfig.json`, <...>
| # | Finding (`path:line`) | Rule (source) | Fix | Result |
|---|---|---|---|---|
| 1 | <what> | <rule> (`CLAUDE.md:12`) | <one line> | fixed / declined: <why> |

Passes: N · Fixes: M · Lines: +A/-B
Proof: format → pass · lint → pass · `<build command>` → pass
Open: none | <finding # and why>
Next (human): review `changes.md`, then `/code-review`, `/cnp`, `/pr create`
```
</changes-log-template>

<hard-rules>
- Behaviour never changes: rename, move, dedupe, delete dead code. Never alter
  a condition, a query, a response shape, or an error path.
- Never apply a rule the repo does not state or demonstrate.
- Never edit a file outside the `changes.md` list and its tests.
- Never widen scope: no new features, no fixing pre-existing code the diff
  did not touch. Those go under `Open` as follow-ups.
- No docs, no `.env*`, no git, no new dependency.
- A fix without a log row is not done.
</hard-rules>
