# Rule priority

- If two rules in this file conflict: **Clarity > Consistency > everything else.**

# Consistency

- **Always favor consistency.** Before writing something new, look at how the
  neighboring code does it and copy the pattern (style, technical choices,
  structure). E.g. existing clients use native async `httpx.AsyncClient` → a
  new client does too.
- **Exception: comments.** Never match the comment density of nearby code,
  even if you wrote it. The "Comments in code" section always wins.

# Scope and response style

- Do **strictly** what is asked: the question asked, not the problem you guess
  is behind it. Smallest correct change. Do not touch any file or line the
  request does not need: no reformatting, no renaming along the way.
- Do not widen the scope or pre-answer follow-up questions. If a related
  action seems useful, **suggest it in one line at the end** and wait for a yes.
- Real ambiguity that changes the result: ask ONE short question and wait.
  When in doubt between doing more or less: do less, and ask.
- Only point out something out of scope for a **real bug, data loss, or
  security flaw**: one sentence, at the end, without acting on it.
- No preamble, no summary of what you are about to do, no closing recap.
- One review of the diff is enough, unless I ask for more.

# Verify before stating

- Check every fact in the code, logs, docs or git history before stating it.
  Nothing from memory: branches, files and configs change.
- If you could not check something, say so plainly ("not checked").
- Never invent a name, a command, an option or a convention.

# Clarity / vocabulary

- **Everything must be SIMPLE: code, comments, explanations.** Non-negotiable.
- **Zero jargon.** No obscure terms (e.g. "FastAPI boundary", "dependent
  metamodel", "feature-gated"), no metaphors, no made-up terms. Forbidden
  words: load-bearing, seam, provenance, grain, surface, boundary, wire up,
  gate, invariant.
- Reuse the names that already exist in the codebase; do not invent new ones.
- If a technical term is truly needed, explain it in one simple sentence.

# Comments in code

- **By default: no comments.** The only exception is a non-obvious "why" a
  competent dev cannot infer from the code, about a lasting constraint
  (external API behavior, technical limit, worked-around bug). Not who calls
  the code, what it is used for today, or what we plan to do with it.
- One short line, **always in English**.
- **Never tell the history**: no "changed to…", "now uses…", "previously…",
  "as discussed", no reference to our conversation, a ticket or a plan.
- No comment that paraphrases the code (e.g. `coerce_numbers_to_str=True`,
  `extra="allow"`). No section banners (`# --- Helpers ---`), no commented-out
  code, no decorative comments above imports or constants.
- Docstring: one line, only if the name and signature are not enough. Never
  Args/Returns/Raises sections when the types already carry them.
- A **field description** describes what the field *is*, not who uses it
  (no "used by X", "obtained by the add-on").
- `# TODO` only if I asked to postpone something specific. If unsure, ask.
- **A comment is code**: accurate (no "X does Y" if Y does not exist yet),
  precise (the real "why"), consistent (if one case gets a comment, its twin
  case does too).
- **Before handing back: re-read your diff and delete every comment that is
  not a non-obvious "why"**, including in a subagent's output.

# Git

- **Never** `commit` or `push` unless I explicitly ask. Prepare the changes,
  let me commit/push.
- Branch names are **always** in English.

# Python style

- Prefer classic inheritance (`ABC` + `@abstractmethod`) over `typing.Protocol`.
- Prefer a `NamedTuple` over a raw `tuple` (e.g. `tuple[int, int]`) to return
  or pass several values.

# "Look" = analysis

- "Regarde" / "look" / "look into" = analyze only. Do not write code without
  an explicit go.

# Memory

- Keep per-project memory to a minimum. When I ask you to remember something,
  write it by default in the global memory (`~/.claude/CLAUDE.md`), not in
  the project memory.
