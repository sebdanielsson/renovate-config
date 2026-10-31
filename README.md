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
- [`customManagers:githubActionsVersions`](https://docs.renovatebot.com/presets-customManagers/#custommanagersgithubactionsversions), which updates tool versions pinned inside workflow `run:` steps. Renovate's `github-actions` manager only reads `uses:` references, so a version hardcoded in a shell script is invisible to it. Move the version into an env var ending in `_VERSION` and annotate it on the line directly above:

```yaml
- name: Install Vercel CLI
  env:
    # renovate: datasource=npm depName=vercel
    VERCEL_CLI_VERSION: 59.16.0
  run: pnpm add "vercel@$VERCEL_CLI_VERSION"
```

Optional fields after `depName`, in this order: `packageName`, `versioning`, `extractVersion`, `registryUrl`.
