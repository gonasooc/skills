# Skills

[Korean README](docs/README.ko.md)

A collection of [Agent Skills](https://agentskills.io) for Claude Code and other coding agents.

## Install

Pick the skills you want with the [skills.sh](https://skills.sh) installer:

```bash
npx skills@latest add gonasooc/skills
```

Or install the whole collection as a Claude Code plugin:

```
/plugin marketplace add gonasooc/skills
/plugin install gonasooc-skills@gonasooc
```

### Codex, Gemini CLI, and Antigravity

These skills follow the open [Agent Skills](https://agentskills.io) standard, so any compliant agent runs them unmodified.

- **Codex** — the skills.sh installer above can target Codex directly: pick Codex when prompted.
- **Gemini CLI / Google Antigravity / manual install** — copy (or symlink) a skill folder into the shared standard path: `~/.agents/skills/` (user-wide) or `.agents/skills/` inside a repository. Codex, Gemini CLI, and Antigravity all read this path for project skills. (Global skills live in each agent's own directory — e.g. `~/.gemini/skills/` for Gemini CLI — so check your agent's docs.)

Invoke with `$interviewer`, the `/skills` picker, or just ask in natural language. Agent-specific files like `agents/openai.yaml` are ignored by agents that don't use them.

## Skills

| Skill | Description |
| --- | --- |
| [interviewer](skills/interviewer/SKILL.md) | Artifact-grounded technical interviewer. Give it a repository or an article you produced and it interviews you about your own work — verifying every claim against the source, drilling into shallow answers, and leaving a weakness-map report when the session ends. Built to repay the cognitive debt of AI-assisted work: reading summaries creates the illusion of understanding; being interviewed forces retrieval. |

## Why

Work shipped with AI assistance accumulates cognitive debt — code and prose its owner cannot explain. Passive summaries don't repay it. These skills exist to force the recall that does.

## License

MIT
