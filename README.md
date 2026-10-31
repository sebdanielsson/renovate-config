# renovate-config

Shared [Renovate](https://docs.renovatebot.com/) presets for my repositories.

## Usage

Extend the default preset in a repository's `renovate.json`:

```json
{
  "extends": ["github>sebdanielsson/renovate-config"]
}
```

To also automerge minor and patch updates of stable dependencies, add the opt-in preset:

```json
{
  "extends": [
    "github>sebdanielsson/renovate-config",
    "github>sebdanielsson/renovate-config:automerge-stable"
  ]
}
```

## What `default.json` adds

- [`config:recommended`](https://docs.renovatebot.com/presets-config/#configrecommended).
- `rangeStrategy: bump`, so ranges such as `^1.2.0` are raised to the new version instead of only updating the lockfile.
- `semanticCommits: enabled`, so update commits use Conventional Commits (`chore(deps): ...`, `fix(deps): ...`).
- `minimumReleaseAge: 1 day`, so a release that is pulled again shortly after publishing never reaches a PR. Repositories can set a longer age.
- `minimumReleaseAgeBehaviour: timestamp-optional` for Docker images. Only Docker Hub exposes release timestamps, so without it updates of GHCR, Quay and ECR images would wait for a timestamp forever.
- [`customManagers:githubActionsVersions`](https://docs.renovatebot.com/presets-customManagers/#custommanagersgithubactionsversions), which updates tool versions pinned inside workflow `run:` steps. Renovate's `github-actions` manager only reads `uses:` references, so a version hardcoded in a shell script is invisible to it. Move the version into an env var ending in `_VERSION` and annotate it on the line directly above:

```yaml
- name: Install Vercel CLI
  env:
    # renovate: datasource=npm depName=vercel
    VERCEL_CLI_VERSION: 59.16.0
  run: pnpm add "vercel@$VERCEL_CLI_VERSION"
```

Optional fields after `depName`, in this order: `packageName`, `versioning`, `extractVersion`, `registryUrl`.

## What `automerge-stable.json` adds

Automerge for minor and patch updates of dependencies at version 1.0 or later, once CI passes. It is a separate preset because preset rules are applied before a repository's own rules: a repository rule that excludes a package, such as `"matchPackageNames": ["!typescript"]`, cannot undo an automerge rule from the preset. Repositories that need exclusions keep their own automerge rule instead.
