# Test Session 1: Open the catalog as a guest and inspect a component

## Setup

- Backstage app running locally with all 11 `catalog-info.yaml` files merged and discovery having completed at least one refresh cycle.
- Owner's User entity (`user:default/muratcanozdemir`) present in the catalog.
- Browser with no prior Backstage session/cookies (to genuinely test the no-login path).

## Scenarios (execution order)

### 1. Guest access, no login [PRIMARY] — Scenario 5.1 [AC-5.1, AC-5.2]
- **Act:** Open the Backstage app's URL in a fresh browser session.
- **Assert:** No login/credential prompt appears; the Software Catalog is immediately visible.

### 2. Full catalog list [PIGGYBACK] — Scenario 1.1 [AC-1.1, AC-1.2, AC-1.4]
- **Act:** Stay on the catalog home view.
- **Assert:** All 11 components (10 sibling repos + `backstage`) are listed, each showing name, description, owner, and lifecycle stage.

### 3. proton excluded [PIGGYBACK] — Scenario 1.2 [AC-1.3]
- **Act:** Search/filter the catalog for "proton".
- **Assert:** No result appears.

### 4. Component detail view [PIGGYBACK] — Scenario 2.1 [AC-2.1, AC-2.2]
- **Act:** Click into the `gated` component.
- **Assert:** Detail page shows description, tech stack, type (`service`), license (Apache-2.0), owner, lifecycle (`experimental`), and a GitHub link — matching `exploration.md`'s facts for `gated`.

### 5. Owner entity link [PIGGYBACK] — Scenario 6.1 [AC-6.1, AC-6.2]
- **Act:** From `gated`'s detail page, click the owner field.
- **Assert:** Navigates to a real User profile page for `muratcanozdemir`, not a dead link or plain text.

## Cleanup

- None — read-only browsing session, no state changes.
