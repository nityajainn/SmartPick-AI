# SmartPick-AI

**Product choices, backed by catalogue evidence.**

SmartPick-AI is an incremental project for searching and comparing products against clear
requirements such as budget, brand, rating, and category-specific specifications. The planned
system combines retrieval-augmented generation (RAG) with deterministic filters so recommendations
can be traced to catalogue records.

## Current status

**Phases 0 through 2 are complete.** Phase 1 establishes smartphones as the first evaluated catalogue
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

Phase 2 adds deterministic BM25 keyword search, a local index, traceable ranked results, and a
reviewed 12-query exact-model benchmark. Semantic retrieval, strict shopping filters, storage,
an agent workflow, API, and interface remain later phases.

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
A phrase such as "under 30000" does not enforce a price filter in this phase.

See [the BM25 report](docs/BM25_RETRIEVAL.md) for scoring, benchmark conditions, and limitations.

## Project map

| Path | Purpose |
|---|---|
| `src/searchrank_ai/` | Package code; original internal package name retained |
| `tests/` | Synthetic automated tests |
| `evaluation/` | Reviewed retrieval queries and relevance judgments |
| `docs/` | Architecture, decisions, audits, build log, and evaluation notes |
| `data/` | Local raw and processed data, ignored by Git |
| `artifacts/` | Generated audits and later retrieval artifacts, ignored by Git |

## Roadmap

| Phase | Deliverable | Status |
|---:|---|---|
| 0 | Project foundation | Complete |
| 1 | First category dataset audit, schema, and cleaning | Complete: smartphones |
| 2 | BM25 keyword retrieval baseline | Complete |
| 3 | Semantic and hybrid retrieval with strict filters | Not started |
| 4 | PostgreSQL and pgvector storage | Not started |
| 5 | Bounded agent workflow and evidence verification | Not started |
| 6 | API and demonstration interface | Not started |
| 7 | Evaluation, hardening, and Docker | Not started |
| 8 | Final documentation and release | Not started |

## Provenance

SmartPick-AI began from an adapted SearchRank-AI Phase 0 foundation and now records its own
development history. Phase 2 adapts the original BM25 implementation and tests as new commits in
this repository. The retained audit reports show how evidence changed the catalogue decision,
including rejected laptop and Amazon-phone candidates. They are engineering evidence, not active
catalogue data.

Product IDs, source evidence, missing values, and evaluation conditions remain explicit. Catalogue
text is treated as untrusted input, while numerical constraints and factual checks belong in
deterministic code.
