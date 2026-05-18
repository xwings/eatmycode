---
name: eatmycode
description: Use when the user wants to design or maintain architecture docs for a project — either bootstrapping a new `ARCHITECTURE.md` + `ARCHITECTURE/` directory from scratch, or adding a new module file under `ARCHITECTURE/` for an existing project. Always plans and confirms with the user before writing.
---

# eatmycode

Establishes and maintains a control-center architecture doc for any
project. The convention has two parts:

- **`ARCHITECTURE.md` at the project root** — the control center.
  Carries only cross-cutting material: mission, hardware/runtime
  target, workspace layout, boot/entry flow, roadmap, address map (if
  relevant), an **Index** of `ARCHITECTURE/`, and a reference back to
  the per-module template in this skill.

- **`ARCHITECTURE/<module>.md`** — one file per subsystem. Every file
  conforms to the **Template** below: Goal, Status, Code Structure, Key
  Types and Entry Points, Interactions, How to Test, Open Gaps /
  Roadmap. No exceptions, no skipped sections.

If `AGENT.md` and `CLAUDE.md` exist at the project root, they should be
**symlinks to `ARCHITECTURE.md`** so all three views stay in sync.

This skill is harness-agnostic. It runs on Claude Code, Codex CLI,
OpenCode, and any other harness that loads a `SKILL.md`.

## When to invoke

- A project has no architecture doc and the user wants to design one.
- A new subsystem has landed in the codebase and needs an entry under
  `ARCHITECTURE/`.
- An existing module file is stale and needs an audit against the
  template.
- The user explicitly invokes the skill (syntax varies — see
  [`mapping/<your-harness>.md`](mapping/)).

The workflow below uses generic verbs: *enter-plan-mode*, *ask-user*,
*run-subagent*, *exit-plan-mode*, *write-file*, *run-shell*. The
concrete primitive each verb maps to on your harness lives in
[`mapping/<your-harness>.md`](mapping/). Consult that file once at the
start of a session.

## Workflow

### 1. Detect the path

Before asking anything, check the project root:

- **No `ARCHITECTURE.md` present** → **bootstrap path** (§3).
- **`ARCHITECTURE.md` present** → **add-module path** (§4) (or audit,
  if the user said they're reviewing an existing file).

If `ARCHITECTURE.md` exists, also list the current `ARCHITECTURE/`
directory so you know which modules are covered and which one the user
is about to add.

### 2. Enter planning mode

Invoke *enter-plan-mode*. If the harness has no plan-mode primitive,
state explicitly to the user: *"I am entering a planning phase. I will
ask questions and propose an approach, and will not write any files
until you approve."* All question-gathering and exploration happens in
this phase before any writes — non-negotiable. The doc structure is
load-bearing and writing it blind wastes the user's time.

If the scope is uncertain, inventory the codebase before asking
questions: invoke *run-subagent* if available, otherwise walk the
filesystem yourself with `ls` / `find` / `grep`.

### 3. Bootstrap path — questions

Invoke *ask-user* for each of the following. If the harness has no
structured-question primitive, ask inline, batching where it makes
sense, and wait for answers before proceeding. Capture answers verbatim
in your plan; do not paraphrase.

1. **Mission** — what is this project for? What's the headline goal?
   What problem does it solve that off-the-shelf alternatives don't?
2. **Target environment** — hardware (CPU/arch, RAM, accelerators), OS,
   runtime, deployment shape (single machine, cluster, browser,
   embedded). Capability constraints that drive design decisions?
3. **Workspace layout** — monorepo, multi-crate, single binary,
   monorepo of services? Top-level directories?
4. **Entry / boot flow** — what happens from process start to "ready"?
   Single binary? Service mesh?
5. **Roadmap** — what milestones (M1..MN) are planned? Headline goal?
   Status of each?
6. **Address map / port map / well-known constants** — only if relevant
   to the project (kernels, embedded firmware, network services).

Then **inventory the codebase**. Identify every top-level subsystem —
each one becomes a candidate `ARCHITECTURE/<name>.md`.

Then write:

- `ARCHITECTURE.md` (control center). It must:
  - Include the **Coding Discipline** block (see below) verbatim, as a
    top-level `## Coding Discipline` section placed before the Index.
  - Reference the **Template** section of this skill as the source of
    truth for module file structure.
- One `ARCHITECTURE/<name>.md` per identified subsystem, each following
  the Template.
- If `AGENT.md` / `CLAUDE.md` don't exist, create them as symlinks to
  `ARCHITECTURE.md`. If they exist as regular files, ask the user
  before replacing.

If the user explicitly asks to deviate from any principle in the Coding
Discipline block, add a short *Project-specific deviations* sub-section
under it rather than editing the principles themselves.

### 4. Add-module path — questions per new module

For each module the user wants to add, ask all of the following — do
not proceed without answers:

1. **Module name and source paths.** Which files / directories does
   this subsystem own?
2. **Goal.** What does this subsystem exist to do? Which roadmap
   milestone(s) (`Mx`) does it serve? Tie it to the headline mission.
3. **Status.** Is it `done`, `in progress (Mx)`, `pending (Mx)`, or
   `scaffolding`? If scaffolding, what replaces it and when?
4. **Design details.** What are the 3–10 most important types /
   functions / entry points? Don't ask the user to type these — read
   the source and pull them out yourself with `file:line` references.
   Then confirm any non-obvious design choice with the user (why this
   data structure? why this trap path? why this protocol?).
5. **Interactions.** Which other `ARCHITECTURE/<module>.md` files does
   this depend on or feed into? Read the existing Index. Cross-link
   bidirectionally where it makes sense.
6. **How to prove it's working.** Ask the user explicitly: *"What
   command(s) verify this subsystem is working end-to-end? What does
   passing look like?"* Prefer a Makefile target, a test runner entry,
   or a shell/REPL command, plus the exact expected output substring or
   exit code. Tests must prove the milestone in the **Status** line,
   not just "code compiles."
7. **Open gaps.** What's deferred, what's missing on the target
   platform, what's tagged for a future milestone?

Then:

- Write `ARCHITECTURE/<module>.md` following the Template. Section
  headers must be verbatim — the verification step is grep-based.
- Add a one-line entry to the Index in `ARCHITECTURE.md`. Use the
  format `- [name.md](ARCHITECTURE/name.md) — one-line hook.` and keep
  the hook under ~120 characters.
- Do NOT duplicate the module's prose back into `ARCHITECTURE.md` — the
  control center carries only cross-cutting material.

### 5. Exit planning mode

Present the plan for explicit user approval before writing anything.

- If the harness has an *exit-plan-mode* primitive, invoke it with the
  plan ready for review; the harness handles the approval gate.
- Otherwise, summarise the plan inline and ask the user a direct
  *"approve?"* question. Wait for an unambiguous yes before any
  *write-file* call runs.

After approval, execute the writes, then run the **Verification** block
below.

## Template

Every `ARCHITECTURE/<module>.md` must use exactly these section headers,
in this order. Copy them verbatim — the verification step is grep-based
and any typo (e.g. `## Tests` instead of `## How to Test`) will trip it.

````markdown
# <Subsystem name>

## Goal

What this subsystem exists to do. Tie back to the project's roadmap
milestone(s) (`M1`..`Mx` in the control center). One short paragraph.
If the subsystem doesn't serve a milestone, say so — "infrastructure
under every milestone" or "scaffolding, no milestone" are valid
answers.

## Status

One of:

- `done` — implemented and tested; the **How to Test** command passes.
- `in progress (Mx)` — partial; explain what's missing.
- `pending (Mx)` — not started; explain what's blocking it.
- `scaffolding` — works today but slated for replacement; explain what
  replaces it and when.

## Code Structure

File-by-file table of the source files in this subsystem. Use absolute
paths from the repo root (e.g. `kernel/src/memory/buddy.rs`, not
`buddy.rs`).

| File | Role |
| ---- | ---- |
| `path/to/file.rs` | one-line description of what it owns |

If the subsystem is a single file, the table is one row — still include
it. Consistency is what makes the doc set scannable.

## Key Types and Entry Points

The 3–10 most important structs, traits, and functions in this
subsystem, with `file:line` references. The goal: a reader (or agent)
should be able to jump straight from this section to the load-bearing
code.

- `path/to/file.rs:42` — `TypeName` — one-line role.
- `path/to/file.rs:123` — `function_name(args)` — what it does.

## Interactions

Which other subsystems this one calls into or is called from. Reference
other `ARCHITECTURE/<module>.md` files by name; cross-link with
markdown.

- Called by [boot.md](boot.md) during init.
- Calls [memory.md](memory.md) for buffer allocation.
- Feeds [transformer.md](transformer.md) with parsed tensor info.

## How to Test

Concrete commands the user or agent runs to verify this subsystem
works. Prefer Makefile targets (`make test-console`, `make build`),
test-runner entries, or REPL/shell commands. **State what "passing"
looks like** — expected output substring, exit code, file produced.

```sh
make build              # what success looks like
```

At the `<project>>` prompt:

- `command-name` — pass = `[o] expected substring`.

Tests must prove the milestone in the **Status** line, not just "code
compiles."

## Open Gaps / Roadmap

Bullet list of known TODOs, deferred work, platform discrepancies, or
follow-up milestones. Tag each with its `Mx` milestone where known.

- **M7**: real-hardware bring-up not yet validated.
- Hugepage support deferred until [memory.md](memory.md) lifts the
  order-20 cap.
- No interrupt-driven I/O yet — polls today.
````

## Coding Discipline

Every `ARCHITECTURE.md` produced by this skill must include the block
below verbatim, as a top-level `## Coding Discipline` section placed
before the Index. The eight principles give future agents a fixed
standard to write *and* review code against — the rules applied at
review time are the rules followed at write time.

The bootstrap workflow (§3) drops this in. The add-module workflow (§4)
inherits it through the existing control center — module files do not
repeat the discipline.

````markdown
## Coding Discipline

This project's code is written and reviewed against the following
eight principles. The standards applied when reviewing are the same
standards followed when writing.

1. **Module Depth.** Substantial functionality behind a simple
   interface. Maximize the gap between interface complexity and
   implementation complexity. Avoid trivial wrappers, one-use methods,
   and shallow classes. Prefer general-purpose interfaces; keep the
   common case simple.

2. **Information Hiding.** Encapsulate design decisions (data
   structures, algorithms, assumptions) inside one module. `private`
   alone is not hiding — getters/setters that expose internals still
   leak. Watch for back-door leakage (multiple modules knowing the
   same format) and temporal decomposition (one class per execution
   phase when the same knowledge is reused).

3. **Abstraction Layers.** Each layer presents a different abstraction
   from the layers above and below. Eliminate pass-through methods and
   pass-through variables. Pull complexity down into modules rather
   than pushing it up to callers; internal representation should
   differ from the external interface.

4. **Cohesion & Separation.** Together-or-apart decisions matter at
   every scope. Combine code that shares information, is always
   co-used, or can't be understood independently. Separate
   general-purpose mechanisms from special-purpose logic. Split
   methods to clarify abstractions, never just to shorten them.

5. **Error Handling.** Reduce the *places* where errors must be
   handled. Define errors out of existence by redesigning APIs so the
   "exceptional" case is normal. Mask exceptions at low levels;
   aggregate handling at high levels. Crash on unrecoverable errors
   instead of layering speculative recovery.

6. **Naming & Obviousness.** Names should make a reader's first guess
   correct. Avoid `data`, `info`, `count`, `status` — be precise
   (`fileBlock`, not `block`). Use a given name for one concept
   everywhere, and never reuse it for a different concept. Match
   reader expectations; document anything that violates them.

7. **Documentation.** Comments capture what code cannot: rationale,
   intent, invariants, units, null/edge semantics. Don't restate what
   the code already shows. Write interface docs while designing — it
   forces clearer abstractions. Cross-module design decisions live in
   obvious central locations, not buried in implementations.

8. **Strategic Design.** Working code is not enough. Invest ~10–20% of
   dev time in design quality; every modification should leave the
   design better than it was. Develop in increments of abstractions,
   not features. Avoid premature optimization, but stay aware of
   fundamentally expensive operations (network, disk, allocation,
   cache misses).
````

## Verification

After the user approves the plan and the files are written, run these
to confirm compliance — preferably as a `Bash` block so the user sees
the output:

```sh
# 1. Every module mentioned in the Index has a corresponding file.
ls ARCHITECTURE/

# 2. Every module file has all seven required sections.
for f in ARCHITECTURE/*.md; do
  missing=""
  for s in "## Goal" "## Status" "## Code Structure" \
           "## Key Types and Entry Points" "## Interactions" \
           "## How to Test" "## Open Gaps / Roadmap"; do
    grep -qF "$s" "$f" || missing="$missing [$s]"
  done
  [ -z "$missing" ] && echo "[ok] $f" || echo "[MISS] $f -$missing"
done

# 3. Control center includes the Coding Discipline block.
grep -qF "## Coding Discipline" ARCHITECTURE.md \
  && echo "[ok] ARCHITECTURE.md has Coding Discipline" \
  || echo "[MISS] ARCHITECTURE.md is missing Coding Discipline"

# 4. Symlinks intact (if applicable).
readlink AGENT.md CLAUDE.md 2>/dev/null

# 5. Spot-check: pick one "How to Test" command from a new module file
#    and confirm it runs (e.g. grep for it in Makefile or test runner).
```

If any module file fails the template check, fix it before declaring
the task complete.

## Rules

1. **Single source of truth.** If a file under `ARCHITECTURE/` describes
   a subsystem, no other file (including `ARCHITECTURE.md`) should
   repeat that subsystem's prose. Cross-link instead.
2. **Update the Index.** Adding a new `ARCHITECTURE/<name>.md` without
   adding the Index entry means the file might as well not exist —
   agents start at the control center and follow the index from there.
3. **Keep line refs current.** When code moves, fix the `file:line` refs
   in **Key Types and Entry Points**. Stale refs are worse than no
   refs.
4. **Tie tests to milestones.** **How to Test** must produce a result
   that proves the milestone in the **Status** line, not just "the code
   compiles."
5. **Don't invent.** If you don't know the answer to one of the
   per-module questions, ask the user. Don't fill in plausible-sounding
   placeholders.
