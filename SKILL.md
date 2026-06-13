---
name: eatmycode
description: Use when the user wants to create, audit, or maintain an agent-readable architecture doc set with root `ARCHITECTURE.md`, per-subsystem `ARCHITECTURE/<module>.md` files, or the Coding Discipline block. Requires source inspection, planning, and user approval before writing generated docs.
---

# eatmycode

Builds and maintains an agent-readable architecture doc set for a
project. The docs answer two questions: what is this system supposed to
be, and where is each load-bearing decision documented?

## Output Contract

- **`ARCHITECTURE.md`** is the control center. It carries only
  cross-cutting material: mission, target environment, workspace layout,
  boot/entry flow, roadmap, optional address/port map, an Index of
  `ARCHITECTURE/`, and the `## Coding Discipline` block below.
- **`ARCHITECTURE/<module>.md`** is one subsystem file. Every module
  uses the exact Template section order: Goal, Status, Code Structure,
  Key Types and Entry Points, Interactions, How to Test, Open Gaps /
  Roadmap.
- **`AGENT.md` and `CLAUDE.md`** are agent entry files.
  If any exist as regular files, read them as bootstrap source
  material, migrate relevant project guidance into `ARCHITECTURE.md`,
  then replace them with symlinks to `ARCHITECTURE.md` after approval.
  If absent, create symlinks for the names the project or harness uses.

Do not duplicate module prose in `ARCHITECTURE.md`; cross-link to the
owning module file instead.

This skill is harness-agnostic. The workflow uses generic verbs:
*enter-plan-mode*, *ask-user*, *run-subagent*, *exit-plan-mode*,
*write-file*, and *run-shell*. Read `mapping/<your-harness>.md` once at
the start of a session to map those verbs to the local harness.

## Workflow

This skill runs as a small **team, never a solo runner**. Four roles —
**Planner, Coder, Tester, Verifier** — each own a phase. When
*run-subagent* is available, dispatch a separate agent per role; when it
is not, one agent performs each role as a distinct, labeled pass. The
roles never collapse into a single blended step.

- **Planner** — inspect code and docs, then draft the plan: required
  facts, file list, source paths, Index changes, migrated agent-file
  content, and verification commands.
- **Coder** — after approval, write or patch `ARCHITECTURE.md` and
  `ARCHITECTURE/<module>.md` per the approved plan only. Stay surgical.
- **Tester** — run every **How to Test** command and the Verification
  block; report pass/fail with the actual output as evidence.
- **Verifier** — independently re-check the output against the Template,
  Index, `file:line` refs, and the Coding Discipline block, and confirm
  the tests prove each module's Status milestone. Sign off only when the
  result is clean; otherwise hand findings back to the Coder.

### 1. Inspect before asking (Planner)

Detect which path applies:

- `ARCHITECTURE.md` **or** an `ARCHITECTURE/` directory exists -> **update
  path**: read them and update in place per this skill.
- Neither exists -> **create path**: bootstrap the doc set from scratch.
- Any regular `AGENT.md` or `CLAUDE.md` -> migrate its
  durable guidance into `ARCHITECTURE.md` before replacing it with a
  symlink, on either path.

Read existing docs, list `ARCHITECTURE/`, inspect any `AGENT.md` or 
`CLAUDE.md`, and inspect the relevant source before
asking questions. Use *run-subagent* for broad codebase inventory when
available; otherwise inspect directly with shell search. Do not ask the
user to restate facts already present in code or docs.

### 2. Plan before writing (Planner)

Invoke *enter-plan-mode*. If the harness has no plan-mode primitive,
state that this is a planning phase and no files will be written until
the user approves. Capture user answers verbatim in the plan.

Before any *write-file* call, present the planned files, source paths,
Index changes, migrated agent-file content, symlink replacements, and
verification commands. Invoke *exit-plan-mode* if available; otherwise
ask for an explicit approval and wait for an unambiguous yes.

### 3. Create path — bootstrap (Coder)

Ask or infer the minimum project facts needed for a useful control
center:

- **Mission:** headline goal and why this project exists.
- **Target environment:** hardware, OS, runtime, deployment shape, and
  constraints that affect design.
- **Workspace layout:** top-level directories, packages, services, or
  binaries.
- **Entry / boot flow:** process start through "ready".
- **Roadmap:** milestones (`M1`..`MN`) and current status.
- **Maps / constants:** address map, port map, or well-known constants
  only when relevant.

If `AGENT.md` or `CLAUDE.md` already contain project
instructions, use them to seed `ARCHITECTURE.md`:

- Preserve durable facts: mission, workflow rules, coding standards,
  repository layout, commands, roadmap, constraints, and known
  subsystem notes.
- Drop stale harness boilerplate, duplicate prose, and instructions
  that conflict with this skill unless the user explicitly keeps them.
- Ask about conflicts or unclear facts; do not guess which existing
  agent file is authoritative.

Then inventory top-level subsystems. After approval, write:

- `ARCHITECTURE.md` with `## Coding Discipline` before the Index.
- One `ARCHITECTURE/<name>.md` per real or agreed subsystem, each using
  the Template.
- `AGENT.md` and `CLAUDE.md` symlinks pointing to
  `ARCHITECTURE.md`. Replace existing regular files only after their
  relevant content has been migrated into `ARCHITECTURE.md`.

If the user needs a project-specific exception to Coding Discipline,
add `### Project-Specific Deviations` under that block; do not edit the
principles themselves.

### 4. Update path — add-module or audit (Coder)

For each module, ask or infer:

- **Name and source paths:** files or directories the subsystem owns.
- **Goal:** purpose, mission tie-in, and served roadmap milestone(s).
- **Status:** `done`, `in progress (Mx)`, `pending (Mx)`, or
  `scaffolding`, with missing/replacement work where relevant.
- **Design details:** read the source and identify 3-10 important
  types, functions, and entry points with `file:line` references. Ask
  only about non-obvious choices.
- **Interactions:** upstream/downstream module docs to link.
- **Verification:** exact command(s), expected output substring, exit
  code, or artifact that proves the Status milestone.
- **Open gaps:** deferred work, platform gaps, and future milestones.

After approval, write or patch `ARCHITECTURE/<module>.md`, add the
Index entry in `ARCHITECTURE.md`, and avoid copying module prose into
the control center.

For audits, compare the existing file against the Template, current
source paths and line refs, Index entry, interactions, and test command.
Patch only stale or missing material. If agent entry files still exist
as regular files, reconcile their durable content into
`ARCHITECTURE.md` and replace them with symlinks after approval.

### 5. Test and verify (Tester then Verifier)

The Coder hands the result to the Tester, who runs the affected module's
**How to Test** commands and the [Verification](#verification) block and
fixes every `[MISS]`. The Verifier then re-checks the output
independently and signs off. Report the test results to the user with
evidence; do not declare the task done on the Coder's word alone.

## Template

Every `ARCHITECTURE/<module>.md` must use exactly these section headers
in this order. Copy the headers verbatim; verification is grep-based.

````markdown
# <Subsystem name>

## Goal

One short paragraph: what this subsystem does, which roadmap milestone
(`M1`..`Mx`) it serves, and whether it is infrastructure or scaffolding
when no direct milestone applies.

## Status

One of:

- `done` - implemented and tested; the How to Test command passes.
- `in progress (Mx)` - partial; explain what is missing.
- `pending (Mx)` - not started; explain what is blocking it.
- `scaffolding` - works today but will be replaced; say by what and
  when.

## Code Structure

| File | Role |
| ---- | ---- |
| `src/<area>/<file>` | one-line description of what it owns |

Use repo-root-relative paths. Include the table even for a single-file
subsystem.

## Key Types and Entry Points

3-10 load-bearing types, functions, commands, or entry points with
current `file:line` references.

- `src/<area>/<file>:42` - `TypeName` - one-line role.
- `src/<area>/<file>:123` - `function_name(args)` - what it does.

## Interactions

Cross-link the module docs this subsystem calls, feeds, or is called by.

- Called by [module-a.md](module-a.md) during startup.
- Calls [module-b.md](module-b.md) for allocation.
- Feeds [module-c.md](module-c.md) with normalized data.

## How to Test

Commands that prove the Status milestone, plus what passing looks like:
expected output substring, exit code, or produced artifact.

```sh
make build              # pass = exit code 0
```

- `your-test-command` - pass = output contains `OK`.

## Open Gaps / Roadmap

Known TODOs, deferred work, platform gaps, or follow-up milestones. Tag
items with `Mx` when known.

- **M3**: end-to-end flow not yet validated against real data.
- Caching deferred until [module-b.md](module-b.md) exposes
  invalidation.
````

## Coding Discipline

Every generated `ARCHITECTURE.md` must include this block verbatim as a
top-level `## Coding Discipline` section before the Index. Module files
do not repeat it. Both code writing and code review must strictly follow
it; it is reproduced from the Karpathy `CLAUDE.md` (heading levels
demoted one step to nest under this section). Source:
<https://raw.githubusercontent.com/multica-ai/andrej-karpathy-skills/refs/heads/main/CLAUDE.md>

````markdown
## Coding Discipline

Behavioral guidelines to reduce common LLM coding mistakes. Merge with
project-specific instructions as needed.

**Tradeoff:** These guidelines bias toward caution over speed. For
trivial tasks, use judgment.

### 1. Think Before Coding

**Don't assume. Don't hide confusion. Surface tradeoffs.**

Before implementing:
- State your assumptions explicitly. If uncertain, ask.
- If multiple interpretations exist, present them - don't pick silently.
- If a simpler approach exists, say so. Push back when warranted.
- If something is unclear, stop. Name what's confusing. Ask.

### 2. Simplicity First

**Minimum code that solves the problem. Nothing speculative.**

- No features beyond what was asked.
- No abstractions for single-use code.
- No "flexibility" or "configurability" that wasn't requested.
- No error handling for impossible scenarios.
- If you write 200 lines and it could be 50, rewrite it.

Ask yourself: "Would a senior engineer say this is overcomplicated?" If
yes, simplify.

### 3. Surgical Changes

**Touch only what you must. Clean up only your own mess.**

When editing existing code:
- Don't "improve" adjacent code, comments, or formatting.
- Don't refactor things that aren't broken.
- Match existing style, even if you'd do it differently.
- If you notice unrelated dead code, mention it - don't delete it.

When your changes create orphans:
- Remove imports/variables/functions that YOUR changes made unused.
- Don't remove pre-existing dead code unless asked.

The test: Every changed line should trace directly to the user's request.

### 4. Goal-Driven Execution

**Define success criteria. Loop until verified.**

Transform tasks into verifiable goals:
- "Add validation" → "Write tests for invalid inputs, then make them pass"
- "Fix the bug" → "Write a test that reproduces it, then make it pass"
- "Refactor X" → "Ensure tests pass before and after"

For multi-step tasks, state a brief plan:

```
1. [Step] → verify: [check]
2. [Step] → verify: [check]
3. [Step] → verify: [check]
```

Strong success criteria let you loop independently. Weak criteria ("make
it work") require constant clarification.

---

**These guidelines are working if:** fewer unnecessary changes in diffs,
fewer rewrites due to overcomplication, and clarifying questions come
before implementation rather than after mistakes.
````

## Verification

After approved writes, run these checks and fix any `[MISS]` before
declaring the task complete:

```sh
# 1. Every Index target exists.
awk -F'[()]' '/^- \[[^]]+\.md\]\(ARCHITECTURE\/[^)]+\.md\)/ {print $2}' ARCHITECTURE.md |
while IFS= read -r f; do
  [ -f "$f" ] && echo "[ok] $f" || echo "[MISS] missing $f"
done

# 2. Every module file has all seven required sections.
for f in ARCHITECTURE/*.md; do
  [ -e "$f" ] || { echo "[MISS] no ARCHITECTURE/*.md files"; break; }
  missing=""
  for s in "## Goal" "## Status" "## Code Structure" \
           "## Key Types and Entry Points" "## Interactions" \
           "## How to Test" "## Open Gaps / Roadmap"; do
    grep -qF "$s" "$f" || missing="$missing [$s]"
  done
  [ -z "$missing" ] && echo "[ok] $f" || echo "[MISS] $f -$missing"
done

# 3. Control center includes Coding Discipline.
grep -qF "## Coding Discipline" ARCHITECTURE.md \
  && echo "[ok] ARCHITECTURE.md has Coding Discipline" \
  || echo "[MISS] ARCHITECTURE.md is missing Coding Discipline"

# 4. Agent entry files, if present, point at ARCHITECTURE.md.
for f in AGENT.md CLAUDE.md; do
  [ -e "$f" ] || continue
  if [ ! -L "$f" ]; then
    echo "[MISS] $f is not a symlink"
  elif [ "$(readlink -f "$f")" = "$(pwd)/ARCHITECTURE.md" ]; then
    echo "[ok] $f -> ARCHITECTURE.md"
  else
    echo "[MISS] $f points to $(readlink "$f")"
  fi
done

# 5. Run at least one real How to Test command from each new or updated
#    module when practical, and record the result for the user.
```

## Non-Negotiables

- `ARCHITECTURE.md` is cross-cutting only; subsystem prose lives once in
  `ARCHITECTURE/<module>.md`.
- Existing `AGENT.md` and `CLAUDE.md` content is source
  material for `ARCHITECTURE.md`; do not overwrite it before migration.
- Module headers, Index entries, and `file:line` refs must stay current.
- New modules and changed module/function contracts must update
  `ARCHITECTURE.md` and the relevant `ARCHITECTURE/<module>.md` in the
  same change when architecture, ownership, data flow, integration
  points, or public behavior are affected.
- **How to Test** must prove the module's Status milestone.
- Unknown facts are questions, not placeholders.
