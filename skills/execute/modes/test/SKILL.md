---
name: execute test
description: >-
  Second half of /execute. Takes the plan and the changes.md that /execute
  left, writes the spec's test blocks, runs them, fixes the application code
  when a test proves it wrong, then self-checks the diff against the spec.
  Appends to the same changes log. No docs, no git. Use when the user says
  "/execute test <plan path>", "now test the plan", "run the tests for this
  plan and fix what breaks", or right after /execute finishes.
disable-model-invocation: true
---

<objective>
Prove the diff `/execute` produced. Every plan requirement ends with a test
that fails without the change and passes with it. When a test exposes a real
bug in the app code, fix the app code, minimally, and log it.
</objective>

<inputs>
- A plan path (`.claude/plans/<dir>/plan.md`) from `/spec` with a
  `## Test blocks` table, and its `changes.md` from `/execute`. Missing
  `changes.md` → ask whether to run `/execute` first; do not guess what was
  implemented. Missing Test blocks → stop and point at `/spec <plan path>`.
- The spec's Done-when command is the final proof.
- Mode from `changes.md` (what `/execute` ran). PROTOTYPE: the happy-path
  cases in the Test blocks only; IF/THEN cases are listed as deferred in the
  log. PRODUCTION: every case, plus one test per trust boundary the diff adds
  (bad input → 4xx, missing auth → 401/403, other tenant's resource → 404).
</inputs>

<scope>
Edits allowed: test files (`*.spec.ts`, `*.test.ts`, `__tests__/`, `test/`,
fixtures) and the production files listed in `changes.md`. A fix that needs a
file outside that list → stop and ask.
Out of scope, always: docs (`*.md`, `docs/`, Swagger-only or comment-only
edits), `.env*`, git.
</scope>

<blocks>
The spec's `## Test blocks` table IS the list. Show it, then run it as given:
same spec files, same cases. Test blocks are unit specs. Proof per block:
`npx jest <spec path> --runInBand` (or the repo's runner) on that one file.
An integration or e2e spec in the table (Docker, testcontainers, a live DB,
`*.integration.spec.ts`, `*.e2e-spec.ts`) → stop and hand back for `/spec`;
such a spec belongs in Done-when and runs once, at the end. No tests beyond
the table, no coverage padding. A case that can't be written as specified →
stop and hand back for `/spec`.
</blocks>

<loop>
For each test block, in order:
1. SAY the block: number, what it proves, spec file, proof command.
2. WRITE the tests. Given-When-Then per case, one behavior per `it`.
3. RUN the proof.
   - Green → step 4.
   - Red → decide against the plan which side is wrong.
     Code wrong → fix inside the `changes.md` files, minimal diff, rerun.
     Test wrong → fix the test, rerun.
     Still red after one fix on each side → stop, log it, hand back.
4. LOG: append `## Test N · <title>` to `changes.md` using the template.
   Write it before the next block.
5. CHECK drift: any production edit outside `changes.md` files, any changed
   public signature, any new dependency → stop and ask.
After the last block: run the Done-when command once. Append "Final proof".
Then SELF-CHECK.
</loop>

<self-check>
Before any human sees the diff, try to refute it. For each Done-when line and
each Out-of-scope line in the spec:
- State the claim. Try to REFUTE it from the diff (`git diff` is a read, fine).
  Default to refuted unless a specific line proves it. Cite `path:line`.
- Also check: production diff under ~300 lines; no file outside the spec's
  block Files; no new runtime dependency unless the spec allowed it.
A refuted claim → fix it if inside the listed files, else name it. Append the
table to `changes.md`, then print it.
</self-check>

<changes-log-template>
```markdown
## Test 1 · <title>
Proves: <plan step or requirement Rn>
Spec: `apps/.../x.spec.ts:10-60`
Proof: `npx jest apps/.../x.spec.ts --runInBand` → <pass / fail + one line>
App fix: none | `src/a.ts:34` <what the test caught, one line>
Seen, not touched: <follow-up, or "none">

## Final proof
`<done-when command>` → <result>
Tests: N · App fixes: M · Lines: +A/-B

## Self-check
| Claim (from spec) | Verdict | Proof / refutation (`path:line`) |
|---|---|---|
| <done-when line> | holds / refuted | `src/x.ts:42` <one line> |
| <out-of-scope line> untouched | holds / refuted | <diff stat or path> |
| production diff under ~300 lines | holds / refuted | <N lines> |
| no files outside spec blocks | holds / refuted | <list or "none"> |
| production: every block's Hardening ok | holds / refuted / n/a (prototype) | <block # and gap> |

Next (human): review `changes.md`, then `/code-review`, `/cnp`, `/pr create`
```
</changes-log-template>

<hard-rules>
- Never make a test pass by deleting it, `skip`ping it, widening an
  assertion, or mocking the unit under test.
- Never edit a production file `changes.md` does not list.
- No doc file created or edited.
- No git: no branch, stash, commit, push.
- No new dependency unless the plan names it.
- A block without a log entry is not done.
- Never touch `.env*`; env var names from `.env.example` only.
</hard-rules>
