---
description: "Install and configure mergeone in this repository"
argument-hint: "[--ci] [--portal <token>]"
---

Set up mergeone here, following the `setup-mergeone` skill.

1. If `mergeone version` fails, tell the user how to install it (`brew install haalp/tap/mergeone`, or a release from https://github.com/haalp/mergeone/releases) and wait; do not install anything without their agreement.
2. Run `mergeone init --mcp --write $ARGUMENTS`. It does not overwrite an existing `.mergeone.yaml` unless `--force` is passed; do not add `--force` yourself.
3. Report what was written (`.mergeone.yaml`, `.mcp.json`, the `CLAUDE.md` snippet) and which API key the chosen model needs. Remind the user to keep keys in `.env` (ignored by git), and to restart Claude Code or run `/mcp` so the repository's server picks up the policy.
