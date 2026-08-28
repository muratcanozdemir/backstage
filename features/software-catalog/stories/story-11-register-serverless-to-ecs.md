# Story 11: Register serverless-to-ecs in the catalog

**Status:** APPROVED
**Depends On:** 1
**UI Changes:** No

## User Story

As the owner, I can see `serverless-to-ecs` in the Backstage catalog with accurate metadata, a real license, and CI/CD parity with my other repos, so the catalog doesn't just report that this repo has no automated checks — it no longer has that gap.

## Acceptance Criteria

- AC-1.2: `serverless-to-ecs` appears as a Component in the catalog.
- AC-1.4: Its catalog list entry shows name, description/purpose, owner, and lifecycle stage.
- AC-2.1: Its detail view shows description, tech stack, project type (`tool`), license, owner, lifecycle, and a link to the GitHub repo.
- AC-2.2: The metadata shown matches `features/software-catalog/exploration.md`'s findings for this repo.
- AC-3.1: A Docs view renders this repo's README.md.
- AC-3.2: The docs view is reachable from the component's detail page.
- AC-4.3: The component appears via GitHub discovery once its `catalog-info.yaml` is on the repo's default branch — no manual Backstage reconfiguration.
- AC-6.2: The component's `owner` references the User entity from Story 1.
- AC-7.1: The repo has a LICENSE file (MIT), matching the convention used by the rest of the owner's repos.
- AC-7.5: The repo has CI (lint/vet/test) and release (tag-triggered build + GitHub release) workflows added, matching the pattern used by the owner's other Go repos, and keeps its existing dependabot configuration.

## Scope

**Included:**
- Adding `catalog-info.yaml` to the `serverless-to-ecs` repo.
- Adding a MIT LICENSE file.
- Adding CI and release GitHub Actions workflows matching the pattern used in `gated`/`grease`.
- A `feature/software-catalog` branch and commit in that repo (per architecture.md: feature branch, no PR/push).

**Excluded:**
- Any change to the repo's application code or Terraform-generation logic.
- Pushing the branch or opening a PR.

## Entry Criteria (Functional)

Before starting this story, ensure:
- Story 1's acceptance criteria are met: the Backstage platform is running, guest access works, GitHub discovery is configured, and the owner's User entity exists.

## Exit Criteria (Functional)

This story is complete when:
- All acceptance criteria above are verified.
- The new CI workflow passes against the repo's existing code.
- Quality checks pass.
- Code reviewed.
