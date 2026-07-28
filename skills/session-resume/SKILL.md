---
name: session-resume
description: Pick up work from a checkpoint written into ./sessions/ by the session-checkpoint skill, possibly on a different machine or by a different agent. Reads the checkpoint plus the standing decision records under ./sessions/decisions/, checks every claim against the current repository and environment, reports where the document and reality have diverged, and stops for confirmation before doing any work. Explicitly invoked only; pass a date, slug, or branch to select a specific checkpoint.
argument-hint: "date, slug, or branch to resume (default: latest)"
disable-model-invocation: true
allowed-tools:
  - Bash(git status:*)
  - Bash(git log:*)
  - Bash(git branch:*)
  - Bash(git rev-parse:*)
  - Bash(git cat-file:*)
  - Bash(git merge-base:*)
  - Bash(ls:*)
---

# Session resume

Load a checkpoint from `./sessions/` and re-establish the context needed to continue the work — on a machine that was not there when it was written.

**Why this exists.** A checkpoint is a snapshot of one machine at one moment. By the time it is read, the repository has usually moved: commits landed, a branch was renamed, work was left uncommitted on the machine that wrote it. A summary read at face value is how a session confidently rebuilds something that already exists, or edits a file against a state that no longer holds.

So this skill does not simply read the document. **It treats the checkpoint as a claim and the repository as the truth**, reports every place the two disagree, and hands the decision back to the user before touching anything.

The counterpart is the `session-checkpoint` skill, which writes these files. Its document format — the fixed `##` headings — is the contract this skill reads.

There are two kinds of file to load, and they are not equal. A **checkpoint** describes one moment and expires. A **decision record** in `sessions/decisions/` states a constraint that holds until something supersedes it. The checkpoint tells you where the work stopped; the decision records tell you what you are not free to change.

## Rules

1. **The repository outranks the document.** Wherever they conflict, reality wins and the conflict gets reported. Never silently reconcile a difference in your head.
2. **Verify before proposing.** Read the files named in `References` and `Next steps` before saying what to do next. The checkpoint says what was true then; only the files say what is true now.
3. **Carry the verified/assumed split forward.** Anything the checkpoint marked *assumed* is still unverified — do not promote it to fact just because it is written down. If a next step depends on an assumed item, say so.
4. **Standing decisions bind; they are not suggestions.** A decision record that nothing supersedes constrains the work whether or not the checkpoint mentions it. If a next step would violate one, say so before proposing it — do not quietly follow the newer checkpoint over the older constraint.
5. **Do no work.** Loading context is the whole job. Report, then stop and wait. Resuming a stale checkpoint automatically is the exact failure this skill exists to prevent — and it applies even when the next step looks trivial and safe.
6. **Read only.** No edits, no branch switches, no `git checkout`, no `stash`, no fetch. Recommend them; let the user run them. This includes the decision records: superseding one is the `session-checkpoint` skill's job, never this one's.

## Flow

### 1. Find the checkpoint

Look for `sessions/` at the repository root (`git rev-parse --show-toplevel`). If it is missing or empty, say so and stop — do not go hunting elsewhere or reconstruct context from git history instead.

- **No argument** — take the newest file by filename. Name which one you picked and how many others are there, so a wrong pick is obvious immediately.
- **With an argument** — match it against filenames, the `branch` frontmatter field, and titles. On several matches, list them and ask. On none, say so and offer the newest.

If the chosen checkpoint has a `continues:` field and its `Goal` alone does not explain the work, follow the chain back — but at most two hops, and read only `Goal` and `Decisions` from the older files.

### 2. Work out which decisions still stand

List `sessions/decisions/`. If it is absent, skip this step silently — a project may simply have recorded none.

Read the frontmatter of every file there and build the superseded set: any filename appearing in another file's `supersedes:` is retired. What remains is **standing**. Records are append-only, so a retired file still reads as if current — its own text will never say it was replaced. Missing this is how a session follows a constraint that was overturned months ago.

Then read in full:

- every standing record the checkpoint's `Decisions` section links to;
- every standing record whose subject touches the checkpoint's `Next steps`;
- all of them, if there are only a handful.

For the rest, the title is enough. Say how many you listed versus read in full, so an unread constraint is never mistaken for an absent one.

### 3. Check the document against reality

This is the core of the skill. Run the checks, then report each divergence explicitly.

**Git state** — compare the frontmatter to now:

```bash
git rev-parse --short HEAD                 # vs. `commit:`
git rev-parse --abbrev-ref HEAD            # vs. `branch:`
git status --porcelain                     # this machine's tree
git cat-file -e <commit>^{commit}          # does the recorded commit exist here?
git log --oneline <commit>..HEAD           # what landed since
```

Report whichever applies:

- **Commit not found locally** — it was never pushed, or this machine has not fetched. The checkpoint describes work that is not here. Say this first; nothing else in the document can be trusted until it resolves.
- **HEAD has moved past it** — list what landed since. Parts of `Next steps` may already be done.
- **Different branch** — name both and recommend the checkout; do not run it.
- **`tree: dirty` in the checkpoint** — uncommitted work existed on the writing machine and is **not here**. Say so prominently. This is the most common way a cross-machine handoff loses work, and the document cannot tell you what was in it.
- **This machine's tree is dirty** — local changes exist that the checkpoint knows nothing about. Flag before recommending anything that touches those files.

**Environment** — walk the `Environment` section and check what is checkable on this machine: are the named environment variables set (presence only, never print values), do the required local files exist, are the tool versions comparable, is the named service reachable. List what is missing as setup the user must do first.

### 4. Report

Keep it short and in this order:

1. **Which checkpoint** — filename, when it was written, from which machine and agent.
2. **Goal** — one or two sentences, from the document.
3. **Divergences** — every mismatch from step 3. If there are none, say that explicitly; silence reads as "not checked."
4. **State** — what is verified, what is only assumed, with the split intact.
5. **Standing decisions** — the constraints from step 2 that bear on what happens next, one line each with a path. Note how many were listed versus read in full. If a `Next step` would violate one, say so here rather than in the next item.
6. **First action** — the first item from `Next steps`, adjusted for what you found, and named as a proposal.

Then stop. Do not begin.

### 5. After the user confirms

Proceed with the work normally, within the standing decisions. The `Suggested skills` section is a recommendation, not an instruction — invoke those skills when they genuinely fit the task in front of you.

If the work ends up overturning a standing decision, that is legitimate — but it must be recorded, not just done. Say so at the time, and leave it to `session-checkpoint` to write the superseding record.

When that work reaches a natural stopping point, the `session-checkpoint` skill is what writes the next document. Do not invoke it on your own; suggest it.

## Language

Mirror the user's language. The `##` headings inside checkpoint files stay English by design — quoting them verbatim in a Korean report is correct, not a mismatch.
