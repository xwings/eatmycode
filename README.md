# eatmycode

A portable skill that teaches any coding agent — Claude Code, Codex
CLI, OpenCode, OpenClaw, Grok Build, or anything else that loads a
`SKILL.md` — to build and maintain a durable architecture doc set for a
project: an `ARCHITECTURE.md` control center at the root, plus one
`ARCHITECTURE/<module>.md` per subsystem, every module file following a
fixed seven-section template and one project-level Coding Discipline
block.

The point is to give future sessions (yours or another agent's) a
single place to check whether the work in progress is still aligned
with what the project is supposed to be. No more re-deriving the
architecture every conversation.

## Why

The skill covers bootstrap, add-module, and audit workflows. In each
case the agent inspects existing code/docs first, including any
`AGENT.md`, `AGENTS.md`, or `CLAUDE.md`, asks only for missing facts,
gets explicit approval, writes the doc changes, then verifies the
result. That approval gate matters because the doc structure becomes the
project's agent-facing source of truth.

## How it works

1. **Detect path** — bootstrap (no `ARCHITECTURE.md` yet) or add-module
   (exists; user wants to add or audit one).
2. **Plan** — enter the harness's planning mode if it has one; state
   the planning phase explicitly otherwise.
3. **Ask** — collect required facts that code/docs cannot answer.
4. **Confirm** — present the plan, wait for an unambiguous *yes*.
5. **Write + verify** — produce the files, then run a grep-based check
   that every module file has all seven required sections.

The full workflow lives in [`SKILL.md`](SKILL.md). Read it once.

## Installation

Install `SKILL.md` plus the optional `mapping/` directory so the
harness-specific verb table stays available. The directory name must be
`eatmycode` (matches the `name:` in the frontmatter).

| Harness     | Project-level (committed to repo)       | User-level (machine-wide)                       |
| ----------- | --------------------------------------- | ----------------------------------------------- |
| Claude Code | `.claude/skills/eatmycode/SKILL.md`     | `~/.claude/skills/eatmycode/SKILL.md`           |
| Codex CLI   | `.agents/skills/eatmycode/SKILL.md`     | `~/.codex/skills/eatmycode/SKILL.md`           |
| OpenCode    | `.opencode/skills/eatmycode/SKILL.md`   | `~/.config/opencode/skills/eatmycode/SKILL.md`  |
| OpenClaw    | `.openclaw/skills/eatmycode/SKILL.md`   | `~/.openclaw/skills/eatmycode/SKILL.md`         |
| Grok Build  | `.grok/skills/eatmycode/SKILL.md`       | `~/.grok/skills/eatmycode/SKILL.md`             |

Pick the row that matches your harness and run the corresponding
command from the directory containing `SKILL.md` and `mapping/`:

```sh
# Claude Code (user-level)
mkdir -p ~/.claude/skills/eatmycode && cp SKILL.md ~/.claude/skills/eatmycode/ && cp -R mapping ~/.claude/skills/eatmycode/

# Codex CLI (project-level)
mkdir -p .agents/skills/eatmycode && cp SKILL.md .agents/skills/eatmycode/ && cp -R mapping .agents/skills/eatmycode/

# OpenCode (user-level, native path)
mkdir -p ~/.config/opencode/skills/eatmycode && cp SKILL.md ~/.config/opencode/skills/eatmycode/ && cp -R mapping ~/.config/opencode/skills/eatmycode/

# OpenClaw (user-level, native path)
mkdir -p ~/.openclaw/skills/eatmycode && cp SKILL.md ~/.openclaw/skills/eatmycode/ && cp -R mapping ~/.openclaw/skills/eatmycode/

# Grok Build (user-level, native path)
mkdir -p ~/.grok/skills/eatmycode && cp SKILL.md ~/.grok/skills/eatmycode/ && cp -R mapping ~/.grok/skills/eatmycode/
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

## Harness mappings

`SKILL.md` is written in generic verbs — *enter-plan-mode*, *ask-user*,
*run-subagent*, *exit-plan-mode*. The concrete primitive each harness
uses for each verb lives in [`mapping/`](mapping/):

- [`mapping/claude-code.md`](mapping/claude-code.md)
- [`mapping/codex.md`](mapping/codex.md)
- [`mapping/opencode.md`](mapping/opencode.md)
- [`mapping/openclaw.md`](mapping/openclaw.md)
- [`mapping/grok.md`](mapping/grok.md)

Every mapping file follows the same standard format: install paths,
invocation syntax, and a `verb → primitive` table. To add support for a
new harness, copy any existing file and fill in the same sections.

## What it produces

```
your-project/
├── ARCHITECTURE.md           # control center: mission, layout, roadmap, index
├── ARCHITECTURE/
│   ├── module-a.md           # one file per subsystem,
│   ├── module-b.md           # each following the seven-section template
│   └── module-c.md           # defined in SKILL.md
├── AGENT.md   -> ARCHITECTURE.md   # optional symlinks, kept in sync
├── AGENTS.md  -> ARCHITECTURE.md
└── CLAUDE.md  -> ARCHITECTURE.md
```

Each module file has exactly these sections, in order: **Goal**,
**Status**, **Code Structure**, **Key Types and Entry Points**,
**Interactions**, **How to Test**, **Open Gaps / Roadmap**. The
verification block in `SKILL.md` greps for these headers.

If `AGENT.md`, `AGENTS.md`, or `CLAUDE.md` already exist, the skill
uses their durable project guidance to seed `ARCHITECTURE.md` before
replacing them with symlinks.

The generated `ARCHITECTURE.md` also carries a **Coding Discipline**
section — five coding and architecture principles: clarify before
coding, simple deep design, cohesive boundaries, surgical changes, and
goal-based verification. Future agents read this block before writing
or reviewing code, so work stays consistent across sessions and
harnesses.

## Sources

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
