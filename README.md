# magic

Site: https://anupam-io.github.io/magic/

Spec to finished code, headless. `magic` takes an execution-ready plan, opens a
git worktree, and runs a coding agent through build, test, docs, two confirm
passes, review, and a draft PR. The only human gate is the plan check at the
start. After that the agent decides inside the plan's scope and logs every
assumption.

```
check → build → test → docs → confirm ×2 → commit 1 → review → commit 2 → beep → report
```

## Install

```bash
npm install -g @anupam-io/magic
```

Requires `git`, `jq`, `python3`, `gh` (for `--pr`), and one engine: the
`claude` CLI (default) or `codex`. macOS and Linux.

The plan is written interactively with the `/spec` skill inside Claude Code.
Link it into your skills once:

```bash
ln -s "$(npm root -g)/@anupam-io/magic/skills/spec" ~/.claude/skills/spec
```

`execute` needs no install: `magic` inlines it into each agent prompt.

## Workflow

1. In your repo, inside Claude Code: `/spec <ticket id | issue URL | pasted request>`.
   It writes `.claude/plans/<YYYY-MM-DD>-<slug>/plan.md`: EARS requirements, one
   Done-when command, out-of-scope, and after reading the repo an exact Blocks
   table (file, change, precedent, proof command, requirement served). It gates
   you at requirements and at blocks, and ends at `READY_TO_EXECUTE`.
2. `magic .claude/plans/<dir>` runs the check gate. A plan that is not
   `READY_TO_EXECUTE`, has open questions, lacks a Blocks table, or lists
   integration/e2e specs as test blocks exits here. Open questions land in
   `<plan dir>/questions.md`.
3. The run starts at once in `.claude/worktrees/<slug>` on the plan's branch,
   with ignored local files copied in and dependencies installed. Build, test,
   docs, and the confirms share one agent session. Review is a fresh one.
4. Commits land locally. With `--pr`, a green review pushes the branch and opens
   a draft PR whose title carries the review status circle. The body follows
   the repo's own PR template when it has one.

Reruns are safe: steps already recorded in `changes.md` are skipped. `--force`
reruns them.

## Usage

```
magic <plan dir | plan.md> [--claude|--codex] [--pr] [--dev] [--force] [--branch <name>] [--no-beep]
```

| Flag | Effect |
|---|---|
| `--claude` / `--codex` | Engine. `magic-claude` and `magic-codex` are shorthands. |
| `--pr` | After a green review: push and open a draft PR, plus a two-line "Contributed with Magic" comment. |
| `--dev` | Everything runs, including local commits, but nothing leaves the machine: no push, no `gh`. |
| `--force` | Rerun steps already recorded in `changes.md`. |
| `--branch <name>` | Branch when the plan has no `**Branch:**` line. Default: `<type>/<slug>` with the type inferred from the plan title. |
| `--no-beep` | Silence the finish sound. |
| `--version`, `--help` | |

The plan directory must live inside the target repo at `.claude/plans/<name>/`.
Base branch is `develop` when the remote has one, else `main`. `magic` refuses
to run on `main`, `master`, or `develop`.

## Environment

| Variable | Default | Meaning |
|---|---|---|
| `MAGIC_ENGINE` | `claude` | `claude` or `codex` |
| `MAGIC_MODEL` | `claude-fable-5-1` | Claude model. Falls back to `claude-opus-5-5` when unavailable. |
| `MAGIC_CODEX_MODEL` | codex default | Codex model |
| `MAGIC_MAX_ROUNDS` | `15` | Confirm passes before the run gives up |
| `MAGIC_CTX_LIMIT` | `180000` | Context tokens after which a Claude session is rotated |
| `MAGIC_HOME` | `~/.magic` | Where per-repo, per-branch `rows.tsv` timings accumulate across runs |
| `MAGIC_DEV` | `0` | `1` is the same as `--dev` |

## What a run leaves behind

Inside the worktree's plan directory: `changes.md` (every step, assumptions,
verify entries, the review), `magic-run.md` (the timing and cost table), and
`rows.tsv`. The PR body draft is at `debug/pr.md`.

## Agent tools

The agent runs with Read, Edit, Write, Glob, Grep, and Bash limited to `yarn`,
`npm`, `npx`, `git diff`, `git status`, and `git log`. It cannot push, call
`gh`, or reach the network. `magic` itself does the git operations.

## License

MIT
