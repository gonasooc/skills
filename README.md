# Skills

[Korean README](docs/README.ko.md)

A collection of [Agent Skills](https://agentskills.io) for Claude Code and other coding agents. Each one checks its claims against the artifact itself instead of trusting a plausible summary, and says plainly what it could not verify.

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

Invoke a directly installed skill by name — `/commit` in Claude Code or `$commit` in Codex — or use the product's skill picker. Claude Code namespaces plugin skills, so the plugin form is `/gonasooc-skills:commit`. Both skills also respond to a plain natural-language request. Agent-specific files like `agents/openai.yaml` are ignored by agents that don't use them.

## Skills

| Skill | Description |
| --- | --- |
| [commit](skills/commit/SKILL.md) | Commits in whatever convention the repository already uses, instead of imposing one. Reads the format from what the repo declares — commitlint, a commit template, CONTRIBUTING.md — and otherwise infers it from recent history: Conventional Commits, Jira smart commits, ticket prefixes, freeform. Writes compact imperative subjects — summarized, never truncated into a hand-wave. Detects the package manager rather than assuming npm, and runs the project's own build only when code actually changed. Shows the exact message for approval before committing, stages explicit paths, adds no attribution trailers, and reads the message back after committing to prove none were injected. |
| [interviewer](skills/interviewer/SKILL.md) | Artifact-grounded technical interviewer. Give it a repository or an article you produced and it interviews you about your own work — verifying every claim against the source, drilling into shallow answers, and leaving a weakness-map report when the session ends. Built to repay the cognitive debt of AI-assisted work: reading summaries creates the illusion of understanding; being interviewed forces retrieval. |

## License

MIT
