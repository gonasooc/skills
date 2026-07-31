---
name: session-checkpoint
description: Write a session checkpoint into ./sessions/ so the work can be picked up on another machine or by another agent. Records the goal, what is verified versus assumed, next steps, open questions, and the machine-specific environment — all grounded in the real git state rather than in memory — and promotes decisions that constrain future work into durable records under ./sessions/decisions/. Explicitly invoked only; pass what the next session should focus on as an argument.
allowed-tools: Bash(git status:*) Bash(git log:*) Bash(git rev-parse:*) Bash(git rev-list:*) Bash(git check-ignore:*) Bash(date:*)
---

# Session checkpoint

Write the current session to `./sessions/` in this repository, so it can be resumed on a different machine, by a different agent, or by the user weeks later.

**Why this exists.** Work moves between machines — a work laptop running one agent, a home machine running another. The code moves with git. The context does not: why this approach was chosen and what was rejected, which claims were actually verified and which were assumed, and what is sitting half-finished and uncommitted on the machine you just left. A checkpoint writes that context down in a form any agent can read.

It is invoked explicitly, never automatically. A session log is worth keeping only when the user decides it is; the rest should leave no trace.

**Invocation gate.** Continue only when the user directly requested a checkpoint or explicitly invoked `session-checkpoint` (including a client-specific namespaced form). If the skill was loaded only because an agent inferred it from context, explain that it is manual-only and stop without writing.

The counterpart is the `session-resume` skill, which reads these files back and checks them against reality.

## Two kinds of writing

A checkpoint is dated and disposable: it describes one moment and stops being interesting once the work moves on. But some things a session produces outlive it — the decisions that constrain what anyone may do next. Those must not be buried in a file named after a Tuesday in July.

So this skill writes to two places:

| | Path | Lifetime |
| --- | --- | --- |
| **Checkpoint** | `sessions/YYYY-MM-DD-HHmmss-<slug>.md` | Superseded by the next checkpoint |
| **Decision record** | `sessions/decisions/YYYY-MM-DD-HHmmss-<slug>.md` | Stands until a later decision supersedes it |

The test for promoting something to a decision record: **could someone who does not know this undo it** — by re-litigating the same question, or by writing code that contradicts it? If yes, it earns its own file. A tactical choice scoped to this session's work stays in the checkpoint.

## The document is the product

Everything below serves one test: **someone on a different machine, with no memory of this conversation, can read the file and take the next action without asking a question.** If a section does not serve that test, cut it.

## Rules

1. **Ground every fact in a tool call, not in memory.** Date, branch, HEAD commit, upstream state, and working-tree cleanliness — read them from the shell *in this session*, immediately before writing. A checkpoint whose commit hash is wrong is worse than no checkpoint: `session-resume` trusts these fields to detect drift.
2. **"Verified" and "assumed" are different words — use them precisely.** A test you actually ran and saw pass is verified. A test you believe would pass is assumed. Code you wrote but never executed is assumed. This distinction is the single most valuable thing in the file, because the next session will otherwise build on top of it. When in doubt, downgrade to assumed.
3. **Reference, never duplicate.** Specs, ADRs, issues, diffs, commits, and plans already exist somewhere. Point at them by path, URL, or commit hash. Restating them creates a second copy that goes stale silently.
4. **Redact.** No API keys, tokens, passwords, connection strings, hostnames, or personal data. For environment variables, record the **name only** — never the value. Use a non-sensitive machine label the user already supplied, or `undisclosed`; never collect the system hostname just to fill the field. These files are committed to a repository that other people may read.
5. **Write only inside `sessions/`.** Never modify code, configuration, or any other file to make the checkpoint tidier. Never stage, commit, or push — the user syncs on their own terms.
6. **Write for a stranger's agent.** Another agent on another machine will read this. Do not lean on anything specific to the current tool: no session IDs, no internal tool names, no "as we discussed above." Plain prose and real paths only.
7. **Do not invent.** If the session established nothing for a section, write `없음` / `None`. An empty section is information; a plausible-sounding filler sentence is a trap.
8. **Never overwrite.** Check every destination immediately before writing. If a generated filename already exists, add `-2`, `-3`, and so on before `.md`. Do not edit or replace an existing checkpoint or decision record.

## Flow

### 1. Collect ground truth

Run these before writing anything:

```bash
git rev-parse --show-toplevel          # repo root — sessions/ goes here
git rev-parse --abbrev-ref HEAD        # branch
git rev-parse HEAD                     # full commit ID; do not abbreviate it
git status --porcelain=v1              # empty output means a clean tree
git rev-parse --abbrev-ref --symbolic-full-name '@{upstream}'
git rev-list --left-right --count '@{upstream}...HEAD'
git log --oneline -5                   # recent commits, for the References section
date "+%Y-%m-%dT%H:%M%z"               # timestamp with offset
date "+%Y-%m-%d-%H%M%S"                # filename prefix
```

The two upstream commands may fail when the branch has no upstream. Record that state as `upstream: none`; do not guess a remote. `git rev-list --left-right --count '@{upstream}...HEAD'` prints **behind first, ahead second**. The counts are relative to the local remote-tracking ref because this skill does not fetch. Record that limitation, and use the ahead count plus `git log '@{upstream}..HEAD' --oneline` to identify commits that are not known to be pushed.

For `machine`, use a non-sensitive label the user already supplied in the conversation. Otherwise write `undisclosed`. Do not call `hostname` or derive a personal machine name.

If this is not a git repository, say so and ask where `sessions/` should live before continuing.

### 2. Decide whether there is anything worth recording

The user asked for this, so the default is to write. But if the session produced nothing another machine would need — no decisions, no partial work, no open questions — say that plainly and ask before creating a file. Clutter in `sessions/` costs the `session-resume` skill its signal.

### 3. Promote durable decisions

Apply the promotion test above to everything the session settled. For each decision that passes it, write `<repo-root>/sessions/decisions/YYYY-MM-DD-HHmmss-<slug>.md` in the format below.

First read the existing filenames in `sessions/decisions/`. If this decision reverses or replaces one of them, set `supersedes:` to that filename. **Do not edit the older file** — not even to mark it obsolete. It stays exactly as written; `session-resume` derives what is still standing by following `supersedes:` forward. Append-only records avoid the routine merge conflicts caused by editing shared historical files.

If nothing passes the test, create nothing. Most sessions produce no decision records, and that is the normal case.

### 4. Write the checkpoint

Path: `<repo-root>/sessions/YYYY-MM-DD-HHmmss-<slug>.md`

- Create `sessions/` if it does not exist.
- `<slug>` is 2–4 kebab-case words naming the work, **ASCII only** — non-ASCII filenames normalize differently across macOS and Linux and break the round trip.
- **One file per checkpoint. Never append to or edit an existing one.** Second-resolution timestamps make cross-machine collisions less likely; the pre-write existence check prevents local overwrites. Do not claim that distributed collisions are impossible.
- If this checkpoint continues an earlier one, set `continues:` to that filename rather than restating its content.

### 5. Report back

Print every path you wrote — the checkpoint, and each decision record. Then run `git check-ignore -q sessions/` — if `sessions/` is ignored, say so directly: the files exist locally but will never reach the other machine. Otherwise remind the user that they sync only once they commit and push, since this skill deliberately does neither.

## Document format

**The `##` headings below are a fixed contract** — `session-resume` locates sections by them. Keep them verbatim and in this order, in English, even when the prose inside is Korean. Write the prose in whatever language the user has been using.

```markdown
---
date: 2026-07-28T14:32+0900
machine: <non-sensitive label, or undisclosed>
agent: <the agent writing this — e.g. claude-code, codex>
branch: <branch>
commit: <full commit ID>
tree: clean | dirty
upstream: <remote branch, or none>
ahead: <integer, omit when upstream is none>
behind: <integer, omit when upstream is none>
status: in-progress | blocked | done
continues: <earlier filename, or omit>
---

# <one-line title>

## Goal
What this work is trying to achieve, in one to three sentences. The outcome, not the activity.

## State
What is done, split explicitly:
- **Verified** — things observed working. Name how they were observed (test run, command output, manual check).
- **Assumed** — written but never executed or confirmed.
- **Not started** — in scope but untouched.

## Next steps
Ordered. The first item must be executable immediately by someone who has read nothing else — an exact file, command, or decision.

## Decisions
Link every decision record written this session: `- [<title>](decisions/<filename>)`. Below those, list the tactical choices that did not earn a record — each with its reason and the alternative rejected. Never restate the body of a linked record here.

## Open questions
Unresolved questions and blockers. For a blocker, name what specifically is blocked and what would unblock it.

## Environment
Anything true on this machine that will not be true on the next one:
- Uncommitted or unpushed work, and where it lives. **Say this loudly** — it is the most common way a cross-machine handoff loses work.
- For uncommitted work, name the affected paths from `git status --porcelain=v1`; do not imply that a dirty tree travels with the checkpoint.
- For unpushed work, name the commits reported ahead of the local upstream. If there is no upstream, say that push state is unverifiable. If the remote-tracking ref was not fetched in this session, say the result may be stale.
- Environment variables the work needs, **by name only**.
- Local services, ports, containers, or databases that must be running.
- Tool or runtime versions that mattered.
- Files needed but not in git.

## References
Paths, URLs, issue and PR numbers, commit hashes. Pointers only — no summaries of what they contain.

## Suggested skills
Skills the next agent should invoke, and when. Omit the section if none apply.
```

## Decision record format

Same contract: the `##` headings are fixed and English.

```markdown
---
date: 2026-07-28
agent: <the agent writing this>
supersedes: <filename in this directory, or omit>
---

# <the decision, stated as a claim — not a question or a topic>

## Context
What forced a choice. The constraint, not the history.

## Decision
What was decided, in the imperative. Someone must be able to comply with this sentence.

## Rejected alternatives
What else was viable and why it lost. Without this, the decision gets re-litigated by the next person who thinks of the obvious alternative.

## Consequences
What this now commits the project to, including anything it makes harder.
```

Two conventions that differ from classic ADRs, both for the same reason — two machines write into this directory independently:

- **Timestamps, not sequence numbers.** Numbered ADRs (`001-`, `002-`) require a central allocator. Two machines offline both reach for `004-` and collide on the next pull.
- **Forward links only.** A superseding record names what it replaces; the replaced record is never touched. Back-links would mean editing old files, which is exactly what conflicts.

Multiple decisions on one day are fine — the timestamp and slug distinguish them.

## Length

Aim for one screen. A checkpoint long enough to need skimming will get skimmed, and the drift warnings in `Environment` are exactly what a skimmer misses. If it runs long, cut duplication under Rule 3, or move a decision into its own record and leave a link — never compress `State` or `Environment`.

## Arguments

Anything the user passes describes what the **next** session will focus on. Let it shape emphasis: order `Next steps` around it, and expand the parts of `State` and `Environment` it depends on. It never becomes a reason to omit a section.
