---
name: ask-the-gate
description: "Use before merging a branch or pull request, or when asked to merge this, land this, or is this safe to merge. Asks the mergeone gate and acts on its verdict (pass, hold or block). The gate never merges or closes anything."
---

# Ask the gate

Ask the gate before any merge. Do not merge first and ask later.

## How to ask

Two ways, in this order:

1. **MCP tool, if the repository has it.** When a `gate_decide` tool is available (the repository's `.mcp.json`
   starts `mergeone mcp`; `mergeone init --mcp --write` writes it), call `gate_decide` with `repo` (`owner/name`),
   `head_sha` (`git rev-parse HEAD`), `repo_path` `"."`, `base` (default `main`), `head` (default `HEAD`) and `pr`
   when there is one.
2. **Otherwise the CLI.** Run `mergeone pr`:
   - for an open pull request: `mergeone pr <owner/name> <N> --policy .mergeone.yaml` (reads the PR from GitHub when
     `GITHUB_TOKEN` is set, else renders the local diff);
   - for a branch with no PR yet: `mergeone pr --base <base> --head HEAD --policy .mergeone.yaml`.
   The command prints JSON and exits 0 / 1 / 2 for pass / hold / block. If `mergeone` is missing or there is no
   `.mergeone.yaml`, follow the `setup-mergeone` skill first.

Both return `verdict`, `route`, `rule`, `reason`, `probs`, `model_version`, `head_sha` and, on failure, `error`.
Always show the user the **head SHA** and the **rule** with the verdict.

## What to do with the verdict

- `pass`: merging is allowed. Merge only if the user asked you to merge; `pass` is permission, not an instruction.
- `hold`: do not merge. Open a PR if none exists and request the named `route`: `human` means ask a maintainer to
  review; `dedup` means link the PR it duplicates (see `before-you-open-a-pr`) and let the author or maintainer choose.
- `block`: stop. Report the `rule` and `reason` and change nothing. A person has to override it.
- `error` set: the gate failed closed (`hold` / `human`). The gateway already retried transient failures; an error
  starting with `CircuitOpen` means the model endpoint is down and the breaker is open — say so, name the
  `retrying at` time in the error, and do not retry in a loop.
- A new push changes the head SHA. Ask again after every push.

Rules:
- The gate never merges and never closes a PR. A `hold` is not a reason to close anything.
- Do not work around a `hold` or `block` by merging by another route.
- With the MCP server: `gate_verdict` (`repo`, `head_sha`) returns the stored decision for a SHA without a model
  call — use it to re-read, not to re-decide; `gate_plan` lists landing waves for the open PRs the gateway has seen,
  for "in what order should these merge".
