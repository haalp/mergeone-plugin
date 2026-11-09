# mergeone for Claude Code

A merge gateway for agent pull requests. This plugin teaches Claude Code to ask the gate before it merges and to check for duplicates before it opens a PR. It is skills and commands around the `mergeone` command-line tool; it bundles no server and no code. The gate never merges and never closes anything; a `hold` is a request for a human, not a rejection.

## Install

The plugin starts the `mergeone` binary, so install that first:

```bash
brew install haalp/tap/mergeone            # or a signed release archive: https://github.com/haalp/mergeone/releases
```

Then the plugin, from the community catalog:

```bash
claude plugin install mergeone
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
| skill `ask-the-gate` | asks the gate before a merge — `mergeone pr`, or the repository's `gate_decide` MCP tool when it has one; `pass` allows it, `hold` requests the named route, `block` stops. Shows the head SHA and the rule |
| skill `before-you-open-a-pr` | checks for a duplicate before a PR is opened (`gate_similar`, when the repository's MCP server and PR memory are set up); links the existing PR instead of opening another |
| skill `setup-mergeone` | install, `mergeone init`, what `.mergeone.yaml` holds, shadow mode for the first week, `mergeone policy test`, a self-hosted `/v1/systemone` model, keys in `.env` |
| `/mergeone:pr <N>` | runs `mergeone pr <owner/repo> <N>` and explains the verdict in two sentences |
| `/mergeone:init` | runs `mergeone init --mcp --write` |

**The MCP server is the repository's, not the plugin's.** `mergeone init --mcp --write` writes `mergeone mcp --policy .mergeone.yaml` into the repository's `.mcp.json` (tools `gate_decide`, `gate_similar`, `gate_plan`, `gate_verdict`, `gate_stats`), where the team can see and review it. Without it the skills use the CLI; `gate_similar` additionally needs PR memory (`memory: { enabled: true }`, then `mergeone index owner/name` with `GITHUB_TOKEN` set).

## What it runs, reads and sends

Everything the plugin does goes through the `mergeone` binary you installed; the plugin itself contains no code
beyond the skill and command text.

- **Runs:** `mergeone pr`, `mergeone init` and `mergeone version` from the skills and commands, with your approval
  as for any shell command. Installing the binary is never done without your agreement. The plugin starts no server.
- **Reads:** the repository's git diff and `.mergeone.yaml`; with `GITHUB_TOKEN` set, the pull request's title, body
  and files from the GitHub API. Keys are read from your environment or a git-ignored `.env`: the decision model's
  key (`TYPESAFE_API_KEY`, `PERPLEXITY_API_KEY` or `ANTHROPIC_API_KEY`, one of them) and optionally `GITHUB_TOKEN`.
- **Sends:** the rendered PR state (title, body, file list, a bounded diff) to the decision-model endpoint of the
  model named in `.mergeone.yaml` — Typesafe (`jev`), Perplexity (`pplx`, `pplx-flash`), Anthropic (`haiku`) or your
  own `/v1/systemone` server. With `actions.enabled: true`, a commit status, labels and one comment to GitHub. With a
  `portal` block, the gatelog record (verdict, probabilities, no diff text) to the portal URL you configured.
  Nothing is sent anywhere else; nothing is sent at all until you run a gate tool or command.
- **Keeps:** `.mergeone/gatelog.jsonl` in your repository folder (verdicts, probabilities, the hash of the input). No
  personal data beyond what the PR itself carries (author login, title, body).

The gate never merges and never closes a pull request.

## Keys

Keys stay in your environment or a git-ignored `.env`, never in the repository; the plugin asks for none. `jev` (default) reads `TYPESAFE_API_KEY`, `pplx-flash` reads `PERPLEXITY_API_KEY`, `haiku` reads `ANTHROPIC_API_KEY`. No key yet: `mergeone mcp --mock` checks the wiring with a stand-in model.

## Links

- Site: https://mergeone.dev
- Gateway and benchmark: https://github.com/haalp/mergeone
- What was measured, including the negative results: https://github.com/haalp/mergeone/blob/main/docs/EVIDENCE.md
- Policy reference: https://github.com/haalp/mergeone/blob/main/docs/POLICY.md

★ the repo if it helps.

MIT, see [LICENSE](LICENSE).
