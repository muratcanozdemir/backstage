# Story 4: Register gated in the catalog

**Status:** APPROVED
**Depends On:** 1
**UI Changes:** No

## User Story

As the owner, I can see `gated` in the Backstage catalog with accurate metadata and a resolved license, so I can find and understand this project without opening the repo, and without the catalog reporting license info that contradicts the repo's own README.

## Acceptance Criteria

- AC-1.2: `gated` appears as a Component in the catalog.
- AC-1.4: Its catalog list entry shows name, description/purpose, owner, and lifecycle stage.
- AC-2.1: Its detail view shows description, tech stack, project type (`service`), license, owner, lifecycle, and a link to the GitHub repo.
- AC-2.2: The metadata shown matches `features/software-catalog/exploration.md`'s findings for this repo.
- AC-3.1: A Docs view renders this repo's README.md.
- AC-3.2: The docs view is reachable from the component's detail page.
- AC-4.3: The component appears via GitHub discovery once its `catalog-info.yaml` is on the repo's default branch — no manual Backstage reconfiguration.
- AC-6.2: The component's `owner` references the User entity from Story 1.
- AC-7.3: The repo's LICENSE file is changed to Apache-2.0, resolving the mismatch with its README (which already claimed Apache-2.0).
- AC-7.7: The catalog entry's license field reflects Apache-2.0, not the prior MIT.

## Scope

**Included:**
- Adding `catalog-info.yaml` to the `gated` repo, with license metadata reflecting Apache-2.0.
- Replacing the repo's LICENSE file content with Apache-2.0.
- A `feature/software-catalog` branch and commit in that repo (per architecture.md: feature branch, no PR/push).

**Excluded:**
- Any change to the repo's actual code or CI/CD workflows.
- Pushing the branch or opening a PR.

## Entry Criteria (Functional)

Before starting this story, ensure:
- Story 1's acceptance criteria are met: the Backstage platform is running, guest access works, GitHub discovery is configured, and the owner's User entity exists.

## Exit Criteria (Functional)

This story is complete when:
- All acceptance criteria above are verified.
- Quality checks pass.
- Code reviewed.
