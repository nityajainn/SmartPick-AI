# Build Log

This log records SmartPick-AI work when it occurs. Planned work belongs in the roadmap.

## 2026-09-08 — Phase 0: project foundation

Status: complete and published

- Established SmartPick-AI with a fresh repository history.
- Added the Python package boundary, environment configuration, shared logging, documentation,
  smoke tests, and ignore rules for secrets and generated data.
- Generalized the project direction to evidence-grounded product catalogue search and comparison.
- Retained the internal `searchrank_ai` package name and documented the adapted foundation.

Validation recorded for the published Phase 0 snapshot: 3 tests passed; Ruff lint and format checks
passed for 10 files.

## 2026-09-10 — Phase 1: first catalogue data pipeline

Status: complete and published

- Added deterministic, evidence-preserving dataset audit utilities and synthetic regression tests.
- Retained the rejected laptop and Amazon-phone audits as decision evidence.
- Established smartphones as the first implemented catalogue category while keeping the overall
  project direction applicable to separately validated product categories.
- Added the adopted 91mobiles pipeline with source-schema validation, explicit parsing, stable IDs,
  eligibility reasons, full raw-value audit records, and overwrite protection.
- Added the documented 18-field smartphone catalogue schema and reproduction path.
- Kept raw data, processed catalogues, and generated audit artifacts outside Git.

The imported source audit record reports 4,000 source rows, 3,062 retained smartphones, 4,000
unique source URLs and derived IDs, and a reproducible catalogue SHA-256 of
`c3428dd04e5d02c6a66ce00f5961f6b1f6ba9dcc30d86a37ffde84523619c7ea`. These measurements were
produced by the original full-data audit and were not rerun during publication because the dataset
is intentionally excluded from Git.

Validation run on the prepared SmartPick-AI Phase 1 snapshot on 2026-09-10:

- `python -m pytest -q`: 142 passed in 2.05 seconds.
- Ruff lint: passed.
- Ruff format check: 22 files already formatted.
- Git whitespace check: passed.

No retrieval implementation, database, agent workflow, API, interface, or Phase 2 work was added.

## 2026-09-13 — Phase 2: BM25 keyword retrieval

- Adapted the original Phase 2 BM25 implementation, reviewed judgments, and full query-coverage
  tests into SmartPick-AI's independent history.
- Added versioned local index building, keyword search, deterministic scores and ties, provenance,
  schema validation, and reproducible binary-relevance evaluation.
- Updated the README and architecture while retaining the general product-search direction.
  The implemented schema and benchmark still use the first smartphone catalogue.
- Ran the suite against this isolated preparation copy: 162 passed in 2.68 seconds.
- Ruff lint passed; formatting passed for 25 files.
- Built indexes twice from the unchanged local 3,062-product catalogue, then evaluated each against
  the 12 reviewed exact-model queries. Indexes and evaluation reports were byte-identical.
- Recall@10, MRR@10, and NDCG@10 were each 1.000. These results apply only to the reviewed
  exact-model queries, not semantic relevance, numeric constraints, or general shopping needs.
- Dataset and derived artifacts remain local and ignored. Project 1 was used only as a read-only
  source; no later-phase code was included.

## 2026-09-14 — Phase 3: semantic, hybrid, and constrained retrieval

- Adapted the original Phase 3 implementation, tests, reviewed judgments, and design report into
  SmartPick-AI's independent history, retaining the general product-search presentation.
- Added pinned MiniLM encoding, validated local vector artifacts, inspectable hybrid score
  components, and deterministic pre-ranking price, RAM, storage, rating, and brand filters.
- Ran the isolated preparation copy's suite: 182 tests passed in 1.90 seconds. Ruff lint,
  formatting for 29 files, and dependency consistency checks passed.
- Ran the 20-case benchmark offline on CPU using cached model weights and existing validated
  catalogue/BM25/semantic artifacts. No semantic index rebuild or new build timing is claimed.
- Reproduced selection of alpha 0.25: Recall@10 0.925000, MRR@10 0.950000,
  NDCG@10 0.928558, constraint satisfaction 1.000000.
- Kept generated data, embeddings, model weights, and evaluation reports local and ignored.
  No Phase 4 database implementation, agent workflow, API, or interface was included.

## 2026-09-15 — Phase 4: PostgreSQL and pgvector storage

- Adapted the committed source Phase 4 implementation and transaction fix into SmartPick-AI.
- Added complete product persistence, aligned vector ingestion, provenance metadata, ordered
  evidence lookup, and exact pgvector cosine search with deterministic ties.
- Preserved explicit transactions for schema creation and ingestion, using autocommit for
  standalone operations to prevent type registration from masking uncommitted writes.
- Added the source tests, including the transaction regression and optional disposable-database
  integration coverage. Kept the general product-search description and first-category schema.
- Validation: 191 tests passed and one live-database integration test skipped in 3.13 seconds.
  Ruff lint passed; formatting passed for 33 files; the reused environment passed dependency checks.
- Loaded and validated the actual 3,062-product catalogue and aligned 384-dimensional embeddings.
  The catalogue hash and pinned MiniLM identity matched the Phase 3 artifacts.
- No disposable test database was configured for this run; no live ingestion or database-query
  result is claimed. No database service was created or changed.
- Worked only in the isolated SmartPick-AI preparation copy; Project 1's uncommitted changes
  were excluded. No Phase 5 agent implementation, API, interface, or container setup was included.

## 2026-09-16 — Phase 5: bounded agentic RAG and evidence tools

- Adapted the final committed source Phase 5 snapshot, including the verification review fixes,
  into SmartPick-AI's independent history.
- Added typed workflow contracts, catalogue search, product details, deterministic evidence
  verification, comparison policy, provider adapters, and one bounded LangGraph.
- Preserved the four-tool-call ceiling, one reformulation limit, strict stored-record checks,
  structured missing-information validation, and directional comparison safeguards.
- Updated setup, architecture, and limitations while retaining the general product-search
  presentation and the first smartphone catalogue's implemented contracts.
- Validation: 261 tests passed, with two optional live integrations skipped, in 4.82 seconds.
  Ruff lint passed; formatting passed for 43 files; dependency consistency passed in the reused
  Python environment.
- Live LLM testing was disabled and no disposable test database was configured for this run.
  No live-provider quality, paid API usage, or live-database result is claimed.
- Used only committed source files from Project 1; its uncommitted Gemini work was excluded.
  No Phase 6 API or interface, Phase 7 evaluation infrastructure, or Gemini integration was added.
