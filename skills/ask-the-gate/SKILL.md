---
name: ask-the-gate
description: Use before merging a branch or pull request, or when asked to "merge this", "land this" or "is this safe to merge". Calls the mergeone gate (gate_decide) and acts on its verdict: pass, hold or block. The gate never merges or closes anything.
---

# Ask the gate

Call `gate_decide` (MCP server `mergeone`) before any merge. Do not merge first and ask later.

1. Arguments. `repo` is `owner/name` and `head_sha` is the commit being merged (`git rev-parse HEAD`); both are required. Then `repo_path` `"."`, `base` (default `main`) and `head` (default `HEAD`). Add `pr` with the PR number if there is one; it enables the convergence cap.
2. The result has `verdict`, `route`, `rule`, `reason`, `probs`, `model_version`, `error` and `advice`. Always show the user the **head SHA** and the **rule** with the verdict.
3. Act on `verdict`:
   - `pass`: merging is allowed. Merge only if the user asked you to merge; `pass` is permission, not an instruction.
   - `hold`: do not merge. Open a PR if none exists and request the named `route`: `human` means ask a maintainer to review; `dedup` means link the PR it duplicates (run `gate_similar` to find it) and let the author or maintainer choose.
   - `block`: stop. Report the `rule` and `reason` and change nothing. A person has to override it.
4. If `error` is set, the gate failed closed (`hold` / `human`). The gateway already retried transient failures; an
   error starting with `CircuitOpen` means the model endpoint is down and the breaker is open — say so, name the
   `retrying at` time in the error, and do not retry in a loop.
5. A new push changes the head SHA. Ask again after every push.

Rules:
- The gate never merges and never closes a PR. A `hold` is not a reason to close anything.
- Do not work around a `hold` or `block` by merging by another route.
- `gate_verdict` (`repo`, `head_sha`) returns the stored decision for a SHA without a model call; use it to re-read, not to re-decide.
- `gate_plan` (`repo`, optional `semantic`) lists landing waves for the open PRs the gateway has seen; use it when the user asks in what order to merge several PRs.
