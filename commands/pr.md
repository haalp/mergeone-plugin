---
description: Run the mergeone gate on a pull request and explain the verdict
argument-hint: <N> [owner/repo]
allowed-tools: Bash(mergeone pr:*), Bash(git remote get-url:*)
---

Run the gate on pull request $1.

1. Repository: use `$2` if given; otherwise take `owner/repo` from `git remote get-url origin`.
2. Run `mergeone pr <owner/repo> $1 --policy .mergeone.yaml` (needs `GITHUB_TOKEN` in the environment). If `mergeone` is missing or there is no `.mergeone.yaml`, follow the `setup-mergeone` skill instead.
3. Explain the result in two sentences: the verdict (`pass`, `hold` or `block`) with its route, then the rule and reason behind it. Include the head SHA.

The gate never merges or closes the PR; say so if the user seems to expect it.
