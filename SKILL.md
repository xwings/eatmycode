---
name: eatmycode
description: Create, audit, or maintain agent-readable architecture docs and guide autonomous planning, coding, review, verification, and improvement until a change is ready for public or production release.
---

# eatmycode

Build and maintain the project's architecture source of truth, then use
it to write, review, verify, and improve code without losing the goal.

## Output Contract

- `ARCHITECTURE.md` is the control center. Keep only cross-cutting
  material there: mission, target environment, workspace layout,
  boot/entry flow, roadmap, optional address/port map, the Development
  Loop, Coding Discipline, Review Checks, and an Index of
  `ARCHITECTURE/`.
- `ARCHITECTURE/<module>.md` owns one subsystem and follows the Module
  Template below exactly. Do not duplicate its prose in the control
  center; cross-link it.
- Existing regular `AGENT.md` and `CLAUDE.md` files are bootstrap source.
  Migrate their durable guidance before replacing them with symlinks to
  `ARCHITECTURE.md`. If absent, create the symlink names used by the
  project or harness.
- Copy the `## Development Loop`, `## Coding Discipline`, and
  `## Review Checks` sections below into every generated
  `ARCHITECTURE.md`, verbatim and in that order, before the Index. Module
  files do not repeat them.
- Project-specific exceptions go under `### Project-Specific Deviations`
  after the affected shared section. They may strengthen the Definition
  of Done; never weaken or edit the shared text.

## Operating Model

Run four separate roles, using one subagent per role when available or
four distinct labeled passes otherwise:

1. **Planner / Frame** — understand the request, code, docs, and goal;
   define observable success and the plan.
2. **Coder / Write** — make only the planned change.
3. **Tester / Prove** — run behavioral and structural checks; never edit.
4. **Verifier / Review + Gate** — review independently and decide whether
   every release criterion passes; never edit.

Role handoffs are automatic. Do not pause for progress reports, plan
approval, permission to continue, or a review ceremony. Only the Planner
may ask one focused question, and only when a required decision cannot be
found in the request, code, docs, or repository conventions and a wrong
assumption would materially change the result. Otherwise choose the
narrowest repository-consistent interpretation, record it, and continue.

## Architecture Workflow

### 1. Inspect and frame — Planner

Detect the path:

- Existing `ARCHITECTURE.md` or `ARCHITECTURE/` → update or audit in
  place.
- Neither exists → create the doc set.
- Regular `AGENT.md` or `CLAUDE.md` → include migration and symlink
  replacement in either path.

Before asking anything, read existing architecture and agent docs, list
subsystems, and inspect relevant source. Use a subagent for broad
inventory when supported. Never ask the user to restate discoverable
facts.

Perform a distinct planning pass before writing. State the goal and
observable check, assumptions, required facts, files and source paths,
Index changes, agent-file migration, symlinks, and verification commands.
Resolve uncertainty from repository evidence. Ask only under the
Operating Model exception; never ask merely for confirmation. When the
plan is sound, continue immediately.

### 2. Create or update — Coder

For a new doc set, infer and document:

- mission and reason for the project;
- hardware, OS, runtime, deployment shape, and design constraints;
- top-level packages, services, binaries, and directories;
- process start through ready state;
- milestones (`M1`..`MN`) and current status;
- relevant address, port, or well-known constant maps.

Use durable content from existing agent files: mission, workflow rules,
standards, layout, commands, roadmap, constraints, and known subsystems.
Drop stale harness boilerplate and duplication. Resolve conflicts from
repository evidence; return to Planner only when a conflict materially
changes the project contract and no source is authoritative.

Create `ARCHITECTURE.md`, add the three shared sections below before its
Index, create one module file per real subsystem, then create agent-file
symlinks only after migration is complete.

For an update or audit, derive for each affected module:

- owned source paths and mission/roadmap goal;
- status: `done`, `in progress (Mx)`, `pending (Mx)`, or `scaffolding`,
  including missing or replacement work;
- 3–10 load-bearing types, functions, commands, or entry points with
  current `file:line` references;
- linked upstream and downstream module interactions;
- exact test commands and observable passing evidence;
- deferred work, platform gaps, and future milestones.

Patch only stale or missing material and update the Index. When
architecture, ownership, data flow, integration points, module/function
contracts, or public behavior change, update the control center and
owning module doc in the same change.

### 3. Prove, review, and gate — Tester then Verifier

The Tester runs every affected module's **How to Test** commands and the
Verification section below. Each test must prove the module's stated
Status; merely running is insufficient. Every failure becomes an
evidence-backed finding for Coder, as does coverage that Prove finds
missing, duplicated, or obsolete. After every fix, rerun affected tests
and structural checks. Never send a red result to Review.

The Verifier reads full affected files and applies all seven Review
Checks below as separate passes. Every finding cites `file:line`; a check
that did not run does not pass. Verifier returns findings without editing;
Coder fixes only named findings, Tester re-proves, and Verifier
re-reviews. Repeat until the Development Loop's Definition of Done
passes.

## Module Template

Every `ARCHITECTURE/<module>.md` uses these exact headers in this order:

````markdown
# <Subsystem name>

## Goal

State what this subsystem does, which roadmap milestone (`M1`..`Mx`) it
serves, and whether it is infrastructure or scaffolding when no direct
milestone applies.

## Status

Use one status and explain missing or replacement work:

- `done` — implemented; **How to Test** passes.
- `in progress (Mx)` — partially implemented.
- `pending (Mx)` — not started.
- `scaffolding` — works today but will be replaced; state by what and
  when.

## Code Structure

| File | Role |
| ---- | ---- |
| `src/<area>/<file>` | One-line description of what it owns. |

Use repository-root-relative paths. Keep the table for single-file
subsystems.

## Key Types and Entry Points

List 3–10 load-bearing types, functions, commands, or entry points with
current `file:line` references.

## Interactions

Cross-link module docs this subsystem calls, feeds, or is called by.

## How to Test

Give exact commands and observable passing evidence: expected output,
exit code, or artifact.

## Open Gaps / Roadmap

List known TODOs, deferred work, platform gaps, and follow-up milestones.
Tag items with `Mx` when known.
````

## Development Loop

Coding Discipline governs writing; Review Checks govern review. This
loop connects them and defines when work is ready to release.

```text
Frame → Write → Prove → Review → Gate
          ▲          findings      │
          └────────────────────────┘
```

### The loop

**1. Frame.** Convert the request into a goal with an observable check.
Inspect the request, code, docs, and repository conventions; record the
narrowest supported assumptions. Ask one focused question only when a
required decision cannot be discovered or safely inferred and guessing
would materially change the result. Once framed, continue without an
approval pause.

**2. Write.** Make the smallest change that reaches the goal. Add no
unrequested features or abstractions, match local style, touch only
in-scope code, and remove only orphans created by the change.

**3. Prove.** Run relevant tests and retain observable evidence.

*Survey the suite before touching it.* Before adding, changing, merging,
or deleting any test, inventory the whole suite: enumerate every test
file and case name, then read in full each test whose subject, fixtures,
or assertions touch this change. Use a subagent for broad inventory when
supported. From that inventory decide the complete set of test edits at
once — what to change, what to add, what to merge, what to remove — each
backed by `file:line`, then execute only that plan. Never write a test
before the survey, and never discover existing coverage afterward.

The plan obeys four rules:

- **Reuse or extend first.** Add a case to the test that already owns
  the behavior or shares its setup, fixtures, and subject. A new test
  function or file is justified only when the survey found no existing
  test owning the behavior, or when merging would hide which case
  failed.
- **Add only what the goal needs.** A bug fix needs a reproducing
  regression test; a new capability needs a test of its claimed
  behavior. Nothing further.
- **Retire what this change made obsolete.** Delete tests whose behavior
  no longer exists, and merge tests this change turned into duplicates,
  citing the surviving test. Leave unrelated pre-existing tests alone;
  record suspected redundancy under **Open Gaps / Roadmap**.
- **Never delete to reach green.** A failing test is a finding for
  Write. Removal requires evidence that its behavior is gone or is still
  covered elsewhere, cited by `file:line`.

Coverage of claimed behavior must not decrease. A failure returns
directly to Write, never forward to Review.

**4. Review.** Walk all seven Review Checks as separate passes. Read
whole affected files, not only the diff. Every finding needs `file:line`
evidence. Use an independent agent or isolated pass for Fit,
Dependencies, and Security when available.

**5. Gate.** Apply the Definition of Done. Any unticked criterion,
`blocker`, or unresolved `major` returns its evidence to Write. All
criteria passing means the change is ready for public or production
release. There is no separate approval or reporting phase.

### Definition of Done

**Correctness**

- The framed goal and its named check pass.
- Tests cover claimed behavior and pass; a bug fix has a regression test.
- The suite was surveyed before any test was written, changed, or
  deleted; no added test duplicates coverage another test owns, and no
  removal left claimed behavior uncovered.
- The owning module's **How to Test** command passes with evidence.
- The project builds and tests from a fresh clone without local-only
  dependencies.

**Review**

- All seven Review Checks ran; none was skipped or assumed.
- No `blocker` or unresolved `major` remains.
- Nits were applied or consciously declined.

**Legibility and contract**

- A new maintainer can build, test, run, and understand public behavior
  from the docs.
- Every changed line serves the goal; no drive-by formatting, debugging
  remnants, commented-out code, secrets, tokens, or local paths remain.
- Public names, signatures, errors, and recovery are intelligible.
- Architecture docs and `file:line` references are current.
- Breaking changes, deprecations, dependencies, licenses, and attribution
  are handled; commit or PR text explains why.

### Iterating without thrashing

- Every pass closes a named finding and touches only what it names.
- Nits alone do not trigger another pass.
- Re-run Prove after every fix.
- Two no-change passes force Gate re-evaluation: release if Done passes;
  otherwise return the surviving evidence to Frame.
- Three passes on one finding return automatically to Frame for a new
  approach.
- Never widen scope to satisfy a finding. Record out-of-scope work under
  **Open Gaps / Roadmap**.

## Coding Discipline

### 1. Think Before Coding

- Understand the request, code, goal, and repository conventions first.
- Record assumptions and choose the narrowest evidence-backed reading.
- Prefer the simpler approach when it reaches the same verified goal.
- Ask only during planning and only for a required answer that cannot be
  discovered or safely inferred.

### 2. Simplicity First

- Implement only what was requested.
- Do not add single-use abstractions, speculative flexibility, or checks
  for impossible conditions.
- If the implementation is materially larger than the problem, simplify
  it.

### 3. Surgical Changes

- Do not refactor, reformat, or clean up unrelated code.
- Match the surrounding style.
- Remove imports, variables, and functions made unused by this change;
  leave pre-existing dead code alone unless requested.
- Every changed line must trace to the stated goal.

### 4. Goal-Driven Execution

Turn work into verifiable outcomes, then loop until they pass:

- Add validation → invalid inputs are rejected by a named passing test.
- Fix a bug → a regression test fails before the fix and passes after.
- Refactor → behavior tests pass before and after.

Give every plan step its own check. Strengthen vague criteria from
repository evidence before implementation.

## Review Checks

Run every check against every change before merge. Keep checks separate.

Four rules bind all checks:

- **Evidence or no finding.** Every finding cites `file:line`.
- **The repository is authoritative.** Demand only conventions visible
  in the tree.
- **Read files, not only hunks.** Context can invalidate a finding or
  reveal unreachable code, unused parameters, and hidden duplication.
- **Review the change, never the author.** Describe code and impact, not
  how or by whom it was produced.

### 1. Style

Check indentation and local file conventions. Mixed indentation is
`major`; a consistent new file using the wrong local indent is `nit`.
Leave machine-checkable formatting to existing formatters and linters;
never demand unrelated reformatting.

### 2. Naming

Compare new names with nearby precedents before filing a finding. If the
repository is inconsistent, demand nothing. A local mismatch is `nit`;
an inconsistent public name is `major`.

### 3. Duplication

Search distinctive constants, errors, fields, and call sequences—not
only symbol names—for code performing the same job. Cite both sites and
the remedy. Cross-layer duplication is `major`; small local repetition
is `nit`. Similar code with meaningfully different branches is not
duplication.

### 4. Quality

Require followable control flow, errors handled where they occur, and
abstractions proportional to the problem. Swallowed errors,
inappropriate prints, unexplained magic values, and dead branches are
`major`. Remove unrequested configurability, one-caller wrappers, filler
comments, debugging remnants, and unrelated formatting. Missing tests
belong to Prove, not this check.

### 5. Fit

Read `ARCHITECTURE.md` and the owning module doc before the diff. Check
scope, layering, ownership, public-API growth, and performance claims. A
layering violation or unjustified public API is `major`. Architectural or
public-behavior changes must update the relevant docs in the same change.

### 6. Dependencies

Check manifests and imports, maintenance, supply-chain risk, advisories,
install-time behavior, license, transitive cost, and whether the standard
library is sufficient. An unjustified top-level dependency is `major`;
a live advisory or abandoned upstream is `blocker`. Incomplete evidence
does not pass.

### 7. Security

Check both defects and widened exposure: unsafe memory access, unchecked
sizes or offsets, integer overflow, path traversal, unsafe
deserialization, command construction, committed secrets, and unbounded
untrusted input. Trace input to impact; without a reachable path there is
no finding. A real defect is `major`; a trust-boundary break is `blocker`.
Describe the fix without publishing exploit steps.

### Severity and the merge threshold

| Severity | Effect |
| -------- | ------ |
| `blocker` | Must not merge. |
| `major` | Must be resolved before merge. |
| `nit` | Apply or consciously decline. |
| `info` | Context or a question; no action implied. |

Merge only with no `blocker` and no unresolved `major`. A check that did
not run does not pass. Findings feed Write and Gate directly; they do not
create a reporting phase.

## Verification

After every write or fix, verify all affected behavior and structure:

1. Every Index link resolves, and every real subsystem has one owning
   module file.
2. Every module uses all seven Module Template headers in exact order.
3. `ARCHITECTURE.md` contains Development Loop, Coding Discipline, then
   Review Checks before its Index.
4. Code paths and `file:line` references resolve to current source.
5. Agent entry files, when present, are symlinks to `ARCHITECTURE.md` and
   their former durable guidance was migrated.
6. Every new or updated module's **How to Test** command runs and proves
   its stated Status.
7. All seven Review Checks and the Definition of Done pass.

Any miss becomes a named Coder finding. Re-run verification after the
fix, then repeat Review and Gate. The task is complete only when all
checks pass and the result is ready for public or production release.
Provide only the harness's normal concise completion handoff.

## Non-Negotiables

- Inspect before planning; plan before writing; prove and review before
  Gate.
- Preserve agent-file guidance before replacing regular files.
- Keep module prose in one owning module file and links and line
  references current.
- Never mark `done` without a passing **How to Test** command.
- Never delete or weaken a test to turn a red run green.
- Tester and Verifier never repair their own findings.
- Unknown facts come from evidence, the single planning-question
  exception, or an explicit external blocker—never a placeholder.
