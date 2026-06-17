---
name: commit
description: Commit only the files changed during the current conversation session, on the current branch, with an auto-generated message. Fully automatic — asks nothing, stages nothing it did not touch this session. Use when the user runs /commit or asks to commit the work done in this session.
---

# /commit

Stage and commit **only the files this conversation session changed** — nothing
else — on the current branch, with a clear auto-generated message. Run the whole
procedure end to end without pausing. **Ask the user nothing.** Do not push.

This extends the repo's committing rule (`CLAUDE.md` → *Committing*): committing
is quick and automatic. The skill adds one hard constraint: a commit must contain
*only* what was changed in this session, never the repo's other pending changes.

## Why the session scope matters

The working tree usually carries pending changes that have **nothing to do with
this session** (work in progress, other tools, prior edits). A blanket
`git add -A` / `git add .` / `git commit -a` would sweep those into the commit.
This skill never does that. It commits an explicit, verified list of paths.

## Step 1 — Build the session file set (authoritative)

Enumerate every path you created, modified, renamed, or deleted **in this
conversation** via your own tool calls — `Write`, `Edit`, `NotebookEdit`, and any
file-mutating `Bash` command (`mv`, `rm`, `>`/`>>` redirects, `sed -i`, `cp`,
`mkdir` + write, `git mv`, etc.). Read back through the transcript and list them.

- This list — not `git status` — is the source of truth for *what to commit*.
- For a rename, include **both** the old and new path.
- Exclude paths you only read, searched, or inspected.
- If the set is empty (you changed no files this session), **stop**: report
  "Nothing from this session to commit" and do not stage anything. Do **not**
  fall back to committing pre-existing changes.

## Step 2 — Intersect with the real git state

Run `git status --porcelain` (and `git branch --show-current`). Keep only the
session paths that actually show as changed/untracked on disk. Drop any session
path that is clean now (e.g. you edited it, then reverted it) — there is nothing
to commit for it. The result is the **final path list**.

If the final list is empty after intersecting, **stop** and report that there is
nothing to commit (your session edits net to no on-disk change).

## Step 3 — Stage exactly those paths

Stage with explicit pathspecs so only the final list is touched:

```
git add -A -- <path1> <path2> ...
```

`-A` with an explicit pathspec stages adds, modifications, **and** deletions/
renames for *only* those paths, leaving every other pending change unstaged.

**Never** run, in this skill: `git add -A` (no pathspec), `git add .`,
`git add -u` (no pathspec), or `git commit -a`. Each would capture unrelated
changes.

## Step 4 — Verify the staged set

Run `git diff --cached --name-only` and confirm it equals the final list from
Step 2 — no more, no less.

- Extra paths staged → unstage them: `git restore --staged -- <path>`.
- Missing intended paths → re-add them, or drop them if they turned out clean.

Do not proceed to commit until the staged set matches.

## Step 5 — Commit on the current branch

Commit directly on the current branch (do not create or switch branches; do not
push). Generate the message yourself from the staged diff — no questions.

Message format (match this repo's existing history):

- Imperative subject line, ≤ ~72 chars, summarizing what changed.
- Optional body, wrapped ~72 chars, explaining the *why* when it adds signal.
- End with this footer line, after a blank line:

  ```
  Co-Authored-By: Claude Opus 4.8 (1M context) <noreply@anthropic.com>
  ```

Use a HEREDOC so the body and footer are preserved:

```
git commit -F - <<'EOF'
<subject line>

<optional wrapped body>

Co-Authored-By: Claude Opus 4.8 (1M context) <noreply@anthropic.com>
EOF
```

## Step 6 — Report

State the commit's short SHA and subject, and list the files committed. If any
session paths were intentionally left out (clean, or reverted), say so briefly.
Pre-existing changes remain uncommitted and untouched — that is expected; mention
it only if it could surprise the user.

## Guardrails

- **No prompts.** Never ask whether to commit, what message to use, or which
  files — decide and proceed.
- **Session-only.** If unsure whether a changed file belongs to this session,
  leave it out. Under-including is safe; over-including is not.
- **Current branch only.** Honor `CLAUDE.md` — commit on the current branch.
- **No push.** Push only if the user explicitly asks (outside this skill).
- **Nothing to do is a valid outcome.** Report it; do not invent a commit.
