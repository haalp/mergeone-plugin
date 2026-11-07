# mergeone for Claude Code

A merge gateway for agent pull requests. This plugin lets Claude Code ask the gate before it merges and look for duplicates before it opens a PR. The gate never merges and never closes anything; a `hold` is a request for a human, not a rejection.

## Install

The plugin starts the `mergeone` binary, so install that first:

```bash
curl -fsSL https://mergeone.dev/install.sh | sh
```

Then the plugin, from the community catalog:

```bash
claude plugin install mergeone@claude-community
```

or from this repository:

```bash
claude plugin marketplace add haalp/mergeone-plugin
claude plugin install mergeone@mergeone-plugin
```

In your repository, run `/mergeone:init` (or `mergeone init --mcp --write`), set the API key for your model in `.env`, and restart Claude Code.

## What is in it

| piece | what it does |
|---|---|
| `.mcp.json` | stdio server `mergeone mcp --policy .mergeone.yaml`; tools `gate_decide`, `gate_similar`, `gate_plan`, `gate_verdict`, `gate_stats` |
| skill `before-you-open-a-pr` | calls `gate_similar` before a PR is opened; links an existing PR instead of opening a duplicate |
| skill `ask-the-gate` | calls `gate_decide` before a merge; `pass` allows it, `hold` requests the named route, `block` stops. Shows the head SHA and the rule |
| skill `setup-mergeone` | install, `mergeone init`, what `.mergeone.yaml` holds, a self-hosted `/v1/systemone` model, keys in `.env` |
| `/mergeone:pr <N>` | runs `mergeone pr <owner/repo> <N>` and explains the verdict in two sentences |
| `/mergeone:init` | runs `mergeone init --mcp --write` |

`gate_similar` needs PR memory: `memory: { enabled: true }` in `.mergeone.yaml`, then `mergeone index owner/name` with `GITHUB_TOKEN` set.

## Keys

Keys stay in your environment or a git-ignored `.env`, never in the repository. `jev` (default) reads `TYPESAFE_API_KEY`, `pplx-flash` reads `PERPLEXITY_API_KEY`, `haiku` reads `ANTHROPIC_API_KEY`. No key yet: `mergeone mcp --mock` checks the wiring with a stand-in model.

## Links

- Site: https://mergeone.dev
- Gateway and benchmark: https://github.com/haalp/mergeone
- What was measured, including the negative results: https://github.com/haalp/mergeone/blob/main/docs/EVIDENCE.md
- Policy reference: https://github.com/haalp/mergeone/blob/main/docs/POLICY.md

★ the repo if it helps.

MIT, see [LICENSE](LICENSE).
