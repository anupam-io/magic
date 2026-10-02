# What people actually put in a spec

Condensed from EARS (Mavin, Rolls-Royce 2009), Kiro specs, GitHub spec-kit,
and mainstream acceptance-criteria guidance. Read when drafting; do not paste.

## EARS: the sentence grammar

One clause order, always: `While <precondition>, When <trigger>, the <system>
shall <response>`. Most clauses optional. Five patterns plus one combination:

| Pattern | Template | Use for |
|---|---|---|
| Ubiquitous | The system shall <response>. | always-true invariants |
| Event-driven | WHEN <trigger>, the system shall <response>. | request/response, actions |
| State-driven | WHILE <state>, the system shall <response>. | modes, in-progress conditions |
| Unwanted behaviour | IF <trigger>, THEN the system shall <response>. | errors, limits, abuse |
| Optional feature | WHERE <feature>, the system shall <response>. | flags, tiers, config |
| Complex | WHILE <state>, WHEN <trigger>, the system shall <response>. | when two of the above combine |

Ruleset: zero or many preconditions, zero or one trigger, one system name, one
or many responses. A requirement with two triggers is two requirements.

## Kiro: requirements.md shape

User story ("As a <role>, I want <goal>, so that <benefit>") followed by
numbered acceptance criteria, each in EARS form. Three files per spec:
`requirements.md` (what), `design.md` (how), `tasks.md` (steps, each traced
back to a requirement number). Our `/spec` produces all three in one doc:
Requirements, Current state, and Blocks.

## GitHub spec-kit: sections worth borrowing

- User stories ranked P1/P2/P3, each with an "independent test" line: how this
  story alone can be verified and what value it delivers.
- Acceptance scenarios in Given/When/Then.
- Edge cases enumerated separately.
- Functional requirements numbered FR-001.., each MUST be testable.
- Success criteria: measurable, technology-neutral (time, volume, rate).
- Assumptions: defaults chosen where the input was silent, written down.
- Ambiguity marker: `[NEEDS CLARIFICATION: <detail>]`. Never silently guess.
- Explicit exclusion: no implementation details, stack choices, or design.

## Acceptance-criteria guidance (LogRocket, AltexSoft, Qase, ICAgile)

- If you cannot answer "yes" or "no" to whether a criterion is met, it is too
  vague.
- For every happy path, write at least one error scenario.
- Checklist form for simple tickets; Given/When/Then for branching flows.
  Most teams use both.
- Acceptance criteria say what the feature must do. Definition of done says
  what the process must include (tests, docs, review). Keep them apart.
- Write them with dev and QA present, not alone.

## Which pattern, by ticket shape

| Ticket shape | Requirements | Done when |
|---|---|---|
| Feature / endpoint | WHEN + IF lines per path | filtered integration test or curl |
| Bug | IF (the bad trigger) THEN (correct behaviour); one WHEN for the regression | the failing case, now passing |
| Data / ops task | Ubiquitous lines describing the end state | a query or a file that shows the state |
| Refactor | none; behaviour unchanged is the requirement | full suite for the touched module, plus "no public signature changed" |
| Research | none | the questions the doc must answer |

## Sources

- https://alistairmavin.com/ears/
- https://en.wikipedia.org/wiki/Easy_Approach_to_Requirements_Syntax
- https://kiro.dev/docs/specs/
- https://github.com/github/spec-kit/blob/main/templates/spec-template.md
- https://martinfowler.com/articles/exploring-gen-ai/sdd-3-tools.html
- https://blog.logrocket.com/product-management/acceptance-criteria/
- https://www.altexsoft.com/blog/acceptance-criteria-purposes-formats-and-best-practices/
