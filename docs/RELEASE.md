# Phase 8 release preparation

Prepared for SmartPick-AI on **2026-09-22**. Phase 8 completes documentation and local release
checks for the current demonstration. The package remains `0.1.0.dev0`; no versioned tag,
GitHub release, or production deployment is created by this phase push.

## Release-note draft

SmartPick-AI demonstrates evidence-grounded product search and comparison, using smartphones
as its first validated catalogue category:

- Audited 3,062-record catalogue with stable IDs, explicit missing values, and source links.
- BM25, pinned MiniLM semantic, and hybrid retrieval with deterministic strict filters.
- PostgreSQL/pgvector evidence storage and a bounded LangGraph workflow with verification.
- FastAPI endpoints and a Streamlit Search / Ask / Compare interface.
- Optional OpenAI provider; ordinary catalogue search requires no paid LLM calls.
- Docker Compose, reviewed evaluation cases, and setup, demo, results, and interview guides.

The September 18 SmartPick-AI run reproduced the 20-case retrieval benchmark and passed all
16 scripted agent cases. These are development measurements, not live-model accuracy estimates.
See [results and conditions](RESULTS.md).

## Phase 8 validation

On September 22, the isolated SmartPick-AI copy passed **324 tests, with 6 optional integrations
skipped, in 11.50 seconds**. Skips cover four live HTTP cases, one live-provider test, and one
PostgreSQL test. Ruff lint, formatting for 61 Python files, dependency consistency, and Compose
configuration validation passed. One existing Starlette/AnyIO deprecation warning remains.

The release check passed across 74 tracked/non-ignored candidate files: local file links resolved
and no recognized credential, generated/environment path, oversized, or non-UTF-8 findings were
reported. This is a bounded check, not a comprehensive security audit.

The release checker examines tracked and non-ignored candidate files for unpublished local link
targets, generated/environment paths, files above 2 MB, non-UTF-8 content, and several recognizable
credential/private-key formats. It never prints matched credential values.
It does not check remote URLs, heading anchors, Git history, or arbitrary secret formats.
Run it with `python scripts/check_release.py`.

No new retrieval benchmark, live-provider request, container build, database ingestion,
fresh-machine installation, or production deployment was performed in Phase 8. The application
implementation is unchanged from SmartPick-AI Phase 7.

## Remaining versioned-release decisions

Before a versioned release, select the intended code license, decide whether to change both package
version declarations to `0.1.0`, verify the final commit and version metadata, and explicitly approve
the tag and GitHub release. These are separate from publishing the Phase 8 documentation branch.

Raw/processed datasets, indexes, model caches, credentials, and database volumes are excluded
from release assets. The dataset's conflicting usage notes are documented in
[the catalogue audit](MOBILE_CATALOGUE_AUDIT.md); its license label does not license this code.
No code license has been selected. See [setup](SETUP.md) for separately obtained data.
