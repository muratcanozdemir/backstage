# Test Strategy: Software Catalog

**Status:** APPROVED
**Created:** 2026-08-29
**Spec:** [features/software-catalog/spec.md](spec.md)
**Scenarios:** [features/software-catalog/test-scenarios.md](test-scenarios.md)
**Test Sessions Required:** Yes
**E2E Required:** No

## Feature Overview

A real Backstage.io instance catalogs 11 owned repos with GitHub-wide discovery, guest access, and TechDocs, plus source-level fixes to 4 repos.

## Lane Adaptation Note

The standard 5-lane classification (Unit / Handler Integration / Real-Worker Integration / UI E2E / Topology E2E) assumes an application with PostgreSQL, Socket.IO, and a worker child process — none of which exist in this feature or this repo. This feature is infrastructure/configuration spanning 11 independent git repos (a Backstage app + 10 unrelated Go/Python/Bazel projects), so the lanes are mapped to what actually catches failures here:

| Standard lane | Adapted meaning for this feature |
|---|---|
| Unit | **Manifest/Config Validation** — static checks on `catalog-info.yaml` (valid YAML, required fields present, correct entity refs) and on fix content (LICENSE text, go.mod content) without running the app |
| Handler Integration | **Repo Build/CI Check** — does the affected repo still build/lint/test after a change (e.g. `go build` after `grease`'s module-path fix, the new `serverless-to-ecs` CI workflow actually passing) |
| Real-Worker Integration | Not applicable — no worker-backed execution exists anywhere in this feature |
| UI E2E | **Manual Browser Verification (Test Session)** — this repo has no existing Playwright/E2E harness for a Backstage instance, and building one is out of spec scope; verification of catalog/docs rendering happens via a manual test session against the real running app instead |
| Topology E2E | Not applicable — no worker or multi-process topology exists anywhere in this feature |

## Classified Scenarios

### US-1: Browse the catalog

#### Scenario 1.1: Full catalog list on open [AC-1.1, AC-1.2, AC-1.4]

##### Manifest/Config Validation
**Classification:** Covered
**Justification:** Each `catalog-info.yaml` can be validated for correct `kind: Component`, required `metadata`/`spec` fields, and correct `metadata.name` before ever starting the app — catches malformed entries at the source.
**Test approach:** Parse and validate each of the 11 `catalog-info.yaml` files against the Component schema fields listed in `architecture.md`'s Pattern Reference.

##### Repo Build/CI Check
**Classification:** Not applicable
**Justification:** No build step produces the catalog list — it's config-driven, not compiled.

##### Manual Browser Verification (Test Session)
**Classification:** Covered
**Justification:** Only opening the real running Backstage app proves discovery actually picked up all 11 entities and rendered them together — config validation alone can't catch a discovery misconfiguration.
**BDD Scenario:**
**Given** all 11 `catalog-info.yaml` files are present and Backstage is running
**When** the owner opens the app
**Then** the catalog home view lists all 11 components with name, description, owner, and lifecycle

---

#### Scenario 1.2: proton is never cataloged [AC-1.3]

##### Manifest/Config Validation
**Classification:** Covered
**Justification:** Confirms no `catalog-info.yaml` was ever added to `/home/murat/proton`.
**Test approach:** Check `proton` for absence of a `catalog-info.yaml` file.

##### Manual Browser Verification (Test Session)
**Classification:** Covered
**Justification:** Confirms discovery genuinely doesn't surface it, not just that no file was intentionally added.
**BDD Scenario:**
**Given** `proton` has no `catalog-info.yaml`
**When** the owner searches the catalog for "proton"
**Then** no result appears

---

### US-2: View a component's full metadata

#### Scenario 2.1: Component detail view shows accurate metadata [AC-2.1, AC-2.2]

##### Manifest/Config Validation
**Classification:** Covered
**Justification:** The catalog can only display what's in the file — verifying `catalog-info.yaml` field values against `exploration.md` catches data-entry mistakes before they ever render.
**Test approach:** Diff each `catalog-info.yaml`'s description/type/tags against the corresponding section of `exploration.md`.

##### Manual Browser Verification (Test Session)
**Classification:** Covered
**Justification:** Proves the values actually render correctly in the UI (e.g. license/owner resolve to the right linked entities), which static file review can't confirm.
**BDD Scenario:**
**Given** `gated`'s component entity is registered
**When** the owner opens its detail page
**Then** description, stack, type, license, owner, lifecycle, and GitHub link all display and match `exploration.md`

---

### US-3: Read a repository's documentation from the catalog

#### Scenario 3.1: README renders as TechDocs [AC-3.1, AC-3.2]

##### Manifest/Config Validation
**Classification:** Covered
**Justification:** TechDocs annotation/config presence in `catalog-info.yaml` can be checked statically.
**Test approach:** Confirm each `catalog-info.yaml` carries the TechDocs-enabling annotation per the app's `techdocs` config.

##### Manual Browser Verification (Test Session)
**Classification:** Covered
**Justification:** Only the running app proves the local TechDocs builder actually generates and serves the docs page from the README — a missing annotation and a broken build look identical from the file alone.
**BDD Scenario:**
**Given** a component has TechDocs enabled
**When** the owner opens its Docs tab
**Then** the README content renders as a docs page

---

### US-4: New repositories appear automatically

#### Scenario 4.1: New catalog-info.yaml is discovered without reconfiguration [AC-4.1, AC-4.3]

##### Manifest/Config Validation
**Classification:** Covered
**Justification:** The discovery provider's target pattern (`https://github.com/muratcanozdemir/*`) can be checked statically against `app-config.yaml`.
**Test approach:** Confirm `catalog.providers.github` config targets the whole account, not an explicit repo list.

##### Manual Browser Verification (Test Session)
**Classification:** Covered
**Justification:** Only a real discovery cycle against the live GitHub API proves auto-discovery actually works end-to-end (auth, pattern matching, refresh) — config review alone can't catch a token or pattern-scoping mistake.
**BDD Scenario:**
**Given** GitHub discovery is configured and running
**And** a new `catalog-info.yaml` is pushed to a repo's default branch
**When** the next discovery refresh runs
**Then** that repo's component appears in the catalog without touching Backstage's own config

---

#### Scenario 4.2: Repo without catalog-info.yaml stays absent [AC-4.2]

##### Manual Browser Verification (Test Session)
**Classification:** Covered
**Justification:** Confirms discovery's default (ignore, don't error) behavior for the common case of an unregistered repo.
**BDD Scenario:**
**Given** a repo in the account has no `catalog-info.yaml`
**When** the discovery refresh runs
**Then** that repo does not appear and no error is shown

---

### US-5: Open the catalog without logging in

#### Scenario 5.1: Guest access lands directly in the catalog [AC-5.1, AC-5.2]

##### Manifest/Config Validation
**Classification:** Covered
**Justification:** Guest provider config (`auth.providers.guest`, resolver wiring) can be checked statically.
**Test approach:** Confirm `app-config.yaml` configures guest auth as the sign-in method.

##### Manual Browser Verification (Test Session)
**Classification:** Covered
**Justification:** Only opening the app proves no login screen actually appears — a misconfigured resolver can silently fall back to a sign-in prompt.
**BDD Scenario:**
**Given** the app is running with guest auth configured
**When** the owner opens the app
**Then** the catalog is immediately visible with no credential prompt

---

### US-6: See accurate ownership on every component

#### Scenario 6.1: Every component references the owner's User entity [AC-6.1, AC-6.2]

##### Manifest/Config Validation
**Classification:** Covered
**Justification:** Every component's `spec.owner` value can be checked statically against the expected entity ref format (`user:default/muratcanozdemir`), catching plain-string owners before they ever render.
**Test approach:** Grep all 11 `catalog-info.yaml` files for `owner: user:default/muratcanozdemir`.

##### Manual Browser Verification (Test Session)
**Classification:** Covered
**Justification:** Confirms the reference actually resolves to a real, clickable User profile in the UI rather than showing as a broken-entity-ref warning.
**BDD Scenario:**
**Given** the `muratcanozdemir` User entity exists
**When** the owner opens any component's detail page
**Then** the owner field links to that User's profile page

---

### US-7: Trust the metadata the catalog reports

#### Scenario 7.1: Missing-license repos gain a LICENSE file [AC-7.1]

##### Manifest/Config Validation
**Classification:** Covered
**Justification:** Presence and content of the LICENSE file can be checked without running anything.
**Test approach:** Confirm `bazel-maven-resolver`, `s3-vectors-at-home`, and `serverless-to-ecs` each have a LICENSE file with standard MIT text and "Murat Can Ozdemir" as copyright holder.

---

#### Scenario 7.2: lattice gains a standalone LICENSE file [AC-7.2]

##### Manifest/Config Validation
**Classification:** Covered
**Justification:** Same as 7.1 — file presence/content check.
**Test approach:** Confirm `lattice` has a standalone MIT LICENSE file matching `pyproject.toml`'s declaration.

---

#### Scenario 7.3: gated's license mismatch is resolved to Apache-2.0 [AC-7.3, AC-7.7]

##### Manifest/Config Validation
**Classification:** Covered
**Justification:** LICENSE file content and README claim can both be checked statically for agreement.
**Test approach:** Confirm `gated`'s LICENSE file is Apache-2.0 text and its `catalog-info.yaml` license metadata says Apache-2.0.

##### Manual Browser Verification (Test Session)
**Classification:** Covered
**Justification:** Confirms the corrected license actually renders on `gated`'s catalog detail page (Scenario 2.1 already covers rendering generally; this specifically re-checks the corrected value post-fix).
**BDD Scenario:**
**Given** `gated`'s license has been corrected to Apache-2.0
**When** the owner opens `gated`'s detail page
**Then** the license field shows Apache-2.0

---

#### Scenario 7.4: grease's module path is corrected [AC-7.4]

##### Manifest/Config Validation
**Classification:** Covered
**Justification:** `go.mod`'s module path and import statements can be checked statically for the corrected path.
**Test approach:** Confirm `go.mod` declares `github.com/muratcanozdemir/grease` and no source file still imports the old placeholder path.

##### Repo Build/CI Check
**Classification:** Covered
**Justification:** Only actually building the module proves the import-path rewrite was complete and consistent — a missed import elsewhere would only surface as a build failure.
**Test approach:** Run `go build ./...` in `grease` after the fix and confirm it succeeds.

---

#### Scenario 7.5: serverless-to-ecs gains CI/CD parity [AC-7.5]

##### Manifest/Config Validation
**Classification:** Covered
**Justification:** Workflow file presence and structure (jobs for lint/vet/test, a release job) can be checked statically, and existing dependabot config confirmed unchanged.
**Test approach:** Confirm `.github/workflows/ci.yml` and `.github/workflows/release.yml` exist with the same job shape as `gated`/`grease`, and `.github/dependabot.yml` is unmodified.

##### Repo Build/CI Check
**Classification:** Covered
**Justification:** Only actually running the new CI workflow against the repo's existing code proves it passes — a workflow file that looks right can still fail on the repo's actual test/lint output.
**Test approach:** Run the new CI workflow's steps (or equivalent local commands: `go vet ./...`, `go build ./...`, `gofmt -l .`) locally against `serverless-to-ecs` and confirm success.

---

#### Scenario 7.6: backstage repo gains a LICENSE file [AC-7.6]

##### Manifest/Config Validation
**Classification:** Covered
**Justification:** File presence/content check, same as 7.1.
**Test approach:** Confirm `backstage` has a MIT LICENSE file.

---

## Error Scenarios

### Scenario E-1: Malformed catalog-info.yaml [EC-1]

##### Manifest/Config Validation
**Classification:** Not applicable
**Justification:** This scenario is specifically about how the running app handles a bad file, not about the file being correct — can't be caught by validating a file that (in this scenario) is deliberately invalid.

##### Manual Browser Verification (Test Session)
**Classification:** Covered
**Justification:** Only the running app's own error handling and rendering can prove one bad entity doesn't take down the rest of the catalog.
**BDD Scenario:**
**Given** one component's `catalog-info.yaml` has invalid YAML
**When** Backstage processes the catalog
**Then** that entity shows a processing error and every other entity still loads normally

---

### Scenario E-2: GitHub API access issue during discovery [EC-3]

##### Manual Browser Verification (Test Session)
**Classification:** Covered
**Justification:** This is inherent Backstage discovery-provider behavior (documented, not something this feature implements) — worth a one-time sanity check (e.g. temporarily using an invalid token) rather than assuming it, since it directly affects catalog reliability.
**BDD Scenario:**
**Given** the discovery provider's GitHub token is temporarily invalid
**When** a discovery refresh cycle runs
**Then** the catalog keeps showing its previously discovered entities rather than going empty

---

### Scenario E-3: A cataloged repository disappears from GitHub [EC-4]

##### Manual Browser Verification (Test Session)
**Classification:** Not applicable — REDIRECTED
**Justification:** This is documented, out-of-the-box Backstage orphan-entity behavior with no custom logic in this feature; actively testing it would mean deleting a real owned repo, which is disproportionate to the value of confirming stock framework behavior. Trust the Backstage documentation cited in `spec.md`'s Edge Cases instead.

---

## Test Sessions Required / E2E Required

- **Test Sessions Required: Yes** — most scenarios above need the real running app (discovery, rendering, auth, TechDocs) to be genuinely verified; static file checks alone can't catch misconfiguration.
- **E2E Required: No** — no automated Playwright/browser-E2E harness exists for this Backstage instance and building one is out of spec scope (spec explicitly limits this feature to Software Catalog + TechDocs, local-only). Manual test sessions substitute for automated E2E here.

## Classification Summary

| Lane | Scenario Count | Scenarios |
|------|---------------|-----------|
| Manifest/Config Validation | 12 | 1.1, 1.2, 2.1, 3.1, 4.1, 5.1, 6.1, 7.1, 7.2, 7.3, 7.4, 7.5, 7.6 |
| Repo Build/CI Check | 2 | 7.4, 7.5 |
| Real-Worker Integration | 0 | Not applicable to this feature |
| Manual Browser Verification (Test Session) | 12 | 1.1, 1.2, 2.1, 3.1, 4.1, 4.2, 5.1, 6.1, 7.3, E-1, E-2 |
| Topology E2E | 0 | Not applicable to this feature |
| REDIRECTED | 1 | E-3 (to: trust documented Backstage behavior, not independently tested) |

## Coverage Mapping

| AC ID | Description | Manifest/Config Validation | Repo Build/CI Check | Manual Browser Verification | AC Annotations |
|-------|-------------|------|------|--------|----------------|
| AC-1.1 | Catalog is home view | - | - | 1.1 | [AC-1.1] on 1.1 |
| AC-1.2 | All 11 components appear | 1.1 | - | 1.1 | [AC-1.2] on 1.1 |
| AC-1.3 | proton excluded | 1.2 | - | 1.2 | [AC-1.3] on 1.2 |
| AC-1.4 | List entry key fields | 1.1 | - | 1.1 | [AC-1.4] on 1.1 |
| AC-2.1 | Detail view full metadata | - | - | 2.1 | [AC-2.1] on 2.1 |
| AC-2.2 | Metadata matches exploration.md | 2.1 | - | 2.1 | [AC-2.2] on 2.1 |
| AC-3.1 | Docs renders README | 3.1 | - | 3.1 | [AC-3.1] on 3.1 |
| AC-3.2 | Docs reachable from detail | - | - | 3.1 | [AC-3.2] on 3.1 |
| AC-4.1 | Discovery configured account-wide | 4.1 | - | 4.1 | [AC-4.1] on 4.1 |
| AC-4.2 | Repo w/o file not surfaced | - | - | 4.2 | [AC-4.2] on 4.2 |
| AC-4.3 | New repo appears w/o reconfig | - | - | 4.1 | [AC-4.3] on 4.1 |
| AC-5.1 | No login required | 5.1 | - | 5.1 | [AC-5.1] on 5.1 |
| AC-5.2 | Full read access on open | - | - | 5.1 | [AC-5.2] on 5.1 |
| AC-6.1 | User entity exists | 6.1 | - | 6.1 | [AC-6.1] on 6.1 |
| AC-6.2 | Components reference User entity | 6.1 | - | 6.1 | [AC-6.2] on 6.1 |
| AC-7.1 | Missing licenses added | 7.1 | - | - | [AC-7.1] on 7.1 |
| AC-7.2 | lattice standalone LICENSE | 7.2 | - | - | [AC-7.2] on 7.2 |
| AC-7.3 | gated relicensed | 7.3 | - | 7.3 | [AC-7.3] on 7.3 |
| AC-7.4 | grease module path fixed | 7.4 | 7.4 | - | [AC-7.4] on 7.4 |
| AC-7.5 | serverless-to-ecs CI/CD added | 7.5 | 7.5 | - | [AC-7.5] on 7.5 |
| AC-7.6 | backstage LICENSE added | 7.6 | - | - | [AC-7.6] on 7.6 |
| AC-7.7 | Catalog reflects corrected license | - | - | 7.3 | [AC-7.7] on 7.3 |
| EC-1 | Malformed catalog-info.yaml | - | - | E-1 | [EC-1] on E-1 |
| EC-2 | Discovery finds repo w/o file | - | - | 4.2 | [EC-2] on 4.2 |
| EC-3 | GitHub API access issue | - | - | E-2 | [EC-3] on E-2 |
| EC-4 | Repo deleted/renamed | - | - | REDIRECTED | [EC-4] on E-3 |

## Out of Scope

- Automated Playwright/browser E2E for Backstage — no harness exists and building one is outside spec scope.
- Load/scale testing — 11 entities is trivial scale, not worth dedicated testing per spec's Non-Functional Hints.
- Testing Backstage's own framework internals (auth flow security, TechDocs builder correctness) — third-party product code, not this feature's responsibility.

## Notes

- "Manifest/Config Validation" checks in this strategy are static file/content checks performed during implementation and review, not a standing automated test suite — there's no existing unit-test harness for YAML/config repos in this project, and building one for 11 near-identical files would be disproportionate. Each story's `/review-story` pass re-verifies these checks against the actual committed files.
