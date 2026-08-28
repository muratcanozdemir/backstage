# Feature Specification: Software Catalog

**Status:** APPROVED
**Challenged:** No
**UI Required:** Yes
**Created:** 2026-08-29
**Feature:** Stand up a real Backstage.io instance as an internal developer platform that catalogs every repository the owner builds, starting with the 10 existing sibling repos plus the backstage repo itself.

## Overview

The owner maintains a growing collection of independent software projects (currently 10 repos: bazel-maven-resolver, codegraph-lsp, gated, github-probe/org-harvest, grease, lattice, notecast-py, pr-drift, s3-vectors-at-home, serverless-to-ecs) but has no single place to see what exists, what each project does, what shape it's in, or how mature it is. This feature turns the `backstage` repo into a running instance of Backstage.io — Spotify's open-source internal developer platform — so the owner can browse a live Software Catalog listing every owned repository (including `backstage` itself) with consistent metadata: purpose, tech stack, license, ownership, lifecycle stage, and rendered documentation.

Because the catalog surfaces license status directly, a handful of real inconsistencies found across the repos during exploration (missing LICENSE files, a license mismatch, a placeholder module path, one repo with no CI/CD at all) are corrected as part of this feature, so what the catalog reports is accurate rather than cataloging known-wrong data.

## User Stories

### US-1: Browse the catalog
As the owner, I want to open Backstage and see every one of my repositories listed in one place, so I can get an at-a-glance inventory of what I've built without visiting each repo individually.

**Acceptance Criteria:**
- **AC-1.1:** Opening the Backstage app shows the Software Catalog as the primary/home view.
- **AC-1.2:** All 11 owned repositories (the 10 sibling repos + backstage itself) appear as Components in the catalog list.
- **AC-1.3:** `proton` (Valve's upstream repo, not owned by the user) does not appear in the catalog.
- **AC-1.4:** Each list entry shows at minimum: name, description/purpose, owner, and lifecycle stage.

### US-2: View a component's full metadata
As the owner, I want to open any single repository's catalog entry and see its full metadata, so I understand its purpose, stack, and status without reading its source.

**Acceptance Criteria:**
- **AC-2.1:** Clicking a component opens a detail view showing: description/purpose, primary language(s)/tech stack, project type (library, CLI tool, service, etc.), license, owner, lifecycle stage, and a link to the repository on GitHub.
- **AC-2.2:** The metadata shown matches the facts already gathered in `features/software-catalog/exploration.md` for that repo.

### US-3: Read a repository's documentation from the catalog
As the owner, I want to read a repository's README as rendered documentation inside Backstage, so I don't need to leave the catalog to understand how to use a project.

**Acceptance Criteria:**
- **AC-3.1:** Each of the 11 components has a "Docs" view in Backstage rendering that repository's README.md content.
- **AC-3.2:** The docs view is reachable from the component's detail page (US-2).

### US-4: New repositories appear automatically
As the owner, I want a new repository I create later to show up in the catalog without me reconfiguring Backstage, so the catalog stays current with minimal upkeep.

**Acceptance Criteria:**
- **AC-4.1:** Backstage is configured to discover catalog entities across the owner's entire GitHub account, not just an explicit list of the 10 known repos.
- **AC-4.2:** A repository only appears in the catalog once it contains a `catalog-info.yaml` file; repos without one are not surfaced.
- **AC-4.3:** Adding a `catalog-info.yaml` to a new repository makes it appear in the catalog without any change to Backstage's own configuration.

### US-5: Open the catalog without logging in
As the sole user of this instance, I want to open Backstage and land directly in the catalog with no sign-in step, so there's no auth friction for a tool only I use.

**Acceptance Criteria:**
- **AC-5.1:** Opening the Backstage app requires no login/credentials entry.
- **AC-5.2:** The owner has full read access to the catalog and docs immediately on open.

### US-6: See accurate ownership on every component
As the owner, I want every catalog entry to point to a real "owner" profile rather than a plain text label, so ownership is modeled consistently and could support filtering by owner later.

**Acceptance Criteria:**
- **AC-6.1:** A Backstage User entity exists representing the owner.
- **AC-6.2:** All 11 components' `owner` field references that User entity.

### US-7: Trust the metadata the catalog reports
As the owner, I want the specific data-quality issues found during exploration fixed at the source, so the catalog reports accurate facts instead of repeating known-wrong ones.

**Acceptance Criteria:**
- **AC-7.1:** `bazel-maven-resolver`, `s3-vectors-at-home`, and `serverless-to-ecs` each have a LICENSE file (MIT), matching the convention used by the rest of the owner's repos.
- **AC-7.2:** `lattice` has a standalone LICENSE file (MIT) matching what its `pyproject.toml` already declares.
- **AC-7.3:** `gated`'s LICENSE file is changed to Apache-2.0, resolving the mismatch with its README (which already claimed Apache-2.0).
- **AC-7.4:** `grease`'s Go module path is corrected from the placeholder `github.com/user/grease` to `github.com/muratcanozdemir/grease`, with all internal imports updated to match.
- ~~**AC-7.5:** `serverless-to-ecs` has CI (lint/vet/test) and release (tag-triggered build + GitHub release) workflows added, matching the pattern used by the owner's other Go repos (e.g. `gated`, `grease`), and keeps its existing dependabot configuration.~~ **VOIDED 2026-08-29:** the original exploration finding this was based on was wrong — `serverless-to-ecs` already has working CI and release workflows (added in commit `f760727`, with dependabot action-bump PRs already merged on top), discovered during Story 11 implementation via `git log`. No fix needed here; see the correction note in `exploration.md`'s `serverless-to-ecs` section.
- **AC-7.6:** The `backstage` repo itself has a LICENSE file (MIT), consistent with the rest of the owner's repos.
- **AC-7.7:** Each affected repo's catalog entry reflects the corrected license/status once fixed.

## Functional Requirements

### FR-1: Catalog entity registration
- Each of the 11 owned repositories (the 10 sibling repos plus `backstage`) shall have a `catalog-info.yaml` file at its repository root registering it as a Backstage Component entity.
- Each entity's `metadata.description` shall summarize the repo's purpose as captured in `features/software-catalog/exploration.md`.
- Each entity's `spec.type` shall reflect its project type from exploration (e.g. `library`, `tool`, `service`).
- Each entity's `spec.lifecycle` shall be `experimental` at launch for all 11 components, reflecting their current early-stage maturity (all v0.1.0, low commit counts).
- Each entity shall link back to its GitHub repository.

### FR-2: Ownership
- A Backstage User entity representing the owner shall exist.
- Every component's `spec.owner` shall reference that User entity (not a plain string).

### FR-3: GitHub-wide discovery
- Backstage shall be configured with a GitHub-based discovery mechanism that scans the owner's GitHub account for `catalog-info.yaml` files, rather than a hardcoded list of 10 repos.
- Repos without a `catalog-info.yaml` shall not appear in the catalog.

### FR-4: Documentation (TechDocs)
- Each of the 11 components shall have TechDocs enabled, rendering that repository's existing `README.md` as browsable documentation within Backstage.

### FR-5: Access
- The Backstage instance shall use guest access (no login/credential step) since it is a single-user, local tool.

### FR-6: Repository corrections
- `bazel-maven-resolver`, `s3-vectors-at-home`, `serverless-to-ecs`, and `backstage` shall each gain a LICENSE file (MIT, matching the authorship convention already used across the owner's other repos).
- `lattice` shall gain a standalone LICENSE file (MIT) matching what it already declares in `pyproject.toml`.
- `gated`'s LICENSE file shall be changed to Apache-2.0 (matching what its README already states).
- `grease`'s `go.mod` module path shall be corrected from `github.com/user/grease` to `github.com/muratcanozdemir/grease`, with internal import paths updated accordingly.
- ~~`serverless-to-ecs` shall gain CI and release GitHub Actions workflows...~~ **VOIDED** — already has working CI/release workflows; see AC-7.5 correction above.

## Scope

### In Scope
- Scaffolding a real, runnable Backstage.io instance in the `backstage` repo.
- A `catalog-info.yaml` for each of the 11 owned repos (10 siblings + `backstage`), committed to each respective repo.
- GitHub-account-wide discovery configuration so future repos self-register.
- Guest (no-login) access.
- TechDocs enabled for all 11 components, rendering existing READMEs.
- A User entity for the owner, referenced as `owner` on every component.
- The specific repository fixes listed in FR-6 / US-7.

### Out of Scope
- Registering `proton` (not owned by the user) in the catalog.
- Deploying this Backstage instance anywhere beyond local/personal use (no shared hosting, no production deployment).
- Backstage plugins beyond Software Catalog and TechDocs (e.g. Kubernetes, CI/CD status, cost insights) — not requested.
- Grouping components under a System or Domain entity — the 10 repos are independent tools, not parts of one larger system.
- Any repository fixes beyond the specific ones listed in FR-6 (e.g. no broader refactors, no adding tests, no version bumps beyond what's needed for the listed fixes).
- Adding CI/CD to any repo other than `serverless-to-ecs`.

## Edge Cases

### EC-1: Malformed or incomplete catalog-info.yaml
- **Scenario:** A `catalog-info.yaml` is missing a required field or has invalid YAML.
- **Expected Behavior:** Backstage surfaces its standard processing error for that entity in its own UI; the rest of the catalog continues to load normally.

### EC-2: GitHub discovery finds a repo with no catalog-info.yaml
- **Scenario:** The owner has (or later creates) a GitHub repo that has no `catalog-info.yaml`.
- **Expected Behavior:** That repo is silently ignored by discovery — it does not appear in the catalog and does not produce an error.

### EC-3: GitHub API access issues during discovery
- **Scenario:** The discovery processor hits a GitHub API rate limit or an expired/invalid token.
- **Expected Behavior:** The catalog continues showing the last successfully discovered state rather than going empty; this is Backstage's default processor behavior and requires no custom handling.

### EC-4: A cataloged repository is deleted or renamed on GitHub
- **Scenario:** A previously-registered repo disappears from GitHub.
- **Expected Behavior:** Its entity is marked orphaned per Backstage's default behavior; no custom handling required.

## Non-Functional Hints

- **Performance:** Trivial scale (11 entities today) — catalog list and detail views should load effectively instantly.
- **Security:** Guest access means no access control on this instance; acceptable since it's single-user and local-only. The GitHub token used for discovery needs only read access to repository contents/metadata and must not be committed to any repository.
- **Scalability:** Not a concern at current or near-term scale.

## Dependencies

- `features/software-catalog/exploration.md` — source of the per-repo metadata (purpose, stack, type, license, maturity) used to populate each `catalog-info.yaml`.
- A GitHub personal access token (or equivalent) with read access to the owner's repositories, required for the discovery mechanism (FR-3).
- Write access to push corrections to `bazel-maven-resolver`, `s3-vectors-at-home`, `serverless-to-ecs`, `lattice`, and `gated` (FR-6).

## Open Questions

None — all decisions needed to implement this spec were resolved during requirements gathering. One default worth confirming at approval: the `backstage` repo itself and `lattice`'s standalone LICENSE file were assumed to follow the same MIT convention as the rest of the batch (extending the already-approved "missing licenses → MIT" decision); flag at approval if either should differ.
