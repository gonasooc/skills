---
name: commit
description: Commit the working tree in the convention the repository already uses — read from its declared config, otherwise inferred from recent history, never assumed. Handles Conventional Commits, Jira smart commits, ticket prefixes, and freeform styles; writes compact imperative subjects that summarize without dropping content; shows the exact message for approval before it commits; stages explicit paths only; runs the project's own verification when code changed; adds no attribution trailers; and stops at the commit unless push is asked for. Use when the user asks to commit changes, or runs /commit.
---

# Commit

You are committing the repository's way, not your way. Every repository already has a convention — declared in its config, or written into its history. Find it, apply it, and show your work before anything is written.

**Why this exists.** A commit skill that hardcodes one format is wrong in every repository that uses a different one, and one that guesses silently is worse. A mis-formatted subject line is cheap to fix; a Jira smart commit with the wrong key logs work time against someone else's issue, and an unwanted trailer is already in the history by the time it is noticed. So: infer the format, then put the inference in front of the user **as the commit message itself**. A wrong guess becomes visible at the approval gate, before it costs anything.

## Non-negotiables

1. **Never commit without explicit approval.** Show the exact message, then wait for the user to say go. Silence is not approval, and neither is a "네" given to some earlier question. This gate is what makes every inference in this skill safe to make — bypass it and nothing else here is load-bearing.
2. **Never assume a format.** Derive it, name the evidence, show it. When the evidence is thin or self-contradictory, ask — never fall back to a favorite.
3. **Never add a trailer the user did not write.** No `Co-Authored-By`, no "Generated with", no tool attribution, no emoji the convention does not call for. This holds even when the surrounding harness supplies such a trailer. Commit the approved text and nothing else — and read the message back afterwards to prove it (step 8).
4. **Never stage what was not shown.** No `git add -A`, `git add .`, or `git add -u`. Explicit paths only, listed first.
5. **Never write to the remote.** `/commit` ends at the commit. Push only when the user asks for it in that same request.
6. **Never commit over a failed check.** Verification fails → stop with the error, index untouched.
7. **Never invent a ticket key or a work duration.** Both have effects outside this repository.

## Flow

### 1. Read the tree before touching it

```bash
git rev-parse --is-inside-work-tree
git status --porcelain=v1 --branch
git log --oneline -1
```

Stop and report — do not improvise — on any of these:

- not a work tree, or nothing to commit
- HEAD is detached
- unresolved conflicts
- a merge, rebase, cherry-pick, or bisect in progress. Test for the state file, not the command's exit code: `test -f "$(git rev-parse --git-path MERGE_HEAD)"` — `--git-path` prints a path whether or not the file exists, so it succeeds either way.

### 2. Decide the change set — do not stage yet

**If the index already has staged changes, that staging is deliberate** (someone ran `git add -p`). Commit exactly it. Mention that unstaged changes remain, but never fold them in without being asked.

Otherwise select paths from `git status --porcelain` and hold them as a list. Read the diff — you cannot describe a change you have not read.

**Excluded on sight**, even when the user gestures at "everything": `.env` and `.env.*` (except `*.example`/`*.sample`), `*.pem`, `*.key`, `*.p12`, `*.pfx`, `id_rsa*`, `*.keystore`, `credentials*.json`, `service-account*.json`, `.npmrc`, `*.log`, `.DS_Store`, and anything under `node_modules/`, `dist/`, `build/`, `.next/`, `coverage/`. Name each exclusion out loud. Overriding one requires the user to name that file explicitly; a blanket "just commit it all" does not qualify.

Untracked files are listed separately before inclusion — a new file is as often a scratch file as a deliverable.

**If the change set spans unrelated concerns**, say so and offer to split it into separate commits. Offer; do not insist.

### 3. Verify — only what the project defines, only when code changed

Skip verification and say you skipped it when every path in the change set is documentation or metadata: `*.md`, `*.mdx`, `*.txt`, `*.rst`, `LICENSE*`, `.gitignore`, `docs/**`, images. Skip it too when a `pre-commit` hook or `lint-staged` config will run the same check at commit time — let the hook be the gate and surface its failure.

Otherwise detect the runner. **Never hardcode a command.**

For a JavaScript/TypeScript project, resolve the package manager in this order — the first hit wins:

| Signal | Manager |
| --- | --- |
| `packageManager` field in `package.json` | whatever it names (authoritative — corepack) |
| `pnpm-lock.yaml` | pnpm |
| `yarn.lock` | yarn |
| `bun.lock` / `bun.lockb` | bun |
| `package-lock.json` | npm |
| `package.json` with no lockfile | npm |

Two or more lockfiles and no `packageManager` field: do not guess — report which ones you found and ask. Invoke as `<manager> run <script>`, which is valid for all four.

Then read `scripts` in `package.json` and run **a script that actually exists** — prefer `build`, else `typecheck`/`type-check`, else `lint`. Never invent a script name; if none of them exist, skip and say so. In a workspace repo (`workspaces` field, or `pnpm-workspace.yaml`), run in the nearest package directory above the changed files, or at the root when the change spans packages — and report which directory it ran in.

Other ecosystems, by marker file: `Cargo.toml` → `cargo check`; `go.mod` → `go build ./...`; `Makefile` with a `build` target → `make build`; `build.gradle`/`build.gradle.kts` → `./gradlew build`; `pom.xml` → `mvn -q compile`. For `pyproject.toml` the command is project-specific — read the config, and skip rather than guess.

Verification runs against the working tree, not the index. When the index is a subset of the working tree, say so: the check covered more than this commit will.

**A failure stops everything.** Show the error, leave the index alone, offer to fix.

### 4. Detect the convention

Three tiers. A higher tier wins outright.

**Tier 1 — declared.** A repository that states its format is not a repository to take a vote in. Check, in order:

- `CLAUDE.md`, `AGENTS.md`, `.claude/CLAUDE.md` — any commit-message instruction
- `commitlint.config.{js,cjs,mjs,ts}`, `.commitlintrc*`, or a `commitlint` key in `package.json` → Conventional Commits; read `rules.type-enum` for the allowed types
- `git config commit.template` and the file it points at — the template *is* the format
- `.czrc`, `.cz.json`, `cz.config.js` (commitizen)
- a commit-message section in `CONTRIBUTING.md`
- `.husky/commit-msg` or `.git/hooks/commit-msg` — shows what will be rejected

**If a declared config and the history disagree**, that is a signal, not a tie: say which says what and ask. A commitlint config added last week over two years of freeform history is a real situation with a real answer, and it is not yours to pick.

**Tier 2 — the history vote.** Sample subjects, excluding merges and noise:

```bash
git log --no-merges --format='%s' -30
```

Discard subjects starting with `Revert `, `fixup!`, or `squash!`. Then classify each:

- **Conventional Commits** — matches `^[a-z]+(\([^)]+\))?!?: ` with a recognizable type (`feat`, `fix`, `docs`, `style`, `refactor`, `perf`, `test`, `build`, `ci`, `chore`)
- **Jira smart commit** — an issue key (`[A-Z][A-Z0-9]*-[0-9]+`) **and** at least one directive: `#comment`, `#time`, `#close`, `#resolve`, `#done`, `#in-progress`
- **Ticket-prefixed** — an issue key, no directive: `VD-7800 모달 UI 개선`, `[VD-7800] fix modal`
- **Gitmoji** — a leading emoji or `:shortcode:`
- **Freeform** — none of the above

Decide in this order:

1. **The newest 5 usable subjects are unanimous** → that is the convention. Recency beats volume; this is how a repository that switched conventions gets read correctly.
2. Otherwise **one family holds ≥70% of the 30** → adopt it.
3. Otherwise **re-sample the author's own commits** — `git log --no-merges --author="$(git config user.email)" --format='%s' -30` — and apply the same 70% bar. If it passes, adopt it and say the basis was their own commits in a mixed repository.
4. Otherwise → Tier 3.

**Tier 3 — ask.** Also ask outright when there are fewer than 5 usable commits (a new or shallow repository). Show the 5 most recent subjects as evidence and offer the observed candidates. Do not make the user write a spec.

### 5. Copy the shape, not just the family

The family names the grammar; the sample carries the details. From the same 30 subjects, read off:

- **Language.** Korean subjects → write Korean. Match the majority; do not translate the repository into English.
- **Body usage.** `git log --no-merges -20 --format='%b'` — if bodies are rare, write a subject and stop. A repository of one-line commits does not want your three-paragraph rationale.
- **Scope usage** (Conventional). If most subjects carry `(scope)`, use one — and prefer a scope that already exists in the sample over a new coinage. If they don't, don't.
- **Type vocabulary** (Conventional). Use types the repository actually uses, or `rules.type-enum` when commitlint declares it.
- **Directive set and order** (smart commits). Does *every* sampled commit carry `#time`, or only some? If only some, `#time` is optional here — which matters in step 6. Preserve the observed order.
- **Subject length, capitalization, trailing period.** Match the median and the habit.

### 6. Fill the variables that cannot be inferred

**Issue key** — resolve in this order, and stop at the first hit:

1. The branch name: `git rev-parse --abbrev-ref HEAD`, extract `[A-Z][A-Z0-9]*-[0-9]+`
2. Commits unique to this branch: `git log --no-merges @{u}..HEAD --format='%s'`, or against the default branch when there is no upstream
3. Ask

**Do not read the key off a plain `git log -5`.** On a branch cut fresh from main, those five commits belong to other people's tickets, and a smart commit sends this commit — and its worklog — to one of them. Whatever the source, show the key and get it confirmed before it reaches the message.

**`#time`** — always ask; never estimate. A `#time` directive writes a worklog entry into a shared tracker, and a wrong one has to be deleted by hand in Jira. If the user declines or does not answer, **omit `#time` entirely** and say plainly that this commit will not log work. Committing without a worklog is recoverable; a bad worklog on the wrong ticket is a mess someone else finds.

### 7. Write the message — imperative, compact, complete

**Mood.** Read it off the sample like every other detail, and fall back to the common convention when the sample is mixed:

- **English → imperative.** `add rate limiter to the auth endpoint`. Not `added`, not `adds`, not `this commit adds`. The test: the subject completes the sentence *"Applying this commit will…"*.
- **Korean → 명사형 종결(개조식).** `인증 엔드포인트에 rate limiter 추가`. Not `추가했습니다`, `추가함`, or a full sentence. Same test, same reason.

A commit **names the change; it does not narrate having made it.** Never past tense, never a sentence about yourself, never an opener like `This commit` or `이 커밋은`.

**Length.** The sample's median (step 5) is the target — that measured number, not a remembered rule of thumb. 72 characters is the conventional wrap point and a sane soft ceiling, but a `type(scope): ` prefix spends 15–20 of them, so judge the descriptive part rather than the whole line. Korean carries more per character; hold it to one line that does not wrap in a terminal. Running well past the sample's median is the signal to cut — not a fixed count.

**Compress without dropping.** Concise is not vague. Three real changes become three noun phrases separated by commas — not one hand-wave, and not three clauses of prose:

- ✅ `빈값 fallback(-) 및 파이프 구분선 추가, 모달 UI 개선`
- ❌ dropped — `버그 수정` / `fix several issues`
- ❌ over-explained — `데이터가 빈 값으로 내려올 때 화면에 아무것도 표시되지 않던 문제를 해결하기 위해 fallback 문자를 추가하고 파이프 구분선을 넣었으며 모달 UI도 함께 개선함`

Cut in this order: filler openers, the *how*, the file-by-file restatement of the diff, adjectives. Keep: **what** changed, and **where** when where is not obvious from the change itself.

**If it still will not fit**, that is step 2's split signal arriving late — no honest subject summarizes two unrelated changes. Offer the split instead of writing a longer line.

**Body** — only when the sample actually uses bodies (step 5) **and** there is a *why* the subject cannot carry: a non-obvious constraint, a rejected alternative, a bug's root cause. Never a body that restates the subject at greater length, and never a bulleted inventory of the diff — `git show` already prints that.

### 8. Show, commit, read back

Present, in one block, and then wait:

- the detected convention **and its evidence** — the config path, or "the last 12 of 15 commits"
- the files to be staged, and anything excluded
- the verification result, or why it was skipped
- **the exact commit message, verbatim** — in a fenced block, so it reads exactly as it will be recorded

Then stop and wait. The user has three moves; handle each without improvising:

- **Approve** — an explicit go-ahead. Only then does anything below this line happen.
- **Edit** — any correction to the message, the ticket key, the time, or the file list. Apply it, then **present the revised block and wait again.** A revision is never self-approving, however small it looks.
- **Cancel** — stop. Leave the working tree and the index exactly as found, and say plainly that nothing was committed.

If the reply is ambiguous, ask again. Never resolve an ambiguity in favour of committing.

On approval: stage the explicit paths (`git add -- <path> ...`), then commit by piping the message rather than inlining it, so that Korean text, quotes, and `#` survive intact:

```bash
git commit -F - <<'MSG'
<approved message>
MSG
```

**Never pass `--cleanup=strip`, and never let an editor open.** Git's default cleanup for `-F`/`-m` is whitespace-only, so a `#` survives wherever it sits — including at the start of a line. Under `strip`, that line is deleted outright and a smart commit loses its directive without a word.

Then **verify what was actually recorded**:

```bash
git log -1 --format=%B
```

Compare it to the approved text. A `prepare-commit-msg` hook, a commit template, or an injected trailer will show up here — and `Co-Authored-By` is exactly the thing this check exists to catch. Any difference: report it, and offer `git commit --amend -F -` with the approved text. Do not amend silently.

Report the short SHA and the subject.

### 9. Push — only when asked

`/commit` stops at step 8. Push when the user asks in that request ("`/commit push`", "커밋하고 푸시"), and then:

- `git push -u origin <branch>` for a branch with no upstream; plain `git push` when it has one
- **Never `--force`.** `--force-with-lease` only when the user asks for it by name and says why
- If the current branch is the default branch, confirm before pushing — say that out loud rather than assuming the answer
- Report the result, including the remote branch URL when the host prints one

## When something is off

Say it and stop, rather than producing a plausible commit:

- **Convention genuinely unreadable** — new repository, shallow clone, three commits of `wip`. Ask; do not default to Conventional Commits because it is popular.
- **The diff does not match what the user said they changed.** Trust the diff, raise the discrepancy.
- **A generated or vendored file dominates the diff** — lockfiles, build output, minified bundles. Confirm it belongs in this commit.
- **The change is large and incoherent.** Offer the split. One commit that says "여러 수정" is worse than three that say what they did.
