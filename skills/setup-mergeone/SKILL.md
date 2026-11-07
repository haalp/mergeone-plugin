---
name: setup-mergeone
description: Use when mergeone is not installed or not configured: the `mergeone` command is missing, the mergeone MCP server fails to start, there is no .mergeone.yaml, or the user asks to set up, install or configure the merge gate.
---

# Set up mergeone

The plugin starts `mergeone mcp --policy .mergeone.yaml`. It needs the binary and a policy file in the repository.

1. **Install the binary** (macOS / Linux, one static binary to `/usr/local/bin/mergeone`):
   ```bash
   curl -fsSL https://mergeone.dev/install.sh | sh
   mergeone version
   ```
2. **Write the policy** in the repository root: `mergeone init`. It prints what it detected, writes `.mergeone.yaml` with safe defaults (PR memory off, verdict actions off, fail closed) and never overwrites an existing file without `--force`.
   - `--mcp --write` appends the "call gate_decide before merging" snippet to `CLAUDE.md`.
   - `--ci` writes `.github/workflows/mergeone.yml`; `--all` is `--ci --mcp --write`.
   - `--model <name>` picks the decision model (`mergeone models` lists them).
   - `--portal <token>` (optional) posts gatelog records to the hosted portal. The token comes from the portal; do not invent one.
3. **Keys** go in the environment or a `.env` that is in `.gitignore`, never in the repository or a committed file. Per model: `TYPESAFE_API_KEY` (`jev`, default), `PERPLEXITY_API_KEY` (`pplx`, `pplx-flash`), `ANTHROPIC_API_KEY` (`haiku`). `GITHUB_TOKEN` is needed only for `mergeone pr` on a remote PR, `mergeone index` and verdict actions. Never print a key.
4. **No key yet**: `mergeone mcp --mock` runs the pipeline with a stand-in model. Use it to check wiring only; its verdicts mean nothing.

## What `.mergeone.yaml` holds

`model`, `question_set` (`v2` default), `always_human_paths` (prefixes that always hold for a human), `memory` (PR memory, needed by `gate_similar`; set `enabled: true`, then run `mergeone index owner/name`), `rules` (first match wins, each with a verdict and a route), `default`, `on_error` (fail closed), `drift`, `actions` (off by default; never merges) and the optional `portal` block. Full reference: https://github.com/haalp/mergeone/blob/main/docs/POLICY.md.

## Self-hosted model

To use your own server that speaks `/v1/systemone` (llama.cpp or vLLM), either pick a `local` row (`model: pplx-flash-local`, endpoint from `MERGEONE_LOCAL_ENDPOINT`, default `http://127.0.0.1:8080/v1/systemone`, key `MERGEONE_LOCAL_KEY` optional), or define it inline:

```yaml
provider:
  dialect: systemone
  endpoint: http://10.0.0.5:8080/v1/systemone
  key_env: MY_GATE_KEY
  key_optional: true
```

After setup, restart Claude Code (or run `/mcp`) so the `mergeone` server starts with the new policy.
