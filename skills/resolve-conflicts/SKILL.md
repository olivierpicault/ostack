---
name: resolve-conflicts
description: >
  Resolve Git conflicts after a merge or a rebase. Use this skill when the
  user says "résous les conflits", "il y a des conflits", "fix the
  conflicts", "resolve conflicts", after a `git merge`, `git rebase`,
  `git cherry-pick` or `git stash pop` that failed on conflicts.
---

# Resolve Git conflicts

There are conflicts on the branch after a merge, a rebase or a cherry-pick.
Resolve them **keeping the intent of both sides**, not by blindly picking one
side.

## 1. Understand the state

```bash
git status
git diff --name-only --diff-filter=U
```

Identify:
- **The operation in progress**: merge, rebase, cherry-pick or stash.
- **The conflicted files**, and their kind from `git status`:
  - `both modified`: text conflict, with markers in the file.
  - `deleted by us` / `deleted by them`, `both added`, `added by us` /
    `added by them`: conflict **without markers**. See step 3.

⚠️ **`ours` / `theirs` are swapped during a rebase.** During a rebase, `ours` =
the branch you are rebasing onto, `theirs` = your own commit being replayed.
It is the opposite of a merge. Do not trust the names, trust the **content**.

## 2. Write down the intent of each side

Before editing anything, write one sentence per side saying what it wanted to
do. Get it from the commits:

```bash
git show MERGE_HEAD         # merge: the incoming branch tip
git show REBASE_HEAD        # rebase: the commit being replayed
git show CHERRY_PICK_HEAD   # cherry-pick: the commit being applied
git log --merge --oneline -- <file>   # commits touching this file on both sides
```

If the intent of one side cannot be found, ask the user.

## 3. Resolve each file

For each conflicted file:

1. **See the base version, not only the two sides.** The base is the common
   ancestor: it shows what each side actually changed.
   ```bash
   git show :1:<file>   # base
   git show :2:<file>   # ours
   git show :3:<file>   # theirs
   ```
   Before any edit, `git checkout --conflict=diff3 -- <file>` shows the markers
   with the base in a `|||||||` block. It overwrites the file: never run it
   after you started editing.
2. **Read the whole file**, not only the conflict blocks. The surrounding code
   often decides the right resolution.
3. **Classify each conflict block**, one by one:
   - **Independent changes** (two added functions, an import + another change,
     two list entries): keep both.
   - **Same question, two answers** (both sides changed the same value or the
     same logic differently): pick the answer that matches the intents from
     step 2. Do not mix the two. List it in the summary.
   - **One side makes the other useless** (code deleted or moved on one side,
     modified on the other): keep the side that remains. If the code was moved,
     apply the other side's change at its new place.

   A block that mixes several decisions: split it and classify each part.
4. **Conflicts without markers:**
   - `deleted by us` / `deleted by them`: one side deleted the file, the other
     modified it. Find out why it was deleted (moved? replaced?). Keep it with
     `git add <file>`, or delete it with `git rm <file>`. If it was moved,
     apply the modification to the new file.
   - `both added`: two files created with the same path. Merge their content
     like a text conflict.
5. **Special cases:**
   - imports / dependencies: often keep the union of both.
   - lockfiles (`package-lock.json`, `poetry.lock`, `uv.lock`, `yarn.lock`,
     `Cargo.lock`, `go.sum`): do NOT resolve by hand. Take one version, then
     regenerate (`npm install`, `poetry lock`, `uv lock`, etc.).
   - generated files / migrations: regenerate rather than editing the merge.
   - renumbering, versions, changelogs: check there is no duplicate after the
     merge.
6. **Touch only the conflict zones.** No reformatting, no renaming, no fix
   along the way.
7. Remove **all** `<<<<<<<`, `|||||||`, `=======`, `>>>>>>>` markers.

## 4. Mark as resolved

```bash
git add <file>
```

Check that nothing is left:

```bash
git diff --name-only --diff-filter=U
git grep -nE '^(<<<<<<<|\|\|\|\|\|\|\||=======|>>>>>>>)' || echo "clean"
```

## 5. Verify

A resolution that does not build is not a resolution. Before handing back:
- run the project's build / tests / linter if it is quick;
- re-read the `git diff --cached` of the result to confirm both intents are
  there.

## 6. Summary

Give one line per conflict block:

```
path/file.py:40-52 — independent changes — both kept — <why>
path/other.py:10 — same question, two answers — theirs kept — <why>
```

Put the "same question, two answers" lines first: they are choices the user
may want to change.

## Rules

- **Never run `--continue`**: it creates a commit. Stop after the summary and
  let the user run `git rebase|merge|cherry-pick --continue`. A rebase can stop
  again on new conflicts: the user runs this skill again.
- **Never `git push --force` without asking** (a rebase rewrites history).
- When the intent of one side is really unclear, **ask** rather than guess:
  do not silently delete code from the other side.
- If the user wants to give up: `git rebase --abort` / `git merge --abort`
  goes back to the state before.
