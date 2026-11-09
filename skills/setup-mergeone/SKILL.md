---
name: setup-mergeone
description: "Use when mergeone is not installed or not configured: the mergeone command is missing, there is no .mergeone.yaml, the mergeone MCP server fails to start, or the user asks to set up, install or configure the merge gate."
---

# Set up mergeone

The skills and commands run the `mergeone` binary. It needs to be installed, and the repository needs a policy file.
The MCP server (`gate_decide`, `gate_similar`, …) is not part of this plugin: `mergeone init --mcp --write` adds it
to the repository's own `.mcp.json`, where the team can review it.

1. **Install the binary** (macOS / Linux). Tell the user the command and run it only after they agree:
   ```bash
   brew install haalp/tap/mergeone
   mergeone version
   ```
   Other routes: a release archive from https://github.com/haalp/mergeone/releases (checksums signed with Sigstore),
   `go install github.com/haalp/mergeone/gate/cmd/mergeone@latest`, or the image `ghcr.io/haalp/mergeone`.
2. **Write the policy** in the repository root: `mergeone init`. It prints what it detected, writes `.mergeone.yaml` with safe defaults (PR memory off, verdict actions off, fail closed) and never overwrites an existing file without `--force`.
   - `--mcp --write` writes the `mergeone mcp` server into the repository's `.mcp.json` and appends the "call
     gate_decide before merging" snippet to `CLAUDE.md`.
   - `--ci` writes `.github/workflows/mergeone.yml`; `--all` is `--ci --mcp --write`.
   - `--model <name>` picks the decision model (`mergeone models` lists them).
   - `--portal <token>` (optional) posts gatelog records to the hosted portal. The token comes from the portal; do not invent one.
3. **Keys** go in the environment or a `.env` that is in `.gitignore`, never in the repository or a committed file. Per model: `TYPESAFE_API_KEY` (`jev`, default), `PERPLEXITY_API_KEY` (`pplx`, `pplx-flash`), `ANTHROPIC_API_KEY` (`haiku`). `GITHUB_TOKEN` is needed only for `mergeone pr` on a remote PR, `mergeone index` and verdict actions. Never print a key.
4. **No key yet**: `mergeone mcp --mock` runs the pipeline with a stand-in model. Use it to check wiring only; its verdicts mean nothing.
5. **First week: shadow mode.** Suggest `mode: shadow` in `.mergeone.yaml` for a team's first days with the webhook
   gateway (`mergeone serve`): every PR is decided and recorded, nothing is written to GitHub, and `mergeone stats`
   shows what the gate would have done. Switch to `mode: enforce` when the thresholds look right.
6. **Check the policy without a model**: `mergeone policy check` prints the rules in order; `mergeone policy scaffold`
   then `mergeone policy test` run cases you can edit; `mergeone replay --gatelog .mergeone/gatelog.jsonl` shows which
   past verdicts a policy edit would change (exit 1 on a flip — fine for CI).

## What `.mergeone.yaml` holds

`mode` (`enforce` | `shadow`), `model`, `question_set` (`v2` default), `always_human_paths` (prefixes that always hold for a human), `memory` (PR memory, needed by `gate_similar`; set `enabled: true`, then run `mergeone index owner/name`), `rules` (first match wins, each with a verdict and a route), `default`, `on_error` (fail closed), `resilience` (retries and the circuit breaker around model calls), `drift`, `actions` (off by default; never merges; one verdict comment per PR, edited on each new head) and the optional `portal` block. Full reference: https://github.com/haalp/mergeone/blob/main/docs/POLICY.md.

## Self-hosted model

To use your own server that speaks `/v1/systemone` (llama.cpp or vLLM), either pick a `local` row (`model: pplx-flash-local`, endpoint from `MERGEONE_LOCAL_ENDPOINT`, default `http://127.0.0.1:8080/v1/systemone`, key `MERGEONE_LOCAL_KEY` optional), or define it inline:

```yaml
provider:
  dialect: systemone
  endpoint: http://10.0.0.5:8080/v1/systemone
  key_env: MY_GATE_KEY
  key_optional: true
```

After `--mcp --write`, restart Claude Code (or run `/mcp`) so the repository's `mergeone` server starts with the new policy.
