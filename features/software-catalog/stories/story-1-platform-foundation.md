# Story 1: Platform foundation — open Backstage and see yourself in the catalog

**Status:** APPROVED
**Depends On:** None
**UI Changes:** No

## User Story

As the owner, I can open a running Backstage instance with no login step and see the `backstage` repo itself registered in the catalog, so the platform exists and is ready for every other repo to register into.

## Acceptance Criteria

- AC-1.1: Opening the Backstage app shows the Software Catalog as the primary/home view.
- AC-5.1: Opening the Backstage app requires no login/credentials entry.
- AC-5.2: The owner has full read access to the catalog and docs immediately on open.
- AC-6.1: A Backstage User entity exists representing the owner.
- AC-4.1: Backstage is configured to discover catalog entities across the owner's entire GitHub account, not just an explicit list of known repos.
- AC-4.2: A repository only appears in the catalog once it contains a `catalog-info.yaml` file; repos without one are not surfaced.
- AC-1.2 (partial — `backstage` only): The `backstage` repo appears as a Component in the catalog.
- AC-7.6: The `backstage` repo has a LICENSE file (MIT).
- AC-2.1 (partial — `backstage` only): The `backstage` component's detail view shows description, stack, type, license, owner, lifecycle, and a GitHub link.
- AC-3.1 (partial — `backstage` only): The `backstage` component has a Docs view rendering this repo's README.

## Scope

**Included:**
- Scaffolding the Backstage app into this repo.
- Guest access configuration (no login).
- GitHub-account-wide discovery configuration (entity provider), even though it will find nothing beyond `backstage` until later stories land.
- Local TechDocs configuration.
- The owner's User entity, defined once for all components to reference.
- `backstage`'s own `catalog-info.yaml` and a MIT LICENSE file for this repo.

**Excluded:**
- Registering any of the 10 external sibling repos — one story each (Stories 2–11).
- Any repository fixes in external repos (Stories 2–11).
- Grouping components under a System entity (out of scope per spec).

## Entry Criteria (Functional)

Before starting this story, ensure:
- `features/software-catalog/spec.md` is APPROVED.
- `features/software-catalog/architecture.md` exists and its Pattern Reference (Component/User entity templates, type mapping table) is available to follow.

## Exit Criteria (Functional)

This story is complete when:
- All acceptance criteria above are verified.
- Quality checks pass (lint, typecheck if applicable).
- Code reviewed.
