# OpenClaw — harness mapping

Maps the generic verbs used in [`../SKILL.md`](../SKILL.md) to the
concrete primitives OpenClaw exposes. OpenClaw is AgentSkills-compatible
— same `SKILL.md` + YAML frontmatter format as the other three
harnesses.

## Install paths

| Scope     | Path                                              |
| --------- | ------------------------------------------------- |
| Project   | `.openclaw/skills/eatmycode/SKILL.md`             |
| Workspace | `.agents/skills/eatmycode/SKILL.md` (also discovered) |
| User      | `~/.openclaw/skills/eatmycode/SKILL.md`           |
| User alt  | `~/.agents/skills/eatmycode/SKILL.md` (also discovered) |

OpenClaw scans workspace, project-agent, and user-level skill roots plus
any extra dirs configured by the harness. One copy under
`~/.agents/skills/eatmycode/` covers OpenClaw and Codex CLI; under
`.agents/skills/eatmycode/` it also covers OpenCode.

You can also install from the public registry (ClawHub) by running
`npx skills add <owner>/<repo>` from inside a `.openclaw` directory.

## Invocation

| Style    | How                                                                              |
| -------- | -------------------------------------------------------------------------------- |
| Explicit | Reference the skill by name in your prompt (e.g. *"use the eatmycode skill"*)    |
| Implicit | OpenClaw surfaces the skill when the prompt matches its frontmatter `description` |

## Verb → primitive

| Verb              | OpenClaw primitive                                                                       |
| ----------------- | ---------------------------------------------------------------------------------------- |
| `enter-plan-mode` | No native plan mode — state inline: *"I'm planning; I will not write until you approve."* |
| `ask-user`        | Ask inline in chat (one question or short batch); wait for the user's reply             |
| `run-subagent`    | Delegate via OpenClaw's agent-harness SDK if available; otherwise inventory inline      |
| `exit-plan-mode`  | Summarise the plan inline and ask *"approve?"* — wait for an unambiguous yes            |
| `write-file`      | OpenClaw's built-in file-write capability                                                |
| `run-shell`       | OpenClaw's built-in shell capability                                                     |

## Notes

- OpenClaw has no hard plan-mode gate. The skill enforces "no writes
  before approval" via instruction discipline, not a harness primitive.
  Be strict about it.
- Optional `metadata.openclaw` (single-line JSON) in frontmatter can
  gate the skill by OS or required dependencies; legacy
  `metadata.clawdbot` blocks are also accepted.
- Long-form project guidance complementary to this skill belongs in
  `AGENTS.md` at the repo root.

## Reference

- <https://docs.openclaw.ai/tools/skills>
- <https://docs.openclaw.ai/plugins/sdk-agent-harness>
