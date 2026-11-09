---
name: before-you-open-a-pr
description: "Use before opening a pull request, or when asked to open a PR, push this for review, or submit this change. Asks the mergeone gate whether the change duplicates a recent or open PR, so you link to it instead of opening another."
---

# Before you open a PR

Check for a duplicate once, before `gh pr create` or any equivalent.

The check needs the repository's PR memory, which the mergeone MCP server exposes as `gate_similar`. It is available
when the repository's `.mcp.json` starts `mergeone mcp` (`mergeone init --mcp --write` writes it) and the policy has
`memory: { enabled: true }` with an index built by `mergeone index owner/name` (needs `GITHUB_TOKEN`).

- **`gate_similar` is available:**
  1. Collect `repo` (`owner/name`, from `git remote get-url origin`), the `title` and `body` you would use, `paths`
     as a comma-separated list of changed files (`git diff --name-only <base>...HEAD`), optionally `author`. `k`
     defaults to 5.
  2. Call it. It returns `candidates`, each with `number`, `title`, `state`, `retrieval_score` and, when the model
     answered, `duplicate_of` (0 to 1).
  3. A candidate with `duplicate_of` of 0.7 or more is probably the same change: tell the user which PR (`#number`,
     title, state), show the score, and reference that PR instead of opening a new one — comment on it, or build on
     its branch. Lower scores are pointers: mention them in the new PR's description ("related: #N") and go on. No
     candidates: open the PR.
  4. If the tool answers `PR memory is not enabled in the policy (memory.enabled)`, say so and point to the setup
     above. Do not guess at duplicates yourself.
- **`gate_similar` is not available:** say in one line that the duplicate check is not configured here (and how to
  enable it), then open the PR. Do not search for duplicates by hand and present the result as the gate's.

Rules:
- Never close, merge or edit another PR because of a match. The check reads; a human decides what to do with a
  duplicate.
- A score is a hint. Show it with the PR it points to, and let the user choose.
