# Agent instructions

This repository holds shared Renovate presets. Each repository that uses them extends them from its own `renovate.json`, for example `github>incognitojam/renovate-config:automerge`. They are read straight from `main` by the hosted [Mend Renovate app](https://github.com/apps/renovate), so every push to `main` changes Renovate's behaviour in all of those repositories on their next run. There is no release step and no PR check to catch a mistake, so check every change before pushing.

This repository is public, but some repositories that use these presets are private. Never name a private repository, its files or its setup here, in commit messages, or in rule descriptions. To find the repositories that use the presets, read each repository's `renovate.json`. GitHub code search misses repositories it hasn't indexed. This takes about a minute and includes private repositories you can access:

```sh
for owner in incognitojam yngatech ldn-house; do
  gh repo list "$owner" --no-archived --limit 200 --json nameWithOwner --jq '.[].nameWithOwner'
done | while read -r repo; do
  gh api "repos/$repo/contents/renovate.json" --jq .content 2>/dev/null | base64 -d 2>/dev/null |
    grep -q 'incognitojam/renovate-config' && echo "$repo"
done
```

## Checking a change

Validate every preset before committing:

```sh
npx --yes --package renovate -- renovate-config-validator --strict default.json automerge.json scoped.json
```

For a change to what Renovate does (automerge, grouping, schedules, titles), also dry-run it against a repository that uses the presets, before pushing:

1. Serve this directory locally, for example `python3 -m http.server 8765 --bind 127.0.0.1`.
2. In a clone of that repository, point its `renovate.json` at the local presets: replace `github>incognitojam/renovate-config` with `http://127.0.0.1:8765/default.json` and `github>incognitojam/renovate-config:<name>` with `http://127.0.0.1:8765/<name>.json`. Restore the file afterwards.
3. Run `GITHUB_COM_TOKEN=$(gh auth token) LOG_LEVEL=debug npx --yes renovate --platform=local --dry-run=full --onboarding=false` from the repository root. `--platform=local` only sees files that git tracks, so a new `renovate.json` must be committed first. The `"packageFiles with updates"` log entry lists what Renovate found. `LOG_LEVEL=trace` adds each branch's resolved config, including `automerge` and `groupName`.

If the repository is already up to date, there may be nothing for a new rule to act on. Temporarily set a few versions back in the clone to exercise it, then restore them.

## Conventions

- Commit messages use Scoped Commits with the `presets:` scope, for example `presets: group Drizzle packages`.
- Give every package rule a `description` that says why the rule exists.
- When adding or changing a preset or group, update the table in `README.md`.
- Put a rule here only if it holds wherever its packages appear. A rule tied to one repository's files belongs in that repository's `renovate.json`, such as a custom manager for a version pinned inside one repository's Dockerfile.
- `automerge.json` must never merge major, 0.x or prerelease updates, or Python minor releases. Only extend it in repositories whose pull requests run CI, since automerge waits for those checks.
- Groups are for packages that have to move together, not just to reduce the number of PRs. `miniflare` stays out of the Cloudflare Workers SDK group because a repository pinning one of its prereleases would stop the whole group from automerging.
