# Test Scenarios: Software Catalog

**Status:** APPROVED
**Created:** 2026-08-29
**Spec:** [features/software-catalog/spec.md](spec.md)

## Feature Overview

A real Backstage.io instance catalogs 11 owned repositories (10 sibling repos + `backstage` itself) with consistent metadata, GitHub-wide auto-discovery, guest access, and TechDocs, plus source-level fixes to 4 repos so the metadata the catalog reports is accurate.

## Preconditions

### System State
- Backstage app running locally (`yarn start` / `npm start` from the scaffolded app, per Story 1).
- Each of the 11 owned repos has its `catalog-info.yaml` present on its default branch (or the relevant `feature/software-catalog` branch checked out locally, since no PRs are merged as part of this feature).
- GitHub discovery token configured via environment variable, with read access to the owner's repos.

### Test Data
- The 11 repos and their expected metadata as documented in `features/software-catalog/exploration.md`.
- `proton` present on disk at `/home/murat/proton` as a negative-case fixture (should never be cataloged).

---

## Test Scenarios

### US-1: Browse the catalog

#### Scenario 1.1: Full catalog list on open [AC-1.1, AC-1.2, AC-1.4]

**Given** all 11 owned repos have a `catalog-info.yaml` merged into the catalog
**When** the owner opens the Backstage app
**Then** the Software Catalog is shown as the home view
**And** all 11 components appear in the list
**And** each entry shows name, description, owner, and lifecycle stage

**Example Data:**
| Field | Value |
|-------|-------|
| Component | gated |
| Owner | muratcanozdemir |
| Lifecycle | experimental |

---

#### Scenario 1.2: proton is never cataloged [AC-1.3]

**Given** `proton` has no `catalog-info.yaml` (it is not owned by the user)
**When** the owner browses the catalog
**Then** `proton` does not appear in the component list

---

### US-2: View a component's full metadata

#### Scenario 2.1: Component detail view shows accurate metadata [AC-2.1, AC-2.2]

**Given** the `gated` component is registered
**When** the owner opens its detail page
**Then** the page shows description, tech stack, project type (`service`), license (Apache-2.0), owner, lifecycle (`experimental`), and a link to its GitHub repo
**And** every field matches the facts recorded in `exploration.md` for `gated`

**Example Data:**
| Field | Value |
|-------|-------|
| Type | service |
| License | Apache-2.0 |
| Owner | user:default/muratcanozdemir |

---

### US-3: Read a repository's documentation from the catalog

#### Scenario 3.1: README renders as TechDocs [AC-3.1, AC-3.2]

**Given** a component (e.g. `lattice`) has TechDocs enabled
**When** the owner opens the "Docs" tab from that component's detail page
**Then** the repo's `README.md` content renders as a browsable documentation page

---

### US-4: New repositories appear automatically

#### Scenario 4.1: New catalog-info.yaml is discovered without reconfiguration [AC-4.1, AC-4.3]

**Given** GitHub-wide discovery is configured against the owner's account
**And** a repo in that account gains a new `catalog-info.yaml` on its default branch
**When** Backstage's next discovery refresh cycle runs
**Then** that repo's component appears in the catalog
**And** no change was made to Backstage's own configuration to make it appear

---

#### Scenario 4.2: Repo without catalog-info.yaml stays absent [AC-4.2]

**Given** a repo in the owner's GitHub account has no `catalog-info.yaml`
**When** Backstage's discovery refresh cycle runs
**Then** that repo does not appear in the catalog

---

### US-5: Open the catalog without logging in

#### Scenario 5.1: Guest access lands directly in the catalog [AC-5.1, AC-5.2]

**Given** the Backstage instance is configured with guest auth
**When** the owner opens the app
**Then** no login/credential prompt is shown
**And** the owner has full read access to the catalog and docs immediately

---

### US-6: See accurate ownership on every component

#### Scenario 6.1: Every component references the owner's User entity [AC-6.1, AC-6.2]

**Given** a User entity for `muratcanozdemir` exists in the catalog
**When** the owner views the "owner" field on any of the 11 components
**Then** it links to that User entity's profile, not a plain text label

---

### US-7: Trust the metadata the catalog reports

#### Scenario 7.1: Missing-license repos gain a LICENSE file [AC-7.1]

**Given** `bazel-maven-resolver`, `s3-vectors-at-home`, and `serverless-to-ecs` previously had no LICENSE file
**When** the fix is applied
**Then** each repo has a LICENSE file identifying MIT and Murat Can Ozdemir as copyright holder

---

#### Scenario 7.2: lattice gains a standalone LICENSE file [AC-7.2]

**Given** `lattice` declares MIT only in `pyproject.toml`, with no standalone LICENSE file
**When** the fix is applied
**Then** `lattice` has a standalone MIT LICENSE file consistent with that declaration

---

#### Scenario 7.3: gated's license mismatch is resolved to Apache-2.0 [AC-7.3, AC-7.7]

**Given** `gated`'s LICENSE file said MIT while its README claimed Apache-2.0
**When** the fix is applied
**Then** `gated`'s LICENSE file is Apache-2.0
**And** its catalog entry's license field shows Apache-2.0

---

#### Scenario 7.4: grease's module path is corrected [AC-7.4]

**Given** `grease`'s `go.mod` declared the placeholder module path `github.com/user/grease`
**When** the fix is applied
**Then** `go.mod` declares `github.com/muratcanozdemir/grease`
**And** all internal imports reference the corrected path
**And** the module still builds

---

#### Scenario 7.5: serverless-to-ecs gains CI/CD parity [AC-7.5]

**Given** `serverless-to-ecs` had no GitHub Actions workflows
**When** the fix is applied
**Then** the repo has a CI workflow (lint/vet/test) and a release workflow (tag-triggered build + GitHub release)
**And** its existing dependabot configuration is preserved

---

#### Scenario 7.6: backstage repo gains a LICENSE file [AC-7.6]

**Given** the `backstage` repo previously had no LICENSE file
**When** the fix is applied
**Then** `backstage` has a MIT LICENSE file consistent with the rest of the owner's repos

---

## Error Scenarios

### Scenario E-1: Malformed catalog-info.yaml [EC-1]

**Given** a component's `catalog-info.yaml` has invalid YAML or a missing required field
**When** Backstage processes the catalog
**Then** Backstage surfaces its standard processing error for that entity
**And** the rest of the catalog continues to load and display normally

**Example Data:**
| Input | Expected Error |
|-------|----------------|
| `catalog-info.yaml` missing `spec.type` | Entity processing error shown for that component only |

---

### Scenario E-2: GitHub API access issue during discovery [EC-3]

**Given** the discovery processor hits a GitHub API rate limit or an expired token
**When** the next discovery refresh cycle runs
**Then** the catalog continues showing its last successfully discovered state
**And** no existing entities disappear because of the failed refresh

---

### Scenario E-3: A cataloged repository disappears from GitHub [EC-4]

**Given** a previously-registered component's GitHub repo is deleted or renamed
**When** the next discovery refresh cycle runs
**Then** that entity is marked orphaned per Backstage's default behavior

---

## Boundary Conditions

Not applicable — this feature has no user-input forms or numeric/string fields with valid ranges. All inputs are fixed, known repository facts (11 repos, 2 license values, 3 lifecycle/type enums), not open user input.

---

## Coverage Mapping

| AC ID | Description | Scenario(s) | AC Annotations |
|-------|-------------|-------------|-----------------|
| AC-1.1 | Catalog is the home view | 1.1 | [AC-1.1] on 1.1 |
| AC-1.2 | All 11 components appear | 1.1 | [AC-1.2] on 1.1 |
| AC-1.3 | proton excluded | 1.2 | [AC-1.3] on 1.2 |
| AC-1.4 | List entry shows key fields | 1.1 | [AC-1.4] on 1.1 |
| AC-2.1 | Detail view shows full metadata | 2.1 | [AC-2.1] on 2.1 |
| AC-2.2 | Metadata matches exploration.md | 2.1 | [AC-2.2] on 2.1 |
| AC-3.1 | Docs view renders README | 3.1 | [AC-3.1] on 3.1 |
| AC-3.2 | Docs reachable from detail page | 3.1 | [AC-3.2] on 3.1 |
| AC-4.1 | GitHub-wide discovery configured | 4.1 | [AC-4.1] on 4.1 |
| AC-4.2 | Repo without file not surfaced | 4.2 | [AC-4.2] on 4.2 |
| AC-4.3 | New repo appears without reconfig | 4.1 | [AC-4.3] on 4.1 |
| AC-5.1 | No login required | 5.1 | [AC-5.1] on 5.1 |
| AC-5.2 | Full read access on open | 5.1 | [AC-5.2] on 5.1 |
| AC-6.1 | User entity exists | 6.1 | [AC-6.1] on 6.1 |
| AC-6.2 | Components reference User entity | 6.1 | [AC-6.2] on 6.1 |
| AC-7.1 | Missing licenses added | 7.1 | [AC-7.1] on 7.1 |
| AC-7.2 | lattice standalone LICENSE | 7.2 | [AC-7.2] on 7.2 |
| AC-7.3 | gated relicensed to Apache-2.0 | 7.3 | [AC-7.3] on 7.3 |
| AC-7.4 | grease module path fixed | 7.4 | [AC-7.4] on 7.4 |
| AC-7.5 | serverless-to-ecs CI/CD added | 7.5 | [AC-7.5] on 7.5 |
| AC-7.6 | backstage LICENSE added | 7.6 | [AC-7.6] on 7.6 |
| AC-7.7 | Catalog reflects corrected license | 7.3 | [AC-7.7] on 7.3 |
| EC-1 | Malformed catalog-info.yaml | E-1 | [EC-1] on E-1 |
| EC-2 | Discovery finds repo with no file | 4.2 | [EC-2] on 4.2 |
| EC-3 | GitHub API access issue | E-2 | [EC-3] on E-2 |
| EC-4 | Repo deleted/renamed | E-3 | [EC-4] on E-3 |

## Out of Scope

- Registering `proton` — explicitly excluded per spec (not owned by the user).
- Deployment/hosting testing — instance is local-only per spec scope.
- Any Backstage plugin beyond Software Catalog and TechDocs — not built.
- Automated regression testing of Backstage's own stock UI — Backstage is a third-party product; its own UI is out of this feature's test responsibility. Testing here verifies *our* configuration and data (catalog-info.yaml content, discovery config, entity correctness), not Backstage's internals.

## Notes

- This feature is infrastructure/configuration across 11 independent git repositories, not a typical single-app UI feature — see `test-strategy.md` for how the standard 5-lane classification is adapted to that reality (no PostgreSQL, Socket.IO, or worker processes are involved anywhere in this feature).
