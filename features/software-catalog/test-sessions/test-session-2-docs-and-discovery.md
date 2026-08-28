# Test Session 2: Read docs and confirm new-repo auto-discovery

## Setup

- Backstage app running locally with GitHub discovery configured account-wide (per Story 1).
- A test repo (or a temporary throwaway repo) available under the owner's GitHub account to push a `catalog-info.yaml` to, without affecting any of the 10 real sibling repos' own registration stories.

## Scenarios (execution order)

### 1. README renders as TechDocs [PRIMARY] — Scenario 3.1 [AC-3.1, AC-3.2]
- **Act:** From any registered component's detail page (e.g. `lattice`), open the "Docs" tab.
- **Assert:** The repo's `README.md` content renders as a browsable documentation page.

### 2. New repo appears without reconfiguration [SEPARATE] — Scenario 4.1 [AC-4.1, AC-4.3]
- **Act:** Push a `catalog-info.yaml` to a repo's default branch that Backstage hasn't seen before, then wait for (or manually trigger) the next discovery refresh cycle.
- **Assert:** That repo's component appears in the catalog without any change to Backstage's own `app-config.yaml`.

## Cleanup

- Remove the throwaway `catalog-info.yaml` (or delete the temporary repo) used for scenario 2 so it doesn't linger in the catalog as noise.
