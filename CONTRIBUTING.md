# Contributing

CI validates `default.json` and this repository's own `.github/renovate.json` with `renovate-config-validator --strict`, and lints the Markdown. Run the same checks locally:

```bash
pnpm install
pnpm run lint
```

When you add a preset file, add it to the `lint:renovate` script in `package.json` so CI validates it too.

Everything is pinned: the actions by commit SHA, pnpm and Node.js in `devEngines`, and the npm packages in `pnpm-lock.yaml`. Renovate keeps them current through the shared preset, and bumps the `renovate` package once a week because it publishes several releases a day.
