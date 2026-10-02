---
name: execute
description: >-
  Run a spec's plan. `/execute <plan path>` builds the blocks one at a time,
  proving each. `/execute test <plan path>` is the second half: writes the
  spec's test blocks, runs them, and fixes the application code.
  `/execute docs <plan path>` minimally updates the existing docs the diff
  made stale. `/execute review <plan path>` fixes the diff against the
  project's own standards. Typed by the user only.
argument-hint: "[test|docs|review] <plan path>"
disable-model-invocation: true
---

Read the first word of the arguments. `test`, `docs`, or `review` reads
`modes/<word>/SKILL.md`; anything else reads `modes/build/SKILL.md` and passes
the arguments to it. Order: build → test → docs → review.
Follow the mode file exactly; paths inside resolve against its own folder.
