# Story 5: Register github-probe in the catalog

**Status:** APPROVED
**Depends On:** 1
**UI Changes:** No

## User Story

As the owner, I can see `github-probe` in the Backstage catalog with accurate metadata, so I can find and understand this project without opening the repo.

## Acceptance Criteria

- AC-1.2: `github-probe` appears as a Component in the catalog, named after its GitHub repo name (`github-probe`), not its internal package name (`org-harvest`) — per architecture.md's naming constraint.
- AC-1.4: Its catalog list entry shows name, description/purpose, owner, and lifecycle stage.
- AC-2.1: Its detail view shows description, tech stack, project type (`tool`), license, owner, lifecycle, and a link to the GitHub repo.
- AC-2.2: The metadata shown matches `features/software-catalog/exploration.md`'s findings for this repo, including a mention of its package name (`org-harvest`) in the description/title.
- AC-3.1: A Docs view renders this repo's README.md.
- AC-3.2: The docs view is reachable from the component's detail page.
- AC-4.3: The component appears via GitHub discovery once its `catalog-info.yaml` is on the repo's default branch — no manual Backstage reconfiguration.
- AC-6.2: The component's `owner` references the User entity from Story 1.

## Scope

**Included:**
- Adding `catalog-info.yaml` to the `github-probe` repo.
- A `feature/software-catalog` branch and commit in that repo (per architecture.md: feature branch, no PR/push).

**Excluded:**
- Any repository fix — this repo has no flagged issues from exploration.
- Renaming the repo or its package — the name mismatch is documented, not corrected.
- Pushing the branch or opening a PR.

## Entry Criteria (Functional)

Before starting this story, ensure:
- Story 1's acceptance criteria are met: the Backstage platform is running, guest access works, GitHub discovery is configured, and the owner's User entity exists.

## Exit Criteria (Functional)

This story is complete when:
- All acceptance criteria above are verified.
- Quality checks pass.
- Code reviewed.
