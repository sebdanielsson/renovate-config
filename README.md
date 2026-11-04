# renovate-config

Shared [Renovate](https://docs.renovatebot.com/) presets for my repositories.

## Usage

Extend the default preset in a repository's `renovate.json`:

```json
{
  "extends": ["github>sebdanielsson/renovate-config"]
}
```

## What `default.json` adds

- [`config:recommended`](https://docs.renovatebot.com/presets-config/#configrecommended).
- Automerge for minor and patch updates of dependencies at version 1.0 or later, once the required checks pass. Enable **Allow auto-merge** in the repository settings so GitHub merges the PR as soon as the checks are green. To keep a package out of automerge, add a rule that sets `automerge: false` for it; repository rules are applied after the preset, so it wins:

  ```json
  {
    "packageRules": [
      {
        "matchPackageNames": ["typescript"],
        "automerge": false
      }
    ]
  }
  ```

- `rangeStrategy: bump`, so ranges such as `^1.2.0` are raised to the new version instead of only updating the lockfile.
- `semanticCommits: enabled`, so update commits use Conventional Commits (`chore(deps): ...`, `fix(deps): ...`).
- `minimumReleaseAge: 1 day`, so a release that is pulled again shortly after publishing never reaches a PR. Repositories can set a longer age.
- Lock file maintenance with automerge. Once a week (Renovate's default, before 4am UTC on Mondays) Renovate deletes each lock file, regenerates it with the package manager and merges the result once the required checks pass. This is what fixes vulnerable transitive dependencies, which Renovate's vulnerability alerts can't reach yet for npm, pnpm or bun ([renovatebot/renovate#46377](https://github.com/renovatebot/renovate/issues/46377)). `minimumReleaseAge` doesn't apply here, because the package manager picks the versions, so set the package manager's own release age. pnpm defaults to 1 day; for the others, set it in the repository:

  | Package manager | Setting |
  | --- | --- |
  | bun | `minimumReleaseAge = 86400` under `[install]` in `bunfig.toml` |
  | npm | `min-release-age=1` in `.npmrc` |
  | uv | `exclude-newer = "24 hours"` under `[tool.uv]` in `pyproject.toml` |

- A `stylelint` group for `stylelint`, `stylelint-*`, `@stylelint/*` and scoped `@*/stylelint-*` packages (such as `@dreamsicle.io/stylelint-config-tailwindcss`), so Stylelint and its configs and plugins land in one PR. Majors get their own grouped PR as usual. A repository rule that sets `groupName` later, such as one grouping all patch updates, takes over for the updates it matches.
- `minimumReleaseAgeBehaviour: timestamp-optional` for Docker images. Only Docker Hub exposes release timestamps, so without it updates of GHCR, Quay and ECR images would wait for a timestamp forever and the images would never be updated. With it, an update that has no timestamp gets its PR right away.
- [`customManagers:githubActionsVersions`](https://docs.renovatebot.com/presets-customManagers/#custommanagersgithubactionsversions), which updates tool versions pinned inside workflow `run:` steps. Renovate's `github-actions` manager only reads `uses:` references, so a version hardcoded in a shell script is invisible to it. Move the version into an env var ending in `_VERSION` and annotate it on the line directly above:

```yaml
- name: Install Vercel CLI
  env:
    # renovate: datasource=npm depName=vercel
    VERCEL_CLI_VERSION: 59.16.0
  run: pnpm add "vercel@$VERCEL_CLI_VERSION"
```

Optional fields after `depName`, in this order: `packageName`, `versioning`, `extractVersion`, `registryUrl`.
