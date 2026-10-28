---
name: release-check
description: >
  Check what is about to be merged from one branch into another and list the
  risks before shipping: functional changes, migrations, contract changes,
  config, dependencies, deletions. Use for "release check", "diff release vs
  main", "risque de mettre en prod", "qu'est-ce qu'on s'apprête à merger".
argument-hint: "<source-branch> <target-branch>"
---

# Release check

You coordinate the check. You do not analyze the code yourself, and you do
not modify anything.

## 1. Find the branches

- Arguments given (`$ARGUMENTS`): first is the source, second is the target.
- Otherwise, use the branch flow from the user's instructions or the
  `CLAUDE.md` files.
- Still unclear → ask one short question.

Update the two branches, then check there is something to merge:

```bash
git fetch origin <source> <target>
git log --oneline origin/<target>..origin/<source>
```

Nothing listed → say so and stop.

## 2. Launch the analysis

Launch one `ostack:release-risk` agent with exactly this prompt, nothing else.
No summary, no opinion, nothing from this conversation.

```
Repository: <absolute path of the repo>
Source: origin/<source>
Target: origin/<target>
```

## 3. Report

Give the agent's report as is, in the user's language. Do not add, remove or
soften any risk.
