# Codex CLI — harness mapping

Maps the generic verbs used in [`../SKILL.md`](../SKILL.md) to the
concrete primitives OpenAI Codex CLI exposes.

## Install paths

| Scope   | Path                                                            |
| ------- | --------------------------------------------------------------- |
| Project | `.agents/skills/eatmycode/SKILL.md`                             |
| User    | `~/.codex/skills/eatmycode/SKILL.md` (or any `.agents/skills/` walked from cwd up to repo root) |

Codex scans `.agents/skills/` in every directory from the current
working directory up to the repository root, plus user / admin / system
locations. Skills are detected automatically; restart Codex if an
update doesn't appear.

## Invocation

| Style    | How                                                                                       |
| -------- | ----------------------------------------------------------------------------------------- |
| Explicit | Type `$eatmycode` in the prompt, or run `/skills` to pick from a list                     |
| Implicit | Codex picks the skill when the prompt matches its `description` (if `allow_implicit_invocation` is true — the default) |

Disable a skill without deleting it via `[[skills.config]]` in
`~/.codex/config.toml`:

```toml
[[skills.config]]
path = "/path/to/eatmycode/SKILL.md"
enabled = false
```

## Verb → primitive

| Verb              | Codex CLI primitive                                                                     |
| ----------------- | --------------------------------------------------------------------------------------- |
| `enter-plan-mode` | No native plan mode — state inline: *"I'm planning; I will not write until you approve."* |
| `ask-user`        | Ask inline in chat (one question or short batch); wait for the user's reply             |
| `run-subagent`    | Delegate via Codex's Subagents feature when scope is large; otherwise inventory inline  |
| `exit-plan-mode`  | Summarise the plan inline and ask *"approve?"* — wait for an unambiguous yes            |
| `write-file`      | Codex's built-in file-write capability (no separate tool name)                          |
| `run-shell`       | Codex's built-in shell capability                                                       |

## Notes

- Codex doesn't have a hard plan-mode gate. The skill enforces "no
  writes before approval" via instruction discipline, not a harness
  primitive. Be strict about it.
- Long-form project guidance complementary to this skill belongs in
  `AGENTS.md` at the repo root.

## Reference

- <https://developers.openai.com/codex/skills>
- <https://developers.openai.com/codex/guides/agents-md>
