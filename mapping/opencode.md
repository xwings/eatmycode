# OpenCode — harness mapping

Maps the generic verbs used in [`../SKILL.md`](../SKILL.md) to the
concrete primitives OpenCode (sst/opencode) exposes.

## Install paths

| Scope   | Native path                                       | Compat path (also discovered)                     |
| ------- | ------------------------------------------------- | ------------------------------------------------- |
| Project | `.opencode/skills/eatmycode/SKILL.md`             | `.claude/skills/eatmycode/` or `.agents/skills/eatmycode/` |
| User    | `~/.config/opencode/skills/eatmycode/SKILL.md`    | `~/.claude/skills/eatmycode/` or `~/.agents/skills/eatmycode/` |

OpenCode walks up from the current working directory to the git
worktree and loads any matching `SKILL.md` from `.opencode/skills/`,
`.claude/skills/`, or `.agents/skills/`. This means one copy installed
for Claude Code or Codex is also discovered by OpenCode automatically.

Disable Claude-Code skill discovery with
`OPENCODE_DISABLE_CLAUDE_CODE_SKILLS=1`; disable all external skill
sources with `OPENCODE_DISABLE_EXTERNAL_SKILLS=1`.

## Invocation

| Style    | How                                                                              |
| -------- | -------------------------------------------------------------------------------- |
| Explicit | Reference the skill by name in your prompt (e.g. *"use the eatmycode skill"*)    |
| Implicit | OpenCode's native skill tool surfaces the skill when the prompt matches its `description` |

Skill permissions are filtered by agent — the agent must have the
`skill` permission to see or use a skill.

## Verb → primitive

| Verb              | OpenCode primitive                                                                       |
| ----------------- | ---------------------------------------------------------------------------------------- |
| `enter-plan-mode` | No native plan mode — state inline: *"I'm planning; I will not write until you approve."* |
| `ask-user`        | Ask inline in chat (one question or short batch); wait for the user's reply             |
| `run-subagent`    | Use a custom agent (configured in `.opencode/agent/`) or run the inventory inline       |
| `exit-plan-mode`  | Summarise the plan inline and ask *"approve?"* — wait for an unambiguous yes            |
| `write-file`      | OpenCode's built-in file-write capability                                                |
| `run-shell`       | OpenCode's built-in shell capability, also available in commands via `!` placeholders   |

## Notes

- OpenCode also supports custom commands in `.opencode/command/`
  (markdown files with `$ARGUMENTS`, `@file`, `!cmd` placeholders).
  This skill does not require any; everything runs from `SKILL.md`.
- Persistent project guidance complementary to this skill belongs in
  `AGENTS.md` (or `opencode.json` `instructions` array).

## Reference

- <https://opencode.ai/docs/skills/>
- <https://opencode.ai/docs/rules/>
- <https://opencode.ai/docs/commands/>
