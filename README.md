# renovate-config

Shared [Renovate](https://docs.renovatebot.com/) presets for my projects and the yngatech and ldn-house organisations. Repositories run Renovate through the [Mend Renovate app](https://github.com/apps/renovate) and extend these presets from their own `renovate.json`.

| Preset | Extend with | What it does |
| --- | --- | --- |
| `default.json` | `github>incognitojam/renovate-config` | `config:recommended`, London timezone, `incognitojam` as assignee (automerging pull requests are only assigned if their checks fail), a three-day minimum release age for npm, PyPI, GitHub Actions and Docker images (where the registry reports release dates), and groups for packages that must move together: Solid 2.0, Effect, Playwright with its Docker image, Drizzle, the Cloudflare Workers SDK, and Bun. |
| `automerge.json` | `github>incognitojam/renovate-config:automerge` | Merges minor, patch and digest updates once the repository's checks pass. Major, 0.x, prerelease and Python minor updates wait for review. Only extend it in repositories whose pull requests run CI. |
| `scoped.json` | `github>incognitojam/renovate-config:scoped` | Titles commits and pull requests in [Scoped Commits](https://scopedcommits.com/) style: `deps: update ...`, and `ci: update ...` for GitHub Actions. |

A repository using all three:

```json
{
  "$schema": "https://docs.renovatebot.com/renovate-schema.json",
  "extends": [
    "github>incognitojam/renovate-config",
    "github>incognitojam/renovate-config:automerge",
    "github>incognitojam/renovate-config:scoped"
  ]
}
```
