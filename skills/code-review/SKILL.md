---
name: code-review
description: >
  Full review of my changes before opening a MR/PR: bugs, security,
  architecture, performance, tests and my ostack rules, each read by a separate agent with no
  context from this conversation. Use for "code review", "review my changes",
  "review before MR", "relis mon code avant la MR".
---

# Code review before MR

You coordinate the review. You do not review the code yourself, and you do not
modify anything.

## 1. Find the diff

```bash
git status -sb
git log --oneline @{upstream}..HEAD
```

- Upstream known → `git diff @{upstream}...HEAD`.
- No upstream → find the branch this one will be merged into, from the user's
  instructions or the `CLAUDE.md` files, else `main`/`master`. Use
  `git diff $(git merge-base HEAD origin/<base>)...HEAD`.
- The user says "the last commit" → `git diff HEAD~1...HEAD`.
- `git status` shows uncommitted changes → ask whether to include them. If yes,
  use `git diff <base>` instead of `<base>...HEAD`.

If the base is unclear, ask one short question.

Write the diff to a file once, so the agents read the same diff without
running git again:

```bash
f=$(mktemp "${TMPDIR:-/tmp}/ostack-review.XXXXXX") && <diff command> > "$f" && echo "$f"
```

## 2. Launch the six reviewers

In a single message, launch these six agents in parallel:
`ostack:review-bugs`, `ostack:review-security`, `ostack:review-architecture`,
`ostack:review-performance`, `ostack:review-tests`, `ostack:review-ostack`.

Send each one exactly this prompt, nothing else. No summary of the work, no
opinion on the code, nothing from this conversation.

```
Repository: <absolute path of the repo>
Diff file: <path of the diff file from step 1>

Return at most 5 findings, most important first, in this format, or
"No findings.":

- severity: 🔴 | 🟠 | 🟡
  location: path/to/file:line
  problem: <what is wrong>
  impact: <what it breaks or costs>
  fix: <suggested fix>
```

For `ostack:review-ostack`, add one line after the diff file line:
`Rules file: ${CLAUDE_SKILL_DIR}/../../rules.md` (as an absolute path).

## 3. Verify

If there is at least one 🔴 or 🟠 finding, launch a single
`ostack:review-verify` agent with all of them, numbered, and only:

```
Repository: <absolute path of the repo>
Diff file: <path of the diff file>
Findings:
1. <the finding, as returned>
2. ...
```

Drop every finding it rejects. Move every finding it marks `unverifiable` to
the "Not checkable from the code" section, with its reason. 🟡 findings are
not verified.

## 4. Report

Merge findings that point to the same problem. Write the report in the user's
language:

```
## Verdict: ✅ ready / ⚠️ fix before MR / ❌ blocking

### 🔴 Blocking
1. `path/file.py:42` — <problem> → <impact> → <fix>
2. ...

### 🟠 To fix
3. ...

### 🟡 Nits
4. ...

### Not checkable from the code
5. `path/file.py:42` — <problem> → <why the code alone cannot prove it>

Rejected after verification: <N>
```

- Number findings 1, 2, 3… across the whole report: numbering does not
  restart in each section, so each number points to one finding.
- Show the 🟡 section only if no 🔴 and no 🟠 finding was confirmed.
  Otherwise drop the 🟡 findings.
- Nothing to report → say so plainly.
- Delete the diff file.
- End with: "Shall I fix the blocking issues?" (in the user's language).
