# backstage

Internal developer platform cataloging every repo Murat Can Ozdemir builds, running [Backstage.io](https://backstage.io) itself.

Every owned repository registers itself in the Software Catalog via a `catalog-info.yaml` at its root, discovered automatically across the whole `muratcanozdemir` GitHub account — see `features/software-catalog/spec.md` for the full feature spec and `features/software-catalog/architecture.md` for how it's wired together.

## Running locally

```sh
yarn install
yarn start
```

Then open http://localhost:3000 and continue as a guest — no login required.

GitHub-wide discovery needs a `GITHUB_TOKEN` environment variable (read access to repository contents) to pick up new repos automatically. Without it, only locally-registered entities (this repo's own `catalog-info.yaml`) are visible.
