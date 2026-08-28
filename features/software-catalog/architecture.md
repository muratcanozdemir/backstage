# Architecture Brief: Software Catalog

**Created:** 2026-08-29

## Key Decisions

### Cross-repo work uses feature branches + PRs, matching this repo's own convention
**Decision:** Every touch to one of the 10 external sibling repos (adding `catalog-info.yaml`, license fixes, `grease`'s module-path fix, `serverless-to-ecs`'s new CI workflows) happens on a `feature/software-catalog` branch in that repo, with a PR opened before merge — not a direct commit to `main`.
**Reasoning:** User's explicit choice. Also keeps the same branch-per-feature discipline this `backstage` repo already follows (per this skill's own Step 1), rather than treating external-repo edits as a special case.
**Alternative considered:** Direct commits to each repo's default branch — rejected as inconsistent with the convention already in use.

### Backstage app is scaffolded at the root of this repo
**Decision:** Run the standard `npx @backstage/create-app` scaffold directly into `/home/murat/backstage` (npm as package manager), so this repo's root *is* the Backstage app (`packages/app`, `packages/backend`, `app-config.yaml`, etc.) rather than nesting it in a subdirectory.
**Reasoning:** The repo is named `backstage` and the entire point of this feature is for it to *be* the running instance — nesting would just add an indirection layer with no benefit. This is the only reasonable reading of "establish Backstage as a store of my software... in /home/murat/backstage."
**Alternative considered:** Scaffolding into a subdirectory (e.g. `app/`) — rejected, adds indirection with no upside for a single-instance personal tool.

### GitHub discovery uses the GitHub Entity Provider, not the legacy Discovery processor
**Decision:** Configure `catalog.providers.github` (the entity-provider-based, schedule-driven discovery mechanism) targeting `https://github.com/muratcanozdemir/*` with `catalogPath: /catalog-info.yaml`, rather than the older config-driven `GithubDiscoveryProcessor`.
**Reasoning:** This is Backstage's currently-recommended approach for GitHub-account-wide discovery (incremental refresh on a schedule vs. full-catalog processor reload) and is what FR-3 (whole-account discovery, new repos self-register) needs.
**Alternative considered:** Legacy `catalog.processors.githubOrg`/discovery processor — deprecated approach, no reason to pick it for a fresh setup.

### TechDocs uses the local builder + local publisher
**Decision:** `techdocs.builder: local`, `techdocs.publisher.type: local`.
**Reasoning:** Spec explicitly scopes this instance to local/personal use only (no shared hosting). The local builder/publisher is Backstage's zero-infra default and needs no cloud storage bucket — appropriate for that scope. Docs source is each repo's existing `README.md` (no new `docs/` folder or `mkdocs.yml` structure required beyond Backstage's default README fallback).
**Alternative considered:** External builder (CI-built docs published to cloud storage) — unnecessary complexity for a single local user.

### Guest auth, no sign-in page
**Decision:** Configure `auth.providers.guest` and set guest as the resolver so opening the app lands directly in the catalog with no credential prompt.
**Reasoning:** Directly matches spec FR-5/US-5 (no login for a single-user local tool) — this is Backstage's built-in mechanism for exactly this case.

### GitHub token supplied via environment variable, never committed
**Decision:** `integrations.github[0].token` in `app-config.yaml` reads from `${GITHUB_TOKEN}`; the actual token lives in a gitignored local env file (e.g. `app-config.local.yaml` or `.env`, already gitignored by Backstage's default scaffold).
**Reasoning:** Standard Backstage secret-handling pattern; required so discovery (FR-3) can authenticate without ever putting a credential in a tracked file.

### Owner is a single Backstage User entity, defined once in this repo
**Decision:** Create one `catalog-info.yaml`-style User entity (e.g. `catalog/org.yaml` in the `backstage` repo, entity ref `user:default/muratcanozdemir`) as part of the app-scaffold story. Every component's `spec.owner` references `user:default/muratcanozdemir`.
**Reasoning:** Spec FR-2 requires a real User entity, not a string. Defining it once in the `backstage` repo (rather than duplicating it per external repo) avoids 11 copies of the same entity definition.
**Constraint this creates:** The User entity must exist and be catalogued *before* (or in the same discovery cycle as) the components that reference it, or Backstage will show a broken-owner-ref warning until it catches up — not a blocker, just an expected transient state.

## Constraints

- Every `catalog-info.yaml` added to an external repo must use `metadata.name` equal to that repo's actual GitHub repository name (e.g. `github-probe`, not the internal Python package name `org-harvest`) — discovery matches entities to repos by URL, and a mismatched name would be confusing since the GitHub URL and catalog name would disagree. The package's actual distributed name (e.g. "org-harvest") belongs in `metadata.description`/`title`, not `metadata.name`.
- `spec.lifecycle: experimental` and `spec.owner: user:default/muratcanozdemir` are identical across all 11 components — no per-repo variation needed.
- `spec.type` varies per repo; use the mapping below (derived from exploration.md's "Type of project" field for each repo) so every story implementer uses the same value instead of re-deriving it:

| Repo | `spec.type` |
|---|---|
| bazel-maven-resolver | `library` |
| codegraph-lsp | `tool` |
| gated | `service` |
| github-probe | `tool` |
| grease | `tool` |
| lattice | `library` |
| notecast-py | `tool` |
| pr-drift | `service` |
| s3-vectors-at-home | `service` |
| serverless-to-ecs | `tool` |
| backstage | `service` |

- Each external repo's PR should be self-contained: where a repo needs both a `catalog-info.yaml` **and** a fix from FR-6 (e.g. `gated` needs its license fixed *and* its catalog-info.yaml added), do both in the same branch/PR for that repo rather than opening two PRs against the same repo — see Cross-Story Coordination.
- `proton` must never receive a `catalog-info.yaml` — it is explicitly out of scope (not owned by the user).

## Pattern Reference

- **Component entity template** (apply to all 11, varying only `name`, `description`, `spec.type`, and `metadata.tags` for language/stack):
  ```yaml
  apiVersion: backstage.io/v1alpha1
  kind: Component
  metadata:
    name: <github-repo-name>
    description: <one-line purpose, from exploration.md>
    tags: [<language>, <stack keywords>]
    annotations:
      github.com/project-slug: muratcanozdemir/<github-repo-name>
  spec:
    type: <see mapping table above>
    lifecycle: experimental
    owner: user:default/muratcanozdemir
  ```
- **User entity** (define once, in this repo):
  ```yaml
  apiVersion: backstage.io/v1alpha1
  kind: User
  metadata:
    name: muratcanozdemir
  spec:
    profile:
      displayName: Murat Can Ozdemir
    memberOf: []
  ```
- **Source of per-repo facts:** `features/software-catalog/exploration.md` — purpose, stack, type, license, and CI status for each repo already gathered there; stories should pull from it rather than re-exploring each repo.

## Cross-Story Coordination

- Build the Backstage app scaffold + `app-config.yaml` (discovery, guest auth, TechDocs, User entity) as one story before or alongside the first component-registration story — every component references the User entity and depends on discovery being configured to find it.
- One story per external repo, each covering *everything* that repo needs: its `catalog-info.yaml` plus any FR-6 fix that applies to it. Concretely: `gated` (catalog-info.yaml + Apache-2.0 relicense), `grease` (catalog-info.yaml + module-path fix), `serverless-to-ecs` (catalog-info.yaml + new CI/release workflows), `bazel-maven-resolver` / `s3-vectors-at-home` (catalog-info.yaml + new LICENSE), `lattice` (catalog-info.yaml + standalone LICENSE file), and a plain catalog-info.yaml-only story for each of `codegraph-lsp`, `github-probe`, `notecast-py`, `pr-drift`. `backstage` itself gets a catalog-info.yaml + LICENSE story within this same repo (no separate branch needed since it's already on `feature/software-catalog`).
- No ordering dependency between the 10 external-repo stories themselves — they're independent and can be implemented/reviewed in any order or in parallel.
- Discovery config (in `app-config.yaml`) doesn't need to "know about" each repo individually — once it's configured against the whole GitHub account, every external-repo PR that lands automatically becomes visible in the catalog on the next discovery cycle. No story needs to touch `app-config.yaml` again after the scaffold story.
