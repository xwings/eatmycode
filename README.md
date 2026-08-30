# eatmycode

A portable skill that teaches any coding agent — Claude Code, Codex
CLI, OpenCode, OpenClaw, Grok Build, or anything else that loads a
`SKILL.md` — to build and maintain a durable architecture doc set for a
project: an `ARCHITECTURE.md` control center at the root, plus one
`ARCHITECTURE/<module>.md` per subsystem, every module file following a
fixed seven-section template, plus three project-level discipline blocks
covering how to think, how to write, and how to verify.

The point is to give future sessions (yours or another agent's) a
single place to check whether the work in progress is still aligned
with what the project is supposed to be. No more re-deriving the
architecture every conversation.

## Why

The skill covers bootstrap, add-module, and audit workflows. In each
case the agent inspects existing code/docs first, including any
`AGENT.md`, `AGENTS.md`, or `CLAUDE.md`, plans from the available
evidence, writes the doc changes, reviews them, and verifies the result.
It asks only during planning, and only when a required answer cannot be
discovered or safely inferred. Everything else runs autonomously until
the goal passes the release gate.

## How it works

1. **Detect path** — create (no `ARCHITECTURE.md` or `ARCHITECTURE/`
   yet) or update (one exists; add a module or audit).
2. **Frame** — understand the request, code, and goal; attach observable
   checks and produce a focused plan.
3. **Resolve** — infer unknowns from repository evidence. Ask one focused
   planning question only when a required decision is genuinely blocked.
4. **Write** — produce the smallest change that reaches the goal.
5. **Prove + review** — run the grep-based check that every module file
   has all seven required sections, run each module's **How to Test**,
   then walk the seven Review Checks over the diff.
6. **Gate** — apply the Definition of Done. Unticked box or surviving
   `major`: send the evidence back to Write or Frame, then repeat. All
   ticked: the result is ready for public or production release.

Throughout, the skill works as a **team, not a solo runner** — a Planner,
Coder, Tester, and Verifier each own a phase (plan → code → test →
verify), via subagents when the harness has them and otherwise as
distinct labeled passes. Those four roles are one turn of the Development
Loop the skill emits: the skill obeys the same discipline it writes down.
The roles hand work directly to each other without approval, progress, or
reporting sessions.

The workflow, templates, coding discipline, review method, and
verification gate all live in [`SKILL.md`](SKILL.md).

## Installation

Install `SKILL.md`. The directory name must be `eatmycode` (matching the
frontmatter `name`).

| Harness     | Project-level (committed to repo)       | User-level (machine-wide)                       |
| ----------- | --------------------------------------- | ----------------------------------------------- |
| Claude Code | `.claude/skills/eatmycode/SKILL.md`     | `~/.claude/skills/eatmycode/SKILL.md`           |
| Codex CLI   | `.agents/skills/eatmycode/SKILL.md`     | `~/.codex/skills/eatmycode/SKILL.md`           |
| OpenCode    | `.opencode/skills/eatmycode/SKILL.md`   | `~/.config/opencode/skills/eatmycode/SKILL.md`  |
| OpenClaw    | `.openclaw/skills/eatmycode/SKILL.md`   | `~/.openclaw/skills/eatmycode/SKILL.md`         |
| Grok Build  | `.grok/skills/eatmycode/SKILL.md`       | `~/.grok/skills/eatmycode/SKILL.md`             |

Pick the row that matches your harness and run the corresponding command
from this repository root:

```sh
# Claude Code (user-level)
mkdir -p ~/.claude/skills/eatmycode && cp SKILL.md ~/.claude/skills/eatmycode/

# Codex CLI (project-level)
mkdir -p .agents/skills/eatmycode && cp SKILL.md .agents/skills/eatmycode/

# OpenCode (user-level, native path)
mkdir -p ~/.config/opencode/skills/eatmycode && cp SKILL.md ~/.config/opencode/skills/eatmycode/

# OpenClaw (user-level, native path)
mkdir -p ~/.openclaw/skills/eatmycode && cp SKILL.md ~/.openclaw/skills/eatmycode/

# Grok Build (user-level, native path)
mkdir -p ~/.grok/skills/eatmycode && cp SKILL.md ~/.grok/skills/eatmycode/
```

### Cross-harness shortcut

OpenCode discovers `.claude/skills/` and `.agents/skills/` (both
project and home variants); OpenClaw also discovers `.agents/skills/`
under the home directory; Grok Build is Claude-Code-compatible and
also reads `.claude/skills/` and `~/.claude/skills/`. That gives you
these multi-harness options:

- Install at `~/.claude/skills/eatmycode/` → **Claude Code + OpenCode
  + Grok Build** with one copy.
- Install at `.agents/skills/eatmycode/` → **Codex CLI + OpenCode**
  with one copy.
- Install at `~/.agents/skills/eatmycode/` → **Codex CLI + OpenClaw**
  (plus OpenCode if it's set to walk the home dir) with one copy.

Two physical copies (or symlinks pointing to one) cover all five
harnesses.

## Invocation

| Harness     | Explicit                | Implicit                                                     |
| ----------- | ----------------------- | ------------------------------------------------------------ |
| Claude Code | `/eatmycode`          | Triggered by frontmatter `description` when relevant         |
| Codex CLI   | `$eatmycode` or `/skills` | Triggered if `allow_implicit_invocation` is true (default) |
| OpenCode    | Reference by name       | Discovered via the native skill tool                         |
| OpenClaw    | Reference by name       | Triggered by frontmatter `description` when relevant         |
| Grok Build  | `/eatmycode` or `/skills` | Triggered by frontmatter `description` when relevant       |

If implicit invocation is disabled or you want a guaranteed entry, just
ask the agent: *"use the eatmycode skill to design the architecture
for this project"*.

## What it produces

```
your-project/
├── ARCHITECTURE.md           # control center: mission, layout, roadmap, index
├── ARCHITECTURE/
│   ├── module-a.md           # one file per subsystem,
│   ├── module-b.md           # each following the seven-section template
│   └── module-c.md           # defined by the SKILL.md Module Template
├── AGENT.md   -> ARCHITECTURE.md   # optional symlinks, kept in sync
├── AGENTS.md  -> ARCHITECTURE.md
└── CLAUDE.md  -> ARCHITECTURE.md
```

Each module file has exactly these sections, in order: **Goal**,
**Status**, **Code Structure**, **Key Types and Entry Points**,
**Interactions**, **How to Test**, **Open Gaps / Roadmap**. The
Verification section in `SKILL.md` checks their presence and order.

If `AGENT.md`, `AGENTS.md`, or `CLAUDE.md` already exist, the skill
uses their durable project guidance to seed `ARCHITECTURE.md` before
replacing them with symlinks.

The generated `ARCHITECTURE.md` also carries three discipline blocks that
future agents read before touching code, so work stays consistent across
sessions and harnesses:

- **Development Loop** — the spine. Five stages that repeat: **Frame**
  (think, state assumptions, attach a check to the goal) → **Write**
  (smallest change that reaches it) → **Prove** (run the tests, keep the
  output) → **Review** (all seven checks against your own diff) →
  **Gate** (Definition of Done: release, or feed the findings directly
  back into the loop).
- **Coding Discipline** — the *Write* stage in detail. Adapted from the
  Karpathy `CLAUDE.md`: think before coding, simplicity first, surgical
  changes, goal-driven execution, and autonomous completion.
- **Review Checks** — the *Review* stage in detail. Seven separate
  passes — style, naming, duplication, quality, fit, dependencies,
  security — with a severity rubric (`blocker` / `major` / `nit` /
  `info`) and a merge threshold.

```
      ┌──────────────────────────────────────────────────────┐
      │                    findings remain                   │
      ▼                                                      │
  1. FRAME  →  2. WRITE  →  3. PROVE  →  4. REVIEW  →  5. GATE
     think       build        evidence     7 checks     done?
                                                          │
                                                          ▼
                                                        ship
```

Writing the code is stage 2 of 5. Nothing is done until Review has run
and Gate has passed.

**"Good" is defined, not felt.** The Definition of Done sets the bar at
open-source standard: *a maintainer who has never seen this change, and
cannot ask you anything, could review and merge it from the diff and the
docs alone*. That expands into checkable boxes across correctness (tests
exist, they pass, it builds from a fresh clone), review (all seven checks
walked, no unresolved `major`), legibility (a stranger can run it from
the README; every changed line traces to the goal; no debug leftovers or
secrets), and contract (the architecture docs updated in the same change,
breaking changes called out, licenses respected). "It runs" and "I feel
good about it" are explicitly not on the list — the block that defines
the loop is the same one that rejects weak success criteria.

Three rules keep the loop honest and finite: **evidence or no finding**
(every finding cites `file:line`), **the repository is the authority** (a
convention nobody can locate in the tree is not a convention), and
**every pass closes a named finding**. Two no-change passes force a Gate
re-evaluation; if Done still fails, the work returns to Frame. Three
passes against the same finding also return to Frame automatically,
because the design is wrong, not the code.

## Sources

The Coding Discipline block is adapted from:

- [Karpathy `CLAUDE.md`](https://raw.githubusercontent.com/multica-ai/andrej-karpathy-skills/refs/heads/main/CLAUDE.md)

The Review Checks block is adapted from a seven-specialist review panel
in which one agent owns each check.

Harness docs used to verify the install paths and invocation
conventions above:

- [Claude Code — Skills](https://code.claude.com/docs/en/skills)
- [Codex — Agent Skills](https://developers.openai.com/codex/skills)
- [Codex — AGENTS.md](https://developers.openai.com/codex/guides/agents-md)
- [OpenCode — Agent Skills](https://opencode.ai/docs/skills/)
- [OpenCode — Rules / AGENTS.md](https://opencode.ai/docs/rules/)
- [OpenClaw — Skills](https://docs.openclaw.ai/tools/skills)
- [OpenClaw — Agent harness SDK](https://docs.openclaw.ai/plugins/sdk-agent-harness)
- [Grok Build — Skills, Plugins & Marketplaces](https://docs.x.ai/build/features/skills-plugins-marketplaces)
