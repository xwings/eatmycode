# Grok Build — harness mapping

Maps the generic verbs used in [`../SKILL.md`](../SKILL.md) to the
concrete primitives xAI's Grok Build CLI exposes. Grok is
Claude-Code-compatible out of the box, so the same `SKILL.md` + YAML
frontmatter format works without modification.

## Install paths

| Scope   | Native path                                  | Compat path (also discovered)              |
| ------- | -------------------------------------------- | ------------------------------------------ |
| Project | `.grok/skills/eatmycode/SKILL.md`            | `.claude/skills/eatmycode/SKILL.md`        |
| User    | `~/.grok/skills/eatmycode/SKILL.md`          | `~/.claude/skills/eatmycode/SKILL.md`      |

Grok walks from the current working directory up to the repo root
looking for `.grok/skills/`, and also reads Claude Code's skill paths
with zero configuration. One copy under `~/.claude/skills/eatmycode/`
therefore covers Claude Code, OpenCode, **and** Grok Build.

Extra discovery paths can be added via `[skills] paths` in the Grok
config.

## Invocation

| Style    | How                                                                              |
| -------- | -------------------------------------------------------------------------------- |
| Explicit | Type `/eatmycode` in the prompt, or run `/skills` to pick from a list            |
| Implicit | Triggered by frontmatter `description` when the user's request matches its intent |

## Verb → primitive

| Verb              | Grok Build primitive                                                                     |
| ----------------- | ---------------------------------------------------------------------------------------- |
| `enter-plan-mode` | No native plan mode — state inline: *"I'm planning; I will not write until you approve."* |
| `ask-user`        | Ask inline in chat (one question or short batch); wait for the user's reply             |
| `run-subagent`    | Spawn an independent child session for inventory work; otherwise inventory inline       |
| `exit-plan-mode`  | Summarise the plan inline and ask *"approve?"* — wait for an unambiguous yes            |
| `write-file`      | Grok's built-in file-write capability                                                    |
| `run-shell`       | Grok's built-in shell capability                                                         |

## Notes

- Grok has no hard plan-mode gate. The skill enforces "no writes before
  approval" via instruction discipline, not a harness primitive. Be
  strict about it.
- Grok also auto-discovers Claude Code marketplaces, plugins, MCP
  servers, hooks, and instruction files (`CLAUDE.md`, `.claude/rules/`).
  Long-form project guidance complementary to this skill can live in
  `CLAUDE.md` or `AGENTS.md` at the repo root.
- Use the TUI's `/skills` modal to inspect what's installed.

## Reference

- <https://docs.x.ai/build/features/skills-plugins-marketplaces>
