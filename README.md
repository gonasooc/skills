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

Invoke a directly installed skill by name — `/interviewer` in Claude Code or `$interviewer` in Codex — or use the product's skill picker. Claude Code namespaces plugin skills, so the plugin form is `/gonasooc-skills:interviewer`. `interviewer` also responds to a plain natural-language request; `session-checkpoint` and `session-resume` act only after a direct request or explicit invocation. Agent-specific files like `agents/openai.yaml` are ignored by agents that don't use them.

## Skills

| Skill | Description |
| --- | --- |
| [interviewer](skills/interviewer/SKILL.md) | Artifact-grounded technical interviewer. Give it a repository or an article you produced and it interviews you about your own work — verifying every claim against the source, drilling into shallow answers, and leaving a weakness-map report when the session ends. Built to repay the cognitive debt of AI-assisted work: reading summaries creates the illusion of understanding; being interviewed forces retrieval. |
| [session-checkpoint](skills/session-checkpoint/SKILL.md) | Save the current session to `./sessions/` so the work can be picked up on another machine or by another agent. Records the goal, what's verified versus merely assumed, next steps, and the machine-specific environment — grounded in the real git state, not in the model's memory — and promotes decisions that constrain future work into durable records under `sessions/decisions/`. |
| [session-resume](skills/session-resume/SKILL.md) | The other half of `session-checkpoint`. Loads a session document plus the decisions still standing, then checks every claim against the current repository — moved HEAD, missing commits, work left uncommitted on the machine that wrote it — reports where document and reality diverged, and stops for confirmation before doing any work. |

### Working across machines

`session-checkpoint` and `session-resume` are a pair, built for working on one machine with one agent and continuing on another. Everything is written inside the repository you're working in, so it travels with the code over git:

```
sessions/
├── 2026-07-28-143215-auth-retry.md    # a moment — superseded by the next checkpoint
└── decisions/
    └── 2026-07-28-143208-token-in-memory.md  # a constraint — stands until superseded
```

The split matters because the two have different lifetimes. A checkpoint expires the moment work moves on; a decision keeps binding, and burying it in a file named after a Tuesday in July means the next session re-litigates it. `session-resume` reads the decisions and reports which still stand.

Both directories are append-only, and records use second-resolution timestamps rather than sequence numbers like classic ADRs. Sequence numbers need a central allocator — two machines working offline both reach for `004-`. A pre-write existence check prevents local overwrites, while the timestamp reduces (but cannot mathematically eliminate) cross-machine filename collisions. Superseding is a forward link on the new record; the old file is never edited.

Both are **explicitly invoked only**. Each skill starts with an invocation gate and stops if an agent loaded it merely because the surrounding conversation looked relevant; Codex additionally enforces this at the host level with `allow_implicit_invocation: false`. No agent decides on its own that a session is worth recording — `sessions/` stays free of sessions you didn't choose to keep.

Neither skill commits or pushes: `session-checkpoint` writes the file and stops, so a checkpoint reaches your other machine only once you commit and push it yourself.

The names are deliberately two words. Short single words are what CLI vendors take for built-in commands — `/checkpoint` is already an alias of Claude Code's `/rewind`, and `/resume` returns to an earlier conversation. A skill sharing one of those names would be shadowed or would shadow it, and since these skills can only be invoked by typing their name, being shadowed makes them unreachable.

## License

MIT
