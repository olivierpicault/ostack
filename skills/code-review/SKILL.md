---
name: code-review
description: >
  Full review of my changes before opening a MR/PR: bugs, security,
  architecture, performance and my ostack rules, each read by a separate agent with no
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

## 2. Launch the five reviewers

In a single message, launch these five agents in parallel:
`ostack:review-bugs`, `ostack:review-security`, `ostack:review-architecture`,
`ostack:review-performance`, `ostack:review-ostack`.

Send each one exactly this prompt, nothing else. No summary of the work, no
opinion on the code, nothing from this conversation.

```
Repository: <absolute path of the repo>
Diff to review: `<diff command from step 1>`

Return each finding in this format, or "No findings.":

- severity: 🔴 | 🟠 | 🟡
  location: path/to/file:line
  problem: <what is wrong>
  impact: <what it breaks or costs>
  fix: <suggested fix>
  certainty: sure | to check
```

For `ostack:review-ostack`, add one line after the diff line:
`Rules file: ${CLAUDE_SKILL_DIR}/../../rules.md` (as an absolute path).

## 3. Verify

For each 🔴 and 🟠 finding, launch one `ostack:review-verify` agent, all in
parallel, with only:

```
Repository: <absolute path of the repo>
Diff to review: `<diff command>`
Finding:
<the finding, as returned>
```

Drop every finding it rejects. 🟡 findings are not verified.

## 4. Report

Merge findings that point to the same problem. Write the report in the user's
language:

```
## Verdict: ✅ ready / ⚠️ fix before MR / ❌ blocking

### 🔴 Blocking
- `path/file.py:42` — <problem> → <impact> → <fix>

### 🟠 To fix
- ...

### 🟡 Nits
- ...

Rejected after verification: <N>
```

- Keep "to check" on findings that were not sure.
- Nothing to report → say so plainly.
- End with: "Shall I fix the blocking issues?" (in the user's language).
