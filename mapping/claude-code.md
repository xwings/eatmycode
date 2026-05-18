# Claude Code — harness mapping

Maps the generic verbs used in [`../SKILL.md`](../SKILL.md) to the
concrete primitives Claude Code (Anthropic's CLI / IDE / desktop app)
exposes.

## Install paths

| Scope   | Path                                       |
| ------- | ------------------------------------------ |
| Project | `.claude/skills/eatmycode/SKILL.md`        |
| User    | `~/.claude/skills/eatmycode/SKILL.md`      |

Skills are discovered automatically — no registration step. Frontmatter
fields `name` and `description` are required; `name` must match the
directory name.

## Invocation

| Style    | How                                                                                   |
| -------- | ------------------------------------------------------------------------------------- |
| Explicit | Type `/eatmycode` in the prompt                                                       |
| Implicit | Triggered by the frontmatter `description` when the user's request matches its intent |

Set `disable-model-invocation: true` in frontmatter to force explicit-only.

## Verb → primitive

| Verb              | Claude Code primitive                                                              |
| ----------------- | ---------------------------------------------------------------------------------- |
| `enter-plan-mode` | `EnterPlanMode` tool — opens a read-only planning phase with an approval gate     |
| `ask-user`        | `AskUserQuestion` tool — structured multiple-choice + free-text                    |
| `run-subagent`    | `Agent` tool with `subagent_type: "Explore"` for codebase research                |
| `exit-plan-mode`  | `ExitPlanMode` tool — writes plan to file, asks the user to approve                |
| `write-file`      | `Write` tool (new file) or `Edit` tool (existing file)                             |
| `run-shell`       | `Bash` tool                                                                        |

## Notes

- Plan mode is enforced at the harness level — any non-read-only tool
  call is blocked until the user approves the plan. The skill's
  approval gate piggybacks on this.
- Skills can also be invoked from Claude Desktop, which reads the same
  `~/.claude/skills/` directory.

## Reference

- <https://code.claude.com/docs/en/skills>
