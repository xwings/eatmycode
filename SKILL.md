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
  `ARCHITECTURE/`, and the three discipline blocks below —
  `## Development Loop`, `## Coding Discipline`, `## Review Checks` — in
  that order.
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

The four roles are one turn of the [Development Loop](#development-loop),
which this skill both emits and obeys: Planner = Frame, Coder = Write,
Tester = Prove, Verifier = Review and Gate. When the Verifier returns
findings, the loop repeats from Coder — it does not run once by default.

- **Planner** — inspect code and docs, then draft the plan: required
  facts, file list, source paths, Index changes, migrated agent-file
  content, and verification commands.
- **Coder** — after approval, write or patch `ARCHITECTURE.md` and
  `ARCHITECTURE/<module>.md` per the approved plan only. Stay surgical.
- **Tester** — run every **How to Test** command and the Verification
  block; report pass/fail with the actual output as evidence.
- **Verifier** — independently re-check the output against the Template,
  Index, `file:line` refs, and the Coding Discipline and Review Checks
  blocks, and confirm the tests prove each module's Status milestone. Run
  the seven [Review Checks](#review-checks) over the Coder's diff and
  report findings with `file:line` evidence. Sign off only when the
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

- `ARCHITECTURE.md` with `## Development Loop`, `## Coding Discipline`,
  then `## Review Checks`, in that order, before the Index.
- One `ARCHITECTURE/<name>.md` per real or agreed subsystem, each using
  the Template.
- `AGENT.md` and `CLAUDE.md` symlinks pointing to
  `ARCHITECTURE.md`. Replace existing regular files only after their
  relevant content has been migrated into `ARCHITECTURE.md`.

If the user needs a project-specific exception — to a Coding Discipline
principle, a Review Check, or a Definition of Done item — add
`### Project-Specific Deviations` under the affected block. Projects may
*add* Done criteria there (a coverage floor, a benchmark, a changelog
entry); they do not edit or delete the blocks themselves.

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

### 5. Test, review, and gate (Tester then Verifier)

The Coder hands the result to the Tester, who runs the affected module's
**How to Test** commands and the [Verification](#verification) block and
reports every `[MISS]` with its evidence. The Verifier then re-checks the
output independently, walks the seven [Review Checks](#review-checks)
over the diff, and applies the Definition of Done in the
[Development Loop](#development-loop).

Neither the Tester nor the Verifier edits the tree: they produce
findings, the Coder produces fixes. A reviewer who patches what they
found has no independent pass left to confirm it.

This step is a gate, not a formality, and it cycles — write, prove, seven
checks, fix, re-prove, re-check. Unticked box or surviving
`blocker`/`major`: hand the named findings back to the Coder, who repeats
from step 3 or 4 against those findings and nothing else; the Tester and
Verifier then run again over the new diff. Repeat until every Definition
of Done box is ticked. The bar is open-source grade, and it is checked
box by box rather than judged by how finished the work feels. All boxes
ticked: sign off. Honour the anti-thrash rules — every repeat closes a
named finding, two consecutive no-change passes end the loop, and three
passes on one finding send it back to the Planner.

Report to the user with evidence: the test output, the seven-check
checklist, and the pass count if the loop ran more than once. Do not
declare the task done on the Coder's word alone.

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

## Development Loop

The spine that connects the other two blocks. Coding Discipline says how
to write; Review Checks say how to review; this says when to move between
them and when to stop. Every generated `ARCHITECTURE.md` must include it
verbatim as a top-level `## Development Loop` section, first of the three
discipline blocks and before the Index. Module files do not repeat it.

The skill's own four roles are one turn of this loop: Planner = Frame,
Coder = Write, Tester = Prove, Verifier = Review and Gate.

````markdown
## Development Loop

Coding Discipline governs how code is written. Review Checks govern how it
is reviewed. This is the loop that runs them, and the gate that ends it.

Code that runs is not code that is done. Done is defined below, it is
checked rather than felt, and the only way to reach it is to go round.

### The loop

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

**1. Frame — think before touching code.** Restate the task as a goal
with a check attached: not "add validation" but "invalid input is
rejected with a named error, proven by a test that fails today". State
your assumptions out loud. If the task has more than one reading, present
them rather than picking silently. If a simpler approach exists, say so
before building the complicated one. If something is unclear, stop and
ask — an hour of building on a wrong assumption costs more than the
question. For multi-step work, write the plan as steps with a check on
each. Detail: Coding Discipline §1 and §4.

**2. Write — the smallest change that reaches the goal.** No features
beyond the ask, no abstraction for a single caller, no configurability
nobody requested, no error handling for conditions that cannot occur.
Touch only what the goal requires; match the surrounding style even
where you would do it differently; clean up the orphans your own change
created and nothing else. Detail: Coding Discipline §2 and §3.

**3. Prove — produce evidence, not belief.** Run the module's **How to
Test** command and keep the output. A bug fix carries a test that failed
before the change and passes after; a new capability carries a test of
the behaviour it claims. "It should work" is not a result, and neither
is a test you wrote but did not run. If Prove fails, return to Write —
do not carry a red test into Review.

**4. Review — walk all seven checks against your own diff.** Style,
Naming, Duplication, Quality, Fit, Dependencies, Security, in order, one
pass each. Change stance rather than merely re-reading: read the diff as
someone looking for a reason to reject it. Read whole files for
Duplication and Quality — the hunk hides unused parameters, unreachable
branches, and forward-only wrappers. The evidence rule binds against
yourself: a self-review finding without `file:line` is a mood, not a
finding. Where an independent reviewer is available — another agent,
another pass, another person — send Fit, Dependencies, and Security
there; those are the judgements authorship blinds you to hardest.

**5. Gate — check Done, then decide.** Walk the Definition of Done. Every
box ticked: ship, and report the checklist. Any box unticked: name the
findings that block it and return to Write with those findings and
nothing else. Record which box failed — an unexplained extra pass is
indistinguishable from thrashing.

### Definition of Done

The bar is open-source standard: a maintainer who has never seen this
change, and cannot ask you anything, could review and merge it from the
diff and the docs alone. Concretely, all of:

**Correctness**

- The goal stated in Frame is met, and the check named in Frame passes.
- Tests cover the behaviour the change claims and they pass. A bug fix
  has a test that fails without it.
- The owning module's **How to Test** command passes, output recorded.
- It builds and tests clean from a fresh clone — no uncommitted file it
  depends on, no step that exists only on your machine.

**Review**

- All seven Review Checks walked, each with a recorded status. None
  skipped, none assumed.
- No `blocker`. No unresolved `major`.
- Nits applied or consciously declined.

**Legibility — this is the part that makes it open source**

- A stranger can build, test, and run it from the README alone.
- Public names, signatures, and errors are intelligible without reading
  the implementation. Error messages tell the reader what to do next.
- Every changed line traces to the stated goal. No drive-by
  reformatting, no debugging leftovers, no commented-out blocks, no
  secrets, tokens, or absolute local paths.
- The commit or PR message says *why*. The diff already says what.

**Contract**

- The architecture docs are current: Review Check 5 passes, and existing
  `file:line` references still resolve.
- Breaking changes are called out and justified; deprecations documented.
- Anything vendored, copied, or newly depended upon is
  license-compatible and attributed.

"Good enough for me" is not on this list, and neither is "I feel good
about it" — Coding Discipline §4 rejects exactly that: strong criteria
let the loop run unattended, weak criteria stall it on your mood. When a
box cannot be ticked, the loop is not finished. Say which box and why.

### Iterating without thrashing

The loop has to stop. These rules end it in both directions — too early
and never.

- **Every pass closes a named finding.** "Have another look" is not a
  pass. If you cannot name what this pass fixes, do not start it.
- **A pass touches only what its findings name.** A review finding
  authorises repair, not redesign. Surgical Changes still binds inside
  the loop, and a review-driven rewrite is the most expensive way to
  violate it.
- **Nits alone do not justify a pass.** Batch them into a pass that
  something real triggered, or decline them and say so.
- **Re-run Prove after every pass.** A fix that was not re-tested is not
  a fix, and passes two onward are where regressions enter.
- **Two consecutive passes that change nothing: stop.** Either it has
  converged — ship — or you are stuck. Say which.
- **Three passes against the same finding: return to Frame.** The design
  is wrong, not the code. Iterating on the implementation will not fix
  it.
- **Never widen scope to satisfy a finding.** Work the finding demands
  beyond the stated goal is a follow-up: record it under the module's
  **Open Gaps / Roadmap**, say you deferred it, and leave it there.

**The loop is working if:** defects are found by Review rather than by
users, passes shrink as they go, and Gate is a check you perform rather
than a feeling you have.
````

## Coding Discipline

Every generated `ARCHITECTURE.md` must include this block verbatim as a
top-level `## Coding Discipline` section, after `## Development Loop` and
before the Index. Module files do not repeat it. Both code writing and code review must strictly follow
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

## Review Checks

Coding Discipline governs how code is written. This block governs how it
is reviewed — before a commit, before a pull request, and during any
audit. Every generated `ARCHITECTURE.md` must include it verbatim as a
top-level `## Review Checks` section, directly after
`## Coding Discipline` and before the Index. Module files do not repeat
it.

Adapted from a seven-check review panel in which one specialist owns each
check and a chair writes the single verdict.

````markdown
## Review Checks

Run these seven checks against every change before proposing it for
merge. Each is a separate pass with its own scope; a pass that blends
into another performs neither. Four rules bind all seven:

- **Evidence or no finding.** Every finding cites `file:line`.
  "Inconsistent naming", "this seems unsafe", and "consider refactoring"
  with nothing to point at are not findings and are not filed.
- **The repository is the authority.** A convention nobody can locate in
  the tree is not a convention, and nothing may be demanded on its
  authority. Where a general rule and this repository's actual practice
  disagree, the repository wins.
- **Read the file, not the hunk.** Unused parameters, unreachable
  branches, forward-only wrappers, and a validation check one frame up
  are all invisible in a diff.
- **Review the change, never the author.** Say what the code does. Do not
  assert how it was produced and do not speculate about who wrote it.

### 1. Style

Indentation and formatting, per language and per file. Mixed indentation
inside one file is **major** — it breaks anyone whose editor is set for
the other one. A new file that is internally consistent but uses the
wrong indent is a **nit**. Reindenting lines the change did not otherwise
need to touch is one finding asking for it to be split out, not one
finding per line.

Line length, import order, trailing whitespace, and end-of-line markers
belong to the formatter and linter in CI. Do not spend attention on what
a machine already handles.

### 2. Naming

Whether new names follow this project's conventions — variables,
functions, methods, classes, modules, files, constants, parameters.

Conventions are discovered, never assumed: before calling a name wrong,
find the closest existing names and read how they are spelled. Cite two
or three, then propose the specific replacement. Where the repository is
genuinely inconsistent in an area, say so and demand nothing. A name
defensible in general but foreign here is still a finding; a name ugly in
general but matching ten neighbours is not. **Nit** inside a module,
**major** on a public name — the project lives with public names far
longer than with the change that introduced them.

### 3. Duplication

Whether the change adds something the project already has.

Near-equivalent means same job, not same text: a helper that resolves a
path, parses a structure, or wraps a call the same way under a different
name counts, and copy-pasted blocks across two platform or backend
implementations count loudest. Grep the distinctive strings inside the
new code — a constant, an error message, a field name, an unusual call
sequence — not only the symbol names; the expensive duplicates are the
ones that do not share a name.

Every finding cites both sites and names the remedy: call the existing
helper, merge the two, or lift the shared part out — and say where the
shared code should live. Two functions that look alike and differ in one
branch are not duplicates. Duplication spanning layers is **major**;
small repetition inside one new module is a **nit**.

### 4. Quality

Whether the code is well written, and whether anything is in the change
that should not be there.

Well written means control flow is followable, errors are handled where
they occur, the abstraction matches the problem, and a reader can tell
what the code does without running it. Exception handlers that swallow
without re-raising or logging, prints where the project has a logger,
unnamed magic numbers, and dead branches are **major**.

Extra weight is the half that reviewers skip: code the change does not
need, configurability nobody asked for, abstractions with exactly one
caller, commented-out blocks, debugging leftovers, comments that restate
the line beneath them, docstrings that name every parameter and explain
none, defensive checks for conditions that cannot occur, and unrelated
reformatting smuggled in beside the real change. Say plainly that it can
be deleted and what breaks if it is not. Filler comments and unused
parameters are **nits**. Missing tests are an observation here, not a
finding — CI owns correctness.

### 5. Fit

Whether the change belongs in this project and leaves it better rather
than merely larger.

Read `ARCHITECTURE.md` and the owning `ARCHITECTURE/<module>.md` before
the diff — that ordering is the point of the doc set. Then ask: is this
in scope, or is it something to build on top of the project instead of
inside it? Does it respect the layering, or reach across a boundary the
architecture keeps closed? Does it deliver a fix, a real capability, or a
measured speedup — or add surface, cost, and maintenance burden for a
gain nobody has stated?

Test performance claims for plausibility: a change that claims to be
faster while adding a per-call allocation to a hot path is making things
worse. A new public entry point overlapping one that exists grows the API
the project must support forever. Layering violations and unjustified
public-API growth are **major**. Where the goal is sound but the
placement is wrong, say where it should go.

A change that alters architecture, ownership, data flow, integration
points, or public behaviour without updating `ARCHITECTURE.md` and the
relevant `ARCHITECTURE/<module>.md` in the same change fails this check.

### 6. Dependencies

Whether the change adds a package, and whether it should.

Four questions, in order. Is one actually being added — check the
manifests *and* the imports, because code can import what the manifest
never declared. Is it maintained: last release, cadence, maintainer
count, archived upstream? Is it a supply-chain risk: known advisories, a
very recent first release, a single maintainer holding a
widely-installed name, a name close to a popular one, install-time code
execution? And is it worth it — the honest comparison is the package
plus its transitive tree plus its future breakage, against the
standard-library code it would replace.

The default answer to a new dependency is no and the burden is on the
change to move it, but say so proportionately: a well-maintained package
doing something genuinely hard is a good trade, and pretending otherwise
is not rigour. An unjustified new top-level dependency is **major**; a
live advisory or an abandoned upstream is a **blocker**. When the
evidence cannot be gathered, say the check could not be completed rather
than guessing.

### 7. Security

Whether the change introduces a security defect, and whether it widens
exposure. Two distinct questions.

Defects in the change: memory safety in native code, unchecked lengths
and offsets, integer overflow feeding an allocation or an index, path
traversal, unsafe deserialisation, commands built from input the code
does not control, secrets or tokens committed, untrusted input reaching a
parser without bounds.

Exposure, separately: whatever this project treats as untrusted — user
input, network data, sandboxed or emulated code, third-party content — a
change that lets it reach the host filesystem, a subprocess, the network,
or host memory is a serious finding even when the change contains no bug
of its own.

State the path from input to impact concretely: where the value enters,
what it is not checked against, and what an attacker gets. If you cannot
trace that path you have a question, not a finding — ask it, and withdraw
it once answered. No severity inflation: a theoretical issue with no
reachable path is **info**, a real bug in the change is **major**, and
anything that breaks a trust boundary is a **blocker**. Describe the flaw
and the fix; do not publish exploit steps.

### Severity and the merge threshold

| Severity | Meaning | Effect |
| -------- | ------- | ------ |
| `blocker` | Breaks a trust boundary, corrupts data, or ships a known-vulnerable dependency. | Must not merge. |
| `major` | Wrong, unsafe, or costly enough that a maintainer would ask for a change. | Merge only with a stated, accepted reason. |
| `nit` | Correct, but inconsistent with the project. | Author's call. |
| `info` | Observation, question, or context. | No action implied. |

Merge only when no `blocker` and no unresolved `major` remains. A verdict
of "looks good" alongside a `major` finding is not an approval — resolve
the finding or record why it stands.

### Reporting

Report once, as one document, not seven. Open with what the change is
trying to do and what it gets right. List the findings that survived a
verify pass, ordered by severity, each with its `file:line` and the
reason it matters. Close with what would have to change. Then record the
checklist, so it is visible which checks were actually walked:

| Check | Status | Note |
| ----- | ------ | ---- |
| 1. Style | pass / concern / blocker | evidence, or "nothing found" |
| 2. Naming | | |
| 3. Duplication | | |
| 4. Quality | | |
| 5. Fit | | |
| 6. Dependencies | | |
| 7. Security | | |

A check nobody could complete is a **concern**, not a pass. Never report
a check that did not run: a confident verdict covering skipped checks
reads exactly like a real one, which makes it worse than no review.
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

# 3. Control center includes all three discipline blocks.
for s in "## Development Loop" "## Coding Discipline" "## Review Checks"; do
  grep -qF "$s" ARCHITECTURE.md \
    && echo "[ok] ARCHITECTURE.md has $s" \
    || echo "[MISS] ARCHITECTURE.md is missing $s"
done

# 3b. All seven checks are present under Review Checks.
for s in "### 1. Style" "### 2. Naming" "### 3. Duplication" \
         "### 4. Quality" "### 5. Fit" "### 6. Dependencies" \
         "### 7. Security" "### Severity and the merge threshold"; do
  grep -qF "$s" ARCHITECTURE.md \
    && echo "[ok] $s" \
    || echo "[MISS] Review Checks is missing $s"
done

# 3c. The loop is complete: five stages, the gate, and the stop rules.
for s in "### The loop" "### Definition of Done" \
         "### Iterating without thrashing"; do
  grep -qF "$s" ARCHITECTURE.md \
    && echo "[ok] $s" \
    || echo "[MISS] Development Loop is missing $s"
done

# 3d. The blocks appear in loop order: Loop, then Discipline, then Checks.
awk '/^## Development Loop$/{a=NR} /^## Coding Discipline$/{b=NR}
     /^## Review Checks$/{c=NR}
     END{ if (a && b && c && a<b && b<c) print "[ok] discipline blocks in order";
          else print "[MISS] discipline blocks out of order or absent" }' ARCHITECTURE.md

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

# 6. Walk the seven Review Checks over the diff this session produced and
#    report the checklist. A check nobody could complete is a concern,
#    not a pass.

# 7. Apply the Definition of Done. Any unticked box sends the named
#    findings back to the Coder, who fixes those findings and nothing
#    else; then re-run checks 1-6 over the new diff and apply Done
#    again. Repeat until every box ticks — open-source grade is the bar,
#    and it is checked box by box, not felt. Report the pass count when
#    the loop ran more than once.
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
- `ARCHITECTURE.md` carries `## Development Loop`, `## Coding Discipline`,
  and `## Review Checks` verbatim, in that order. Exceptions go under
  `### Project-Specific Deviations`, never into the blocks themselves.
- Writing code is not the end of the loop. Review and Gate run before
  anything is called done, and the loop repeats while findings remain.
- The seven checks report; they do not repair. Findings go back to the
  Coder, the fix is re-proved and re-checked, and the cycle runs again.
- Done is the Definition of Done, checked box by box — never "it runs",
  never "it looks fine". An unticked box is named, with its reason.
- No review finding without `file:line`, and no reported check that did
  not actually run.
- Unknown facts are questions, not placeholders.
