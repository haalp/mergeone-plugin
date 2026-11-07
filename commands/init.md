---
description: Install and configure mergeone in this repository
argument-hint: [--ci] [--portal <token>]
allowed-tools: Bash(mergeone:*), Bash(curl -fsSL https://mergeone.dev/install.sh:*)
---

Set up mergeone here, following the `setup-mergeone` skill.

1. If `mergeone version` fails, tell the user the install command (`curl -fsSL https://mergeone.dev/install.sh | sh`) and run it only after they agree.
2. Run `mergeone init --mcp --write $ARGUMENTS`. It does not overwrite an existing `.mergeone.yaml` unless `--force` is passed; do not add `--force` yourself.
3. Report what was written and which API key the chosen model needs. Remind the user to keep keys in `.env` (ignored by git), and to restart Claude Code or run `/mcp` so the server picks up the policy.
