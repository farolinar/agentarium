---
name: code-engineering
description: Disciplined workflow for any coding task inside an existing codebase: building, reviewing, refactoring, or fixing code. Use this whenever the user asks to build or implement a feature, write code, add an endpoint, fix a bug, modify a module, review a pull request or diff, check code quality, refactor or restructure existing code, or asks "how should I build X" in a repo. It makes Claude read the repo's conventions first, pick the dominant pattern, check the design for loopholes and error paths, understand the relevant code before touching it, avoid rewriting old code, and ask before making decisions. Trigger even when the user does not say "engineering" if the task touches source code.
---

# Code Engineering

Follow these seven steps in order for any coding task. Do not skip steps because the change looks small. A one-line fix still needs steps 1, 5, and 6.

## Identify the use case first

Before starting, decide which use case the request is. Each one changes how the seven steps apply.

- **Build**: the user wants new code, a feature, an endpoint, a component, or a fix that adds behavior. All seven steps apply in full, and the output is code changes.
- **Code review**: the user wants feedback on existing code, a diff, or a pull request. Do not edit files. Steps 1, 3, 4, and 5 drive the review. Step 2 is used to judge whether the code follows the repo's dominant pattern. Step 6 is a reviewer's rule: flag unrelated messiness as optional, not as required changes. Step 7 applies to questions for the author, not to edits.
- **Refactor**: the user asks to restructure, rename, simplify, or move existing code without changing behavior. Step 6 is the goal here, not a limit: the refactor is the requested change, so it is in scope. Keep behavior identical, and make sure tests cover the code before and after. Step 7 applies to any structural decision (new module boundaries, public API changes, moving files across packages).

If the request could fit more than one use case (for example, "clean up this module"), ask which one the user means before starting.

## 1. Review the repo's conventions

Before writing anything, learn how this codebase is written. Do not rely on general best practices alone.

- Look at the project root for config and tooling: `README`, `CONTRIBUTING`, lint and formatter configs (`.eslintrc`, `ruff.toml`, `.editorconfig`, `prettier`, `golangci`), `pyproject.toml`, `package.json` scripts, `Makefile`, CI files, and any `docs/` folder.
- Check the language conventions in use: naming (camelCase, snake_case), file layout, error-handling style, logging, import order, test layout, and comment style.
- Sample at least 3 to 5 files that are similar to the target (same layer, same feature type, same kind of component). Note what they share.
- Count how often each pattern appears for the same kind of case. For example, if 8 of 10 service functions return errors as values and 2 throw exceptions, the dominant pattern is "return errors as values."

If the codebase uses patterns inconsistently, go to step 2. If it is consistent, follow it.

## 2. Evaluate the best pattern strategy

Choose the pattern to apply using this order:

1. **Dominant pattern wins** when one pattern clearly covers most similar cases (roughly 2/3 or more) and is not marked deprecated.
2. **Best pattern wins** when usage is split. Prefer the pattern that is:
   - already used by the newest code (check git log or file dates),
   - used by the module closest to the change,
   - documented in the repo's docs or lint rules,
   - safer (explicit error handling, typed interfaces, no hidden side effects).
3. **Ask the user** when the split is close and neither option is clearly better. Show the two options with a one-line tradeoff each and wait.

State which pattern you chose and why in one or two sentences before writing code.

## 3. Find loopholes in the design or request

Before implementing, check the request and the proposed approach for gaps:

- Ambiguous requirements: What counts as success? What input is allowed? What happens with empty, null, huge, or duplicate input?
- Hidden assumptions: Does the design assume a single user, a single process, a particular order of calls, or a network that never fails?
- Conflicts: Does the request contradict existing behavior, an API contract, a database constraint, or another part of the system?
- Scope creep: Is the request asking for more than it says (for example, "add a field" that also implies a migration, backfill, and UI change)?
- Security and data: Does it expose data, skip authorization, trust user input, or log secrets?

Write down each loophole you find in a short list. Do not silently fix them. Raise them with the user (step 7).

## 4. Find potential errors in the flow

Trace the execution path end to end, not just the function you are editing.

- Walk through the happy path, then each failure point: network call, database write, file read, parsing, external API, permission check.
- Check partial failure: if step 2 of 3 fails, is state left inconsistent? Is there a rollback, retry, or idempotency guarantee?
- Check concurrency: shared state, race conditions, double submission, caching.
- Check boundaries: off-by-one, timezone, encoding, integer overflow, empty collections.
- Check callers: who calls this code, and does your change break their expectations about return values, exceptions, or timing?

Report the risky points you found and how you will handle each one. If handling one requires a decision, ask (step 7).

## 5. Understand the code first

Read before you write. Do not start editing from the file name alone.

- Read the target function or module in full, plus its direct callers and direct dependencies.
- Read the existing tests for that code. They document intended behavior.
- Read the git history of the lines you will touch when the reason for a strange line is unclear (`git log -p -- <file>` or `git blame`).
- Trace the data: where it comes from, how it is transformed, where it ends up.
- If you cannot explain what the code does in two or three sentences, keep reading. Do not guess.

Limit your reading to the code related to the request. Do not map the whole repo unless the task requires it.

## 6. Do not refactor old code

Change only what the task requires. This rule protects code outside the request. When the request is itself a refactor, the refactor scope is the requested scope.

- Do not rename, reformat, reorganize, or "clean up" existing code that is unrelated to the request, even if it violates the conventions you found in step 1.
- Do not upgrade dependencies, change lint rules, or move files as a side effect.
- Keep the diff minimal and local. New code should follow the conventions from step 1. Existing code keeps its current style unless the user asks to change it.
- If you see something worth refactoring, mention it in one sentence at the end and leave it alone.
- The only exception is a refactor that is strictly necessary to make the requested change work. In that case, state why, and ask first (step 7).

## 7. Ask before making decisions

Stop and ask the user before you commit to any of the following:

- A choice between two reasonable designs or patterns (from steps 2 and 3).
- A change in public behavior, API shape, database schema, or config.
- Anything that deletes data, drops tables, force-pushes, or changes production settings.
- Adding a new dependency or library.
- Any refactor that step 6 would otherwise forbid.
- An unclear requirement that changes the implementation (from step 3).

When you ask:

- Ask one focused question at a time when possible, or a short numbered list when several are needed.
- For each decision, give 2 to 3 options with a one-line tradeoff and your recommendation.
- Do not bury the question in the middle of a long response.

Proceed without asking only for routine, reversible choices that clearly follow the conventions found in step 1, such as naming a local variable or placing a new function next to similar ones. Say what you assumed in one line.

## Output format for code tasks

Keep the response in this order, skipping sections that do not apply:

1. **Conventions found**: the pattern you will follow and where you saw it (one or two sentences).
2. **Questions or loopholes**: anything that needs a decision. Put this before the code if a decision blocks the work.
3. **Changes**: the code, limited to the scope of the request.
4. **Risks and error paths**: what could still go wrong and how it is handled.
5. **Out of scope**: refactors or issues noticed but not touched.

Write in plain prose. Use lists only for the loophole list, the questions, and the file list. Do not use headers in short answers.

For a **code review**, replace the format above with findings ordered by severity: bugs and security issues first, then error-handling gaps, then convention mismatches, then optional style notes. Each finding names the file and line, says what is wrong, and gives a suggested fix. Do not edit files.

For a **refactor**, state the behavior that must stay the same, list the files that change, and confirm which tests cover the affected code. If coverage is missing, say so and ask whether to add tests first.

## Quick checklist

- [ ] Read the repo's configs and sampled similar files
- [ ] Chose a pattern by dominance or best fit, or asked
- [ ] Listed loopholes in the request or design
- [ ] Traced happy path and failure paths
- [ ] Read the target code, its callers, and its tests
- [ ] Kept the diff to the requested scope
- [ ] Asked before any decision in the "ask first" list
