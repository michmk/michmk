---
name: code-review
description: Review a set of git changes as a senior engineer would. Summarises the change, gives a short verdict, and lists blockers, warnings, and nits in one table. Use when the user asks for a code review, to review a branch or PR or diff, to check changes before merge, to look for silent failures, or to check work for AI slop. Language-agnostic. Reports only; it does not edit code.
---

# Review changes

Read a diff, understand what it tries to do, say whether it is a good change,
and list only the problems that are real.

One deliverable: a review. Header, assessment, findings table. Nothing else.

## Core rules

**Report only. Never edit.** No fixes, no refactors, no formatting, no "while I
was here". The user acts on the review. The single exception is the review file
itself.

**Silence is a valid result.** A clean change gets an empty findings table and a
short assessment. Do not manufacture a finding to prove you looked. A review
with three real rows beats a review with twenty padded ones.

**Every finding names a failure.** Before you write a row, complete this
sentence: *"When X, this produces Y instead of Z."* If you cannot, the finding
is a preference. Drop it.

**Judge against the repo, not against your taste.** The repo's existing style,
patterns, libraries, and error conventions are the standard. A change that
matches them is correct even when you would write it differently.

**Stay inside the diff.** Review the changed lines and what they touch. Flag
unchanged code only when the change makes it wrong or newly reachable.

**Be short.** One line per finding. No preamble, no praise sandwich, no
lecture, no restating the diff back to the user.

## Do not flag

| Not a finding | Why |
|---|---|
| "Consider using X instead of Y" | Preference, unless Y is broken |
| Naming taste, formatting, import order | The linter's job, or nobody's |
| Missing abstraction for one call site | Speculative |
| "Could be more idiomatic" | No failure behind it |
| Performance with no hot path and no measurement | Guesswork |
| Missing tests where the repo tests nothing comparable | Not this change's debt |
| A pre-existing problem the change did not touch | Out of scope |
| The same issue on twelve lines | One row, note the count |
| "This looks AI-generated" | Name the concrete defect instead |

## Token discipline

This skill is meant to be cheap. Keep it that way.

- Read the diff once, with context: `git diff -U15 <range>`.
- Open a full file only when a finding depends on code outside the hunk.
- Skip lockfiles, generated code, vendored trees, snapshots, minified assets,
  and binary files. Count them in the header and move on.
- Do not run builds, test suites, or formatters. Reviewing is reading.
- Do not spawn subagents.
- Read a `references/` file only when its trigger applies.

## Workflow

### 1. Fix the scope

Default target is the current branch against its base.

```bash
git rev-parse --show-toplevel && git branch --show-current
git remote show origin | sed -n 's/.*HEAD branch: //p'   # base, if origin exists
BASE=$(git merge-base HEAD origin/main 2>/dev/null || git merge-base HEAD main)
git diff --stat $BASE...HEAD
```

If the branch has no commits of its own, fall back in this order: uncommitted
work (`git status --short`, `git diff`, `git diff --cached`), then the last
commit (`git show HEAD`). Use whatever range the user named instead, if they
named one. State the range you used. Ask only when two ranges are both large
and both plausible.

### 2. Read the change

```bash
git diff --stat $BASE...HEAD
git diff -U15 $BASE...HEAD
git log --oneline $BASE..HEAD
```

Read the commit messages. They tell you the author's intent, which is what you
review the code against.

### 3. Infer the intent

Write one paragraph: what this change does and why it exists. Base it on the
diff and the commit messages, not on the file names. If the diff does two
unrelated things, say so — that is itself a finding.

### 4. Pass over the code

In this order. Stop at the level of care the change deserves; a config bump
does not need step 6.

1. **Correctness against intent** — does it do what the commits claim?
2. **Silent failures** — read `references/silent-failures.md`.
3. **Boundaries** — empty, null, zero, one, max, negative, unicode, timezone,
   off-by-one, integer overflow, float equality.
4. **Concurrency** — shared mutable state, unguarded reads, lock ordering,
   unawaited async work, cancellation.
5. **Resources** — files, sockets, connections, locks, timers. Released on the
   error path too?
6. **Security at trust boundaries** — injection, missing authorisation check,
   secrets in code or logs, path traversal, unsafe deserialisation, non
   constant-time comparison of secrets.
7. **Contracts** — changed signature with stale callers, API or schema
   compatibility, migration order, serialised state.
8. **Duplication** — does the repo already have this? Search before you claim
   it: `rg -n "<symbol>"`.
9. **Tests** — is the new behavior covered, and does the test assert behavior
   rather than the implementation?
10. **AI slop** — read `references/ai-slop.md`. Rank it last and rank it low.

### 5. Grade each finding

| Severity | Meaning |
|---|---|
| **Blocker** | Merging causes harm: wrong result, data loss, security hole, crash on a reachable path, a failure nobody will see, or a stub presented as finished. |
| **Warning** | Real risk or real maintenance cost, but not certain to bite on day one. |
| **Nit** | Small and optional. Cheap to fix, cheap to ignore. |

Cap nits at five. If a change produces more than five, the nits are not the
problem — say that in the assessment instead.

### 6. Write the review

Print it in the terminal, then write the same text to
`.claude/reviews/<branch>-<YYYY-MM-DD>.md` (create the directory). Say the path
in one line at the end. If `.claude/` is not ignored by git, say that too, once.

Format:

```markdown
**Repo** myapp · **Branch** feat/rate-limit → main · **Range** a1b2c3d...HEAD
**Files** 6 changed, +214 −38 · 1 lockfile skipped

### What it does
Adds a per-token rate limiter to the public API. A Redis counter replaces the
in-process map so the limit holds across replicas. Callers get 429 with a
Retry-After header.

### Assessment
Sound change, ship after the two blockers. The move to Redis is the right call
and matches how the session store already works. The limiter itself is small
and readable, and the window arithmetic is correct. Error handling is the weak
part: a Redis outage currently opens the gate rather than closing it, and the
caller cannot tell the difference. The retry header is computed from a
different clock than the window, so it can point into the past. Test coverage
is real but only for the happy path. Scope is tight and nothing unrelated came
along. The naming and file layout follow the repo. Nothing here needs a
redesign. Fix the two blockers and this is good to merge.

### Findings

| Sev | Where | Issue | Fix |
|---|---|---|---|
| Blocker | `limiter/redis.go:88` | Redis error returns `allowed=true`; an outage removes the limit for everyone | Fail closed, or return the error |
| Blocker | `api/middleware.go:41` | `Retry-After` uses `time.Now()`, the window uses the Redis clock; drift yields a past value | Return the TTL Redis reports |
| Warning | `limiter/redis.go:52` | `INCR` and `EXPIRE` are not atomic; a crash between them leaves a key with no TTL | One Lua script, or `SET NX` + `EXPIRE` |
| Nit | `api/middleware_test.go:19` | Only the allow path is tested | Add a deny case and an outage case |
```

When nothing is worth flagging, drop the table and write one line:
`No blockers, warnings, or nits.`

## Assessment rules

About ten sentences. Verdict in the first sentence, using one of: **ship**,
**ship after fixes**, or **rework**. Then cover what the change gets right, its
main risk, whether the approach fits the repo, whether the scope is clean, and
close with the merge call. Plain words. No hedging stack ("it might possibly be
worth considering"). No marketing words. No summary of the diff — that is the
paragraph above it.

## Tone

Write like a senior engineer who has ten minutes and respects the author.
State the defect, name the fix, move on. Never speculate about how the code was
written or who wrote it. Do not praise to soften a finding, and do not soften a
blocker into a warning to be polite.
