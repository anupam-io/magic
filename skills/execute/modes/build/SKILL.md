---
name: execute
description: >-
  Execute a READY_TO_EXECUTE spec's APPLICATION CODE block by block, exactly as
  the spec's block list says, writing a changes log as each block lands. Runs
  in the spec's Mode: prototype (prove it fast) or production (hardened,
  secure, ship-ready).
  Production code only: no tests, no docs, no git. Tests come after via
  /execute-test. Use when the user says "/execute <plan path>", "execute the
  plan", "implement this plan now", or hands over a spec from /spec.
disable-model-invocation: true
---

<objective>
Turn a nodded spec into a production-code diff a reviewer can follow block
by block. The changes log is written WHILE working, so the human can stop at
any block and still know exactly what exists.
</objective>

<inputs>
- A plan path (`.claude/plans/<dir>/plan.md`) from `/spec` whose Status is
  READY_TO_EXECUTE and which has a `## Blocks` table. Anything else → stop and
  point at `/spec <plan path>`. Invoking `/execute` is the go.
- Mode comes from the plan's `**Mode:**` line. An argument
  (`/execute <plan> production`) overrides it; say so in the log header.
</inputs>

<modes>
PROTOTYPE — prove the idea works end to end, fast.
- Happy path only. Proof per block: type-check plus one run of the real flow.
- Allowed shortcuts: minimal validation, generic errors, no retries, hardcoded
  limits. Every shortcut is listed under `Shortcuts` in the block's log entry,
  never left as a code comment.
- Still never: secrets in code or logs, unscoped tenant queries, writes to
  shared systems, skipped type-check.
- The PR is a draft. Next (human) says so.

PRODUCTION — hardened, secure, ready to ship. Every block, before its proof:
- Trust boundaries: untrusted input validated by shape, length, and contents.
- Errors: precise HTTP status (4xx for client faults), no swallowed errors,
  no internals leaked in responses.
- I/O: every DB/ES/HTTP call has a timeout; select only needed columns;
  query builder over raw SQL.
- Security: every query scoped by tenant/workspace, ownership checked before
  mutation, no secrets or PII in logs, authz on every new route.
- Compatibility: no API contract or DB schema break unless the spec says so.
Record the checklist result per block under `Hardening`. An item that needs a
file outside the block → stop and ask.

Hardening a prototype: run `/execute <plan> production` on the same plan. The
prototype's `Shortcuts` entries become the work, inside the same block Files.
</modes>

<scope>
Only production code: `apps/**`, `src/**`, migrations, config the plan names.
Out of scope, always: test files (`*.spec.ts`, `*.test.ts`, `__tests__/`,
`test/`, fixtures), docs (`*.md`, `docs/`, Swagger-only or comment-only
edits), `.env*`. A plan step that is only tests or docs → skip it and note it
in the log; `/execute test` picks up the tests.
</scope>

<blocks>
The spec's `## Blocks` table IS the block list. Show it, then run it as given:
same order, same files, same proof commands. Never re-split, merge, or reorder.
A block that doesn't match the code (a cited `path:line` moved, a precedent is
gone, a proof command doesn't exist) is a spec problem: stop and hand back for
`/spec <plan path>` to fix it.
</blocks>

<loop>
For each block, in order:
1. SAY the block: number, title, files, proof command. One line each.
2. EDIT only the block's Files. Minimal diff. No drive-by fixes; note them
   under "Seen, not touched".
3. PROVE: run the proof. Fails → fix inside the block's files, rerun once.
   Fails again → stop, log it, hand back.
4. LOG: append the block to `<plan-dir>/changes.md` (create on first block)
   using the template. Write it before starting the next block.
5. CHECK drift: a file outside the block's Files, a new dependency, a public
   signature the plan did not name → stop and ask.
After the last block: run the repo's build command once (`npm run build` or
the equivalent). A failure in a file the plan does not touch is pre-existing:
list it under "Final proof" and move on, never fix it. Append the entry. Print
the block table.
</loop>

<changes-log-template>
```markdown
# Changes — <plan title>
Plan: <path> · Started: <UTC timestamp>

## Block 1 · <title>
Files: `src/a.ts:12-40`, `src/b.ts:88`
Why: <one line tied to the plan step or requirement Rn>
Proof: `<command>` → <pass / fail + one line>
Deviations: none | <what and why>
Shortcuts (prototype): none | <each shortcut taken>
Hardening (production): <validation / errors / timeouts / tenancy / compat — ok or what was done>
Seen, not touched: <thing worth a follow-up, or "none">

## Final proof
`<build command>` → <pass | fail: <pre-existing failures in untouched files, listed, not fixed>>
Blocks: N · Files: M · Lines: +A/-B
Mode: <prototype | production>
Next (human): review `changes.md`, then `/execute test <plan path>`
```
</changes-log-template>

<hard-rules>
- No git: no branch, stash, commit, push.
- No test or doc file created or edited. If a proof needs a test, that is
  `/execute test`'s job.
- No new dependency unless the plan names it.
- Never fix a failure in a file the plan does not touch.
- A block without a log entry is not done.
- Over ~300 lines total → stop after the current block and propose the split
  point for a second PR.
- Never touch `.env*`; env var names from `.env.example` only.
</hard-rules>
