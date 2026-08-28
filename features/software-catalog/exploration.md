# Codebase Exploration: Software Catalog / Internal Developer Platform

**Generated:** 2026-08-28
**Feature:** Establish `backstage` as a catalog/IDP that records metadata about every repository the user owns, starting with the sibling repos found alongside it in `/home/murat/`.

## Scope Note

`/home/murat/backstage` is currently an empty, ungit'd directory containing only an empty `features/` folder — this is a greenfield project, not an existing app. There is no prior "similar feature" to look at inside this repo itself, so this exploration instead surveys the actual candidate data source: the sibling repositories under `/home/murat/` that the catalog needs to describe. A git repo and `feature/software-catalog` branch were created as part of this exploration so downstream pipeline artifacts (spec, stories, etc.) have somewhere to land.

## Sibling Repository Inventory

11 sibling directories were found next to `backstage` in `/home/murat/`. 10 are the user's own projects (all on GitHub under `muratcanozdemir`); one (`proton`) is a clone of Valve's upstream `ValveSoftware/Proton` and is **not** the user's software — excluded from the catalog below, but noted in case the user wants it tracked as an external dependency/reference repo.

All 10 owned repos share strong conventions (see Cross-Cutting Patterns) and were built very recently (commits clustered Aug 25–28, 2026), suggesting they're a coordinated batch of portfolio/experimental projects.

### bazel-maven-resolver
- **Purpose:** Bazel Bzlmod module extension (`maven_wildcard_deps`) that resolves wildcard Maven version globs from a YAML manifest against Maven Central, feeding pinned GAVs into `rules_jvm_external`.
- **Stack:** Starlark + Python 3 (inline scripts), YAML manifests. Bazel/Bzlmod build.
- **Type:** Library / build-system extension (consumed via `bazel_dep`, not standalone).
- **CI/CD:** `ci.yml` — buildifier format check, offline unit tests (required), live Maven Central integration test (non-blocking).
- **License:** None found.
- **Maturity:** v0.1.0, 4 commits, README documents known limitations honestly (non-hermetic network calls, minimal version comparator).

### codegraph-lsp
- **Purpose:** CLI (`codegraph`) that drives LSP servers (pyright, gopls) over JSON-RPC to extract call/reference graphs, builds a file-level coupling graph, and runs Leiden community detection to surface coupled modules — "zero parsing code."
- **Stack:** Python ≥3.12, `uv`/hatchling, deps `igraph` + `leidenalg`. Pure stdlib CLI (argparse).
- **Type:** CLI tool, pip/uv installable.
- **CI/CD:** `ci.yml` (ruff + pytest), `release.yml` (tag → build+publish GitHub Release), dependabot weekly.
- **License:** MIT.
- **Maturity:** v0.1.0, 4 commits. Has a `features/quality-hardening/exploration.md` — evidence it already uses an AI-agent feature-pipeline workflow (same shape as this one).

### gated
- **Purpose:** Supply-chain security "gate" daemon — intercepts OS-level package manager installs (pip/go/maven/cargo/npm), scans artifacts with syft/grype/osv-scanner, evaluates OPA/Rego policies (license, vuln, provenance), blocks non-compliant packages. No wrappers/symlinks — OS-level interception (fanotify/inotify/FSEvents).
- **Stack:** Go 1.25, embeds `open-policy-agent/opa` in-process. Shells out to syft/grype/osv-scanner at runtime.
- **Type:** CLI/daemon, single static binary, cross-platform (Windows watcher is a stub).
- **CI/CD:** `ci.yml` (gofmt/vet/staticcheck/race tests/multi-platform build), `release.yml`, dependabot weekly.
- **License:** Discrepancy — LICENSE file says MIT, README says Apache-2.0. Worth flagging to the user.
- **Maturity:** Pre-release, 8 commits, no tags yet despite README example showing `v0.3.0`. Also has a `features/quality-perf-devex-cicd/exploration.md`.

### github-probe (package name: org-harvest)
- **Purpose:** Downloads a complete, self-consistent snapshot of a GitHub organization (metadata, membership, teams, repos, issues, PRs, releases) via GraphQL, authenticating as a GitHub App installation; writes NDJSON/Parquet + run manifest.
- **Stack:** Python 3.12+, `uv`/hatchling. Deps: `httpx`, `pyjwt[crypto]`, `pyarrow`, `click`. `mypy --strict`.
- **Type:** CLI tool with a documented library API.
- **CI/CD:** `ci.yml` (ruff, format, mypy --strict, pytest; also reusable via `workflow_call`), `release.yml` (SBOM via CycloneDX, SHA-pinned actions). No dependabot config found.
- **License:** MIT.
- **Maturity:** v0.1.0. Full spec-driven pipeline present: `features/org-harvest/{spec.md, exploration.md, architecture.md, stories/}` with all 17 stories marked implemented as of 2026-08-28. Most mature/complete of the batch, feature-complete for v1 but no git tags cut.

### grease
- **Purpose:** CLI that turns a company domain + job posting + résumé into ready-to-send `.eml` files, one per selected contact — finds contacts (Hunter.io), drafts a personalized email via LLM, assembles MIME. Sends nothing itself (no OAuth/inbox access). Framed explicitly as a portfolio piece demonstrating disciplined LLM-boundary design.
- **Stack:** Go (module `github.com/user/grease` — note: module path uses placeholder `user`, not `muratcanozdemir`). Only stdlib at runtime; LLM backend is llama.cpp `/completion` with GBNF grammar.
- **Type:** Single-binary CLI, cross-compiled for 5 platforms.
- **CI/CD:** `ci.yaml` (gofmt/vet/race tests/staticcheck/govulncheck), `release.yaml` (cross-compile + SBOM + checksums). No dependabot.
- **License:** MIT.
- **Maturity:** Tagged v0.1.0, 4 commits — polished but small portfolio project.

### lattice
- **Purpose:** Thin, dependency-minimal Python library for semantic dataframe operations (LLM-backed structured extraction, semantic similarity joins) on top of Polars, against any OpenAI-compatible endpoint.
- **Stack:** Python ≥3.11, `uv`/hatchling. Deps: `polars`, `pydantic`, `httpx`, `numpy`, `usearch` (HNSW). `mypy --strict`.
- **Type:** Library/SDK (imported programmatically, not a CLI).
- **CI/CD:** `ci.yml` (mypy --strict, pytest, black --check), `release.yml` (SBOM + GitHub release), dependabot weekly.
- **License:** MIT (in pyproject.toml only, no standalone LICENSE file).
- **Maturity:** Tagged v0.1.0. README states "All 7 WBS items implemented, 44 tests, mypy --strict clean." Also has `features/codebase-hardening/exploration.md`.

### notecast-py
- **Purpose:** Local, offline note-taking engine using an LLM (via Ollama) to summarize notes, extract keywords, and build/evolve a knowledge graph (theme DAG). Python port of a TypeScript/Bun original.
- **Stack:** Python ≥3.12, `click`, `graphviz` (Python wrapper). System deps: Ollama + Graphviz binary.
- **Type:** CLI tool / single-user local app, SQLite storage, no server component.
- **CI/CD:** `ci.yml` (matrix Python 3.12/3.13, ruff, pytest+coverage), `release.yml` (tag-triggered, verifies pyproject version matches tag), dependabot weekly.
- **License:** MIT.
- **Maturity:** v0.1.0, 2 commits — earliest-stage of the batch.

### pr-drift
- **Purpose:** Unsupervised pull-request drift/anomaly detector — models a repo's historical diffs as a statistical baseline via zstd compression entropy (approximating Kolmogorov complexity), flags PRs that deviate significantly. Deliberately no embeddings/ML/training.
- **Stack:** Python, requires 3.14+ (uses new stdlib `compression.zstd`). Zero runtime deps.
- **Type:** Library + minimal HTTP service (`POST /ingest`, `POST /score`) + CLI/offline scripts, all in one package. `pr_drift.db` is a local, gitignored SQLite runtime artifact (table `pr_events`), not source.
- **CI/CD:** `ci.yml` (ruff + pytest on 3.14), `release.yml` (build + GitHub release), dependabot weekly.
- **License:** MIT.
- **Maturity:** v0.1.0, no tags, no CHANGELOG — alpha-stage, deliberately minimal by design.

### s3-vectors-at-home
- **Purpose:** Self-hosted, from-scratch clone of AWS S3 Vectors — vector database combining MinIO (S3-compatible storage) + FAISS (index) + FastAPI (HTTP API) + sentence-transformers (embeddings). README: "We have S3 vectors at home."
- **Stack:** Python ≥3.12, `uv`/hatchling. Deps: `fastapi`, `uvicorn`, `minio`, `faiss-cpu`, `sentence-transformers`, `torch` (CPU-only).
- **Type:** Dual-mode — API server (`uvicorn`) + CLI (`s3-vectors-cli`) + importable library (`vectorstore` package).
- **CI/CD:** `ci.yaml` — lint, test (spins up MinIO via docker compose), build, publish-to-PyPI on GitHub release.
- **License:** None found.
- **Maturity:** v0.1.0, no tags. README explicitly: "meant to be a PoC of sorts." Also has `features/core-hardening/exploration.md`.

### serverless-to-ecs
- **Purpose:** Go CLI that parses AWS serverless templates (CloudFormation/CDK/SAM), estimates current serverless vs. projected ECS Fargate costs, and generates Terraform stubs + a migration report for a strangler-pattern Lambda→ECS migration.
- **Stack:** Go 1.23, near-zero deps (`yaml.v3` only); optional LLM integration via plain `net/http`, no SDK.
- **Type:** Single-binary CLI / migration-analysis tool.
- **CI/CD:** **CORRECTION (2026-08-29, caught during Story 11 implementation):** this repo does have CI and release workflows (`.github/workflows/ci.yml`, `.github/workflows/release.yml`, added in commit `f760727`, with several dependabot action-bump PRs already merged on top). The original exploration below was wrong — verified by direct inspection of git history, not just current file listing. `git log --oneline -- .github/workflows/` shows the workflows were added well before this feature started. Do not add duplicate CI/CD to this repo; AC-7.5 in spec.md is voided by this correction.
  - ~~**None** — no `.github/workflows/` at all, only a dependabot config. No automated test/build/release pipeline, unlike every other repo in the batch.~~ (superseded, see correction above)
- **License:** None found. (Still accurate — re-verified 2026-08-29.)
- **Maturity:** Earliest-stage — 2 commits, no tags, no CHANGELOG. Has an empty `.claude/` directory (no configured skills).

## Cross-Cutting Patterns

- **Author/ownership:** Every repo (except `proton`) is owned by `muratcanozdemir` on GitHub and licensed to "Murat Can Ozdemir" where a license exists — a single-author portfolio of small, focused tools.
- **License inconsistency:** 7 of 10 repos declare MIT; `bazel-maven-resolver`, `s3-vectors-at-home`, and `serverless-to-ecs` have no LICENSE at all; `gated` has a MIT/Apache-2.0 mismatch between its LICENSE file and README. This is catalog-worthy metadata (license compliance is a common IDP concern) and worth flagging to the user directly.
- **Package manager conventions:** Python repos uniformly use `uv` + hatchling + `pyproject.toml` + `uv.lock`; Go repos uniformly use `go.mod` with minimal/no third-party deps. No JS/TS, no other languages present.
- **CI/CD is near-universal but not universal:** 9 of 10 repos have a `ci.yml`/`ci.yaml` (lint + test) and a tag-triggered `release.yml` that builds artifacts and often generates a CycloneDX SBOM. `serverless-to-ecs` is the one gap — no workflows at all. All repos that do have workflows pin GitHub Actions and most run dependabot weekly.
- **Version convention:** Every repo is at `0.1.0` (tagged or in manifest) — none have shipped a `0.2.0`+ yet. Commit counts are very low (2–8 commits each), consistent with recent bootstrap/hardening passes rather than long-lived history.
- **Existing feature-pipeline workflow:** 6 of the 10 repos (`codegraph-lsp`, `gated`, `github-probe`, `lattice`, `s3-vectors-at-home`, and implicitly this `backstage` repo itself) already contain a `features/<name>/exploration.md` artifact — meaning the same AI-agent feature pipeline (explore → spec → stories → implement) that's building this catalog has already been used standalone inside several of the target repos, mostly for one-off "hardening" or "quality" passes. `github-probe` is the only repo with a *complete* pipeline history (spec, architecture, 17 stories, all implemented). This is a strong signal for what "metadata to collect per repo" should include: not just catalog facts (language, license, CI status) but pipeline/process state (has this repo adopted the feature-pipeline workflow? how far did it get?).
- **LLM-adjacent theme:** Several repos (`grease`, `lattice`, `notecast-py`, `s3-vectors-at-home`) integrate with local or OpenAI-compatible LLM backends (llama.cpp, Ollama) rather than a specific vendor SDK — a recurring architectural preference worth surfacing as a catalog tag/facet if the user wants faceted search later.
- **No shared org/naming convention for module paths:** `grease`'s go.mod uses a placeholder module path (`github.com/user/grease`) instead of the real GitHub path used by `gated` and `serverless-to-ecs` — likely an oversight, worth flagging.

## Open Questions for Spec Stage

These weren't resolved by exploration and should be decided before implementation:

1. **What is "Backstage" here** — an actual instance of Spotify's open-source Backstage.io app (React/Node, `catalog-info.yaml` per repo, plugin ecosystem), or a lightweight custom catalog (e.g. a static site or simple DB) built from scratch under this same repo? The name and the sibling repos' tech stack (Go/Python, no Node/TS anywhere) suggest the latter may fit better, but this is a real product decision.
2. **What metadata schema per repo matters most** — the facts gathered above (purpose, stack, license, CI status, maturity) are a reasonable v1 set, but the user may want richer signals (LOC, test coverage, deploy status, last-touched date, "is this shipped/used" vs. "experiment").
3. **How is metadata kept fresh** — one-time import vs. a periodic re-scan (e.g. a scheduled job that re-reads each sibling repo, or ingests via each repo's own CI on push)?
4. **Does `proton` belong in the catalog** as a tracked external/reference dependency, or should the catalog only ever include repos the user authors?
5. **Where do future repos get registered** — auto-discovery of new sibling directories under `/home/murat/`, or explicit registration?
