---
name: execute docs
description: >-
  Third step of /execute. Reads the plan and changes.md, finds the EXISTING
  docs that the diff made stale (README, docs/, Swagger, CLAUDE.md, inline
  JSDoc), and makes the smallest edit that makes them true again. Never
  creates a doc file. Appends to the same changes log. Use when the user says
  "/execute docs <plan path>" or right after /execute test finishes.
disable-model-invocation: true
---

<objective>
Every doc that described the old behaviour now describes the new one. Nothing
else changes. A doc that was not wrong is not touched.
</objective>

<inputs>
- A plan path with its `changes.md` from `/execute` and `/execute test`.
  Missing `changes.md` → stop and point at `/execute <plan path>`.
- The changed surface is the file list in `changes.md` plus `git diff --stat`.
</inputs>

<scope>
Edits allowed: existing `*.md`, `docs/**`, Swagger/OpenAPI files, JSDoc on the
exact symbols the diff changed. Nothing else.
Out of scope, always: production code, tests, `.env*`, git, any new file.
</scope>

<loop>
1. LIST the changed public surface from `changes.md`: routes, env vars,
   commands, exported symbols, config keys, behaviour named in R1..Rn.
2. GREP each item across the allowed doc set. Collect the hits that are now
   wrong or incomplete.
3. SHOW the hit table (doc `path:line` → what is stale → one-line fix) before
   editing.
4. EDIT each hit minimally: fix the sentence, row, or example. No rewrites,
   no new sections, no tone changes, no reformatting of untouched lines.
5. LOG: append `## Docs` to `changes.md` using the template.
No hits → log "Docs: nothing stale" and stop. That is a valid result.
</loop>

<changes-log-template>
```markdown
## Docs
| Doc | Was | Now | Serves |
|---|---|---|---|
| `README.md:42` | <stale line, short> | <fixed line, short> | R1 |

Files: N · Lines: +A/-B
Next (human): review `changes.md`, then `/execute review <plan path>`
```
</changes-log-template>

<hard-rules>
- Never create a file. A missing doc is reported under `Seen, not touched`,
  never written.
- Never edit a line the diff did not make stale.
- No code, test, or `.env*` edits. No git.
- A doc edit without a log row is not done.
</hard-rules>
