# SmartPick-AI

**Product choices, backed by catalogue evidence.**

SmartPick-AI is an incremental project for searching and comparing products against clear
requirements such as budget, brand, rating, and category-specific specifications. The
system combines retrieval-augmented generation (RAG) with deterministic filters so recommendations
can be traced to catalogue records.

## Current status

**Phases 0 through 8 are complete through release preparation.** A versioned GitHub release and
production deployment have not been published. Phase 1 establishes smartphones as the first evaluated catalogue
category. The adopted 91mobiles source audit retains 3,062 of 4,000 rows as price- and
capacity-filter-ready smartphones. The reproducible catalogue includes stable IDs, INR prices,
explicit RAM and storage, normalized ratings, comparison specifications, dates, and source URLs.

See [the adopted catalogue audit](docs/MOBILE_CATALOGUE_AUDIT.md) for the measured findings,
schema, reproduction command, and limitations. The rejected
[Amazon phone audit](docs/PHONE_DATASET_AUDIT.md), earlier
[laptop audit](docs/DATASET_AUDIT.md), and
[screening notes](docs/DATASET_CANDIDATES.md) remain as decision evidence. Raw data, processed
catalogues, and generated audit files remain local and ignored by Git.

SmartPick-AI is designed around a reusable product-search workflow. Its current data pipeline is
smartphone-specific because strict fields and cleaning rules must be validated for each category.
Additional product categories require their own approved dataset, schema, filters, and evaluation
rather than reusing phone assumptions.

The implemented search supports BM25, semantic, and hybrid ranking with traceable product IDs,
source URLs, and score components. Phase 3 adds strict price, RAM, storage, rating, and brand
filters, plus a reviewed 20-case retrieval benchmark. Phase 4 adds PostgreSQL/pgvector persistence,
transactional catalogue ingestion, ordered product lookup, and database vector search.
Phase 5 connects retrieval and stored evidence through a bounded agent workflow with deterministic
claim verification. Phase 6 exposes the system through a FastAPI backend and a local Streamlit
search/comparison interface. Phase 7 adds reviewed evaluation scenarios, storage readiness checks,
and a Docker Compose setup for local use.

Phase 8 consolidates setup, demonstration, results, interview notes, and release checks.
The current package remains `0.1.0.dev0`.

| Guide | What it covers |
|---|---|
| [Setup](docs/SETUP.md) | Installation, data preparation, Docker, and troubleshooting |
| [Demonstration](docs/DEMO.md) | Search and optional agent comparison walkthrough |
| [Architecture](docs/ARCHITECTURE.md) | Component boundaries and evidence flow |
| [Results](docs/RESULTS.md) | Measured metrics, denominators, and reproduction |
| [Interview notes](docs/INTERVIEW.md) | Design trade-offs and accurate project claims |
| [Release preparation](docs/RELEASE.md) | Validation and remaining versioned-release decisions |

## What SmartPick aims to do

When complete, the application will:

1. Interpret natural-language product-shopping requests.
2. Extract budget, brand, rating, and category-specific requirements.
3. Compare BM25 keyword, semantic, and hybrid retrieval.
4. Apply strict constraints in deterministic Python code.
5. Retrieve complete records for shortlisted products.
6. Generate comparisons using only stored catalogue evidence.
7. Verify factual claims and product-ID citations.
8. State when the catalogue lacks enough evidence.

The project will not train models, scrape live stores, purchase products, or implement user
profiling. See [the architecture notes](docs/ARCHITECTURE.md) for the system boundary.

## Set up the project

Use Python 3.11 or newer. In PowerShell:

```powershell
git clone https://github.com/nityajainn/SmartPick-AI.git
cd SmartPick-AI
python -m venv .venv
.venv\Scripts\Activate.ps1
python -m pip install --upgrade pip
python -m pip install -e ".[dev]"
```

Run the checks:

```powershell
python -m pytest -q
python -m ruff check .
python -m ruff format --check .
```

## Reproduce the Phase 1 catalogue

The real dataset is intentionally excluded from Git. Place the approved archive at the documented
ignored path, then run:

```powershell
python -X utf8 -m searchrank_ai.mobile_catalogue `
  --archive data\raw\suresh_91mobiles_2008_2026\source.zip `
  --audit-dir artifacts\suresh_91mobiles_audit\new-run `
  --catalogue data\processed\suresh_91mobiles_2008_2026\catalogue.csv
```

The command validates the source schema, records explicit rejection reasons, preserves original
values in local audit evidence, and refuses to overwrite existing outputs.

## Build and search the keyword index

After producing the Phase 1 catalogue:

```powershell
python -m searchrank_ai.bm25 build --catalogue data/processed/suresh_91mobiles_2008_2026/catalogue.csv --output artifacts/bm25/phase2-index.json
python -m searchrank_ai.bm25 search --index artifacts/bm25/phase2-index.json --query "snapdragon amoled" --limit 10
python -m searchrank_ai.bm25 evaluate --index artifacts/bm25/phase2-index.json --cases evaluation/bm25_navigational.json --limit 10 --output artifacts/bm25/phase2-evaluation.json
```

Results include product IDs, names, source URLs, and BM25 scores. Indexes and reports remain local
and ignored by Git. Output paths must be new because commands refuse to overwrite existing files.
A phrase such as "under 30000" does not enforce a price filter in the BM25 keyword command.
Use the structured Phase 3 options below for hard requirements.

See [the BM25 report](docs/BM25_RETRIEVAL.md) for scoring, benchmark conditions, and limitations.

## Semantic and hybrid search with strict filters

Build the semantic index after generating the catalogue and BM25 index. The first build needs
the pinned public model revision; later runs can use its local cache.

```powershell
python -m searchrank_ai.retrieval build-semantic --catalogue data/processed/suresh_91mobiles_2008_2026/catalogue.csv --output artifacts/semantic/phase3-index.npz --device cpu
python -m searchrank_ai.retrieval search --catalogue data/processed/suresh_91mobiles_2008_2026/catalogue.csv --bm25-index artifacts/bm25/phase2-index.json --semantic-index artifacts/semantic/phase3-index.npz --query "oneplus nord 6" --mode hybrid --alpha 0.25 --max-price 40000 --min-ram 8 --min-storage 256 --min-rating 4.4 --include-brand OnePlus
```

Strict filters run before ranking. Missing ratings fail a minimum-rating requirement, and
conflicting included/excluded brands are rejected. These command-line filters are supplied explicitly;
natural-language interpretation is available through the configured agent workflow. The filters and semantic search
text are validated for the first smartphone catalogue.

See [the hybrid retrieval report](docs/HYBRID_RETRIEVAL.md) for the pinned model, normalization,
benchmark conditions, reproduction commands, and limitations.

## PostgreSQL and pgvector storage

Phase 4 expects an existing PostgreSQL database with the pgvector extension available. Set
`SEARCHRANK_DATABASE_URL` in your local environment as described in
[the storage guide](docs/STORAGE.md). The configured database is synchronized to the supplied
catalogue: ingestion upserts its records and removes stored IDs absent from that catalogue.

```powershell
python -m searchrank_ai.storage ingest --catalogue data/processed/suresh_91mobiles_2008_2026/catalogue.csv --semantic-index artifacts/semantic/phase3-index.npz
python -m searchrank_ai.storage details 91mobiles:xiaomi-redmi-turbo-5
```

Ingestion validates the catalogue hash, model identity, vector dimensions, and product alignment
before database writes. Product lookup preserves requested order and reports unknown IDs explicitly.
The existing in-memory hybrid retriever remains available.

The Phase 4 suite passed 191 tests; one optional live-database test was skipped. Real catalogue
and embedding alignment was validated locally. No live database result is claimed for this run.
Use a disposable database for the optional integration test documented in the storage guide.

## Bounded agent workflow

Phase 5 provides a Python service boundary using one LangGraph workflow and three tools:
catalogue search, product details, and evidence verification. It handles search, comparison,
clarification, unsupported requests, conflicting constraints, no results, and rejected evidence.
Each request permits at most four tool calls and one unsuccessful-search reformulation.

The workflow rechecks retrieved database records against the original constraints before drafting
an answer. Facts, citations, missing-information claims, and numeric comparison directions must
pass deterministic verification before rendering. Catalogue text cannot authorize tool calls or
relax constraints. The current contracts and evidence policy use the first smartphone catalogue.

`MockLLMProvider` supports network-free tests. An optional OpenAI provider reads credentials and
model configuration from the environment. Phase 6 connects the workflow to the API and interface
described below; provider setup is described in
[the agent workflow guide](docs/AGENTIC_RAG.md).

The Phase 5 suite passed 261 tests; the optional live-provider and live-database tests were skipped.
These tests validate routing and evidence checks with mocks, not real-model answer quality.

## Run the API and local interface

Generate the catalogue and retrieval indexes using the commands above. Configure environment
variables from `.env.example` in your shell or an ignored local environment loader. Start the API:

```powershell
python -m uvicorn searchrank_ai.api:app --reload
```

In a second terminal, activate the same environment and start the interface:

```powershell
$env:SEARCHRANK_API_URL = "http://127.0.0.1:8000"
python -m streamlit run src/searchrank_ai/streamlit_app.py
```

Open the local address printed by Streamlit. Search supports BM25, semantic, and hybrid retrieval
with strict filters. Ask / Compare displays verified answers and their supporting workflow evidence.
Smartphones remain the first implemented catalogue; the general product-search design does not
imply support for other category schemas yet.

The API provides `GET /health`, `POST /search`, `POST /query`, and
`GET /products/{product_id}`, with interactive documentation at `http://127.0.0.1:8000/docs`.
Search needs the local catalogue, indexes, and cached embedding model. Product lookup also needs
PostgreSQL; natural-language queries additionally need the configured real provider.
Health reports which components are ready, so missing optional setup does not hide working search.
The mock provider is reserved for scripted tests.

This is a local demonstration without authentication or production deployment controls.
See [the API and interface guide](docs/API_AND_UI.md) for setup, response contracts, and limitations.

Phase 6 validation on September 17, 2026: **280 tests passed, 2 optional integrations skipped**.
Lint, formatting, and dependency checks passed. API and interface tests use synthetic services;
this run does not establish live-provider quality or production readiness.

## Evaluation and Docker

Phase 7 combines 20 catalogue-grounded retrieval cases with 16 scripted agent scenarios.
The agent benchmark checks workflow routes, constraints, citations, refusals, and rejected
evidence using synthetic products and scripted provider responses. It does not measure live-model
understanding or establish accuracy across all product categories.

```powershell
python -m searchrank_ai.evaluation agent --cases evaluation/agent_scenarios_v1.json --output artifacts/evaluation/phase7-agent.json
```

Reports refuse to overwrite an existing file; choose a fresh output name for repeat runs.
See [the Phase 7 report](docs/PHASE_7_EVALUATION.md) for retrieval and API timing commands,
measured results, historical source evidence, and limitations.

The local Docker setup includes PostgreSQL/pgvector, the API, the interface, and opt-in ingestion.
Generate the catalogue and indexes first, then run:

```powershell
docker compose up -d database
docker compose --profile setup run --rm --build ingest
docker compose up --build -d api ui
docker compose ps
```

Compose uses the `smartpick-ai` project name, giving it separate database and model-cache volumes.
Default API/UI ports are 8000/8501 on localhost; if another project is using them, choose free host
ports in this copy's Compose file. The database is not exposed on a host port. Model files may be
downloaded into the cache on first startup; provider credentials come from your local environment.
Data and artifacts are mounted read-only and excluded from the image.

Phase 7 validation on September 18, 2026 passed **315 tests, with 6 optional integrations skipped**.
Lint, formatting, dependency checks, and Compose configuration validation passed.
Live container, database, and paid-provider tests were not run for this publication.
The source project's earlier Docker measurements are explicitly labelled historical.

Phase 8 validation on September 22 passed **324 tests, with 6 optional integrations skipped**.
Lint, formatting, dependency checks, and Compose configuration validation passed.
The release checker verifies local documentation links and flags common publication mistakes:

```powershell
python scripts/check_release.py
```

It checks candidate files, not Git history, remote URLs, or all possible credential formats.

## Project map

| Path | Purpose |
|---|---|
| `src/searchrank_ai/` | Package code; original internal package name retained |
| `tests/` | Synthetic automated tests |
| `evaluation/` | Reviewed retrieval queries, relevance judgments, and scripted agent scenarios |
| `docs/` | Architecture, decisions, audits, build log, and evaluation notes |
| `scripts/check_release.py` | Read-only local links and release file checks |
| `data/` | Local raw and processed data, ignored by Git |
| `artifacts/` | Generated audits and later retrieval artifacts, ignored by Git |

## Roadmap

| Phase | Deliverable | Status |
|---:|---|---|
| 0 | Project foundation | Complete |
| 1 | First category dataset audit, schema, and cleaning | Complete: smartphones |
| 2 | BM25 keyword retrieval baseline | Complete |
| 3 | Semantic and hybrid retrieval with strict filters | Complete |
| 4 | PostgreSQL and pgvector storage | Complete |
| 5 | Bounded agent workflow and evidence verification | Complete |
| 6 | API and demonstration interface | Complete |
| 7 | Evaluation, hardening, and Docker | Complete |
| 8 | Final documentation and release preparation | Complete; versioned release pending |

## Provenance

SmartPick-AI began from an adapted SearchRank-AI Phase 0 foundation and now records its own
development history. Phase 2 adapts the original BM25 implementation and tests as new commits in
this repository. Phase 3 likewise adapts the source semantic, hybrid, and constraint components
into new SmartPick-AI commits. Phase 4 adapts the storage implementation, including its transaction
fix and regression test. Phase 5 adapts the bounded workflow and reviewed verification fixes.
Phase 6 adapts the API, Streamlit interface, and reviewed integration fixes with SmartPick-AI branding.
Phase 7 adapts the evaluation runners, container setup, and reviewed readiness/metric fixes.
Phase 8 adapts source documentation and release checks to the features present in this repository;
later source Gemini and shopping-interface extensions are not included.
The retained audit reports show how evidence changed the catalogue decision,
including rejected laptop and Amazon-phone candidates. They are engineering evidence, not active
catalogue data.

Product IDs, source evidence, missing values, and evaluation conditions remain explicit. Catalogue
text is treated as untrusted input, while numerical constraints and factual checks belong in
deterministic code.
