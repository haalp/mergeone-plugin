---
name: before-you-open-a-pr
description: Use before opening a pull request, or when asked to "open a PR", "push this for review" or "submit this change". Asks the mergeone gate whether the change duplicates a recent or open PR, so you link to it instead of opening another.
---

# Before you open a PR

Call `gate_similar` (MCP server `mergeone`) once, before `gh pr create` or any equivalent.

1. Collect: `repo` as `owner/name` (from `git remote get-url origin`), `title` and `body` you would use, `paths` as a comma-separated string of changed files (`git diff --name-only <base>...HEAD`), and optionally `author`. `k` defaults to 5.
2. Call `gate_similar` with those arguments. It returns `candidates`, each with `number`, `title`, `state`, `retrieval_score` and, when the model answered, `duplicate_of` (0 to 1).
3. Read the result:
   - A candidate with `duplicate_of` of 0.7 or more is probably the same change. Tell the user which PR (`#number`, title, state), show the score, and reference that PR instead of opening a new one: comment on it, or build on its branch.
   - Lower scores are pointers, not duplicates. Mention them in the new PR's description ("related: #N") and go on.
   - No candidates: open the PR.
4. If the tool answers `PR memory is not enabled in the policy (memory.enabled)`, say so and tell the user to set `memory: { enabled: true }` in `.mergeone.yaml` and run `mergeone index owner/name` (needs `GITHUB_TOKEN`). Do not guess at duplicates yourself.

Rules:
- Never close, merge or edit another PR because of a match. `gate_similar` reads; a human decides what to do with a duplicate.
- A score is a hint. Show it with the PR it points to, and let the user choose.
