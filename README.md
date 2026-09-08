# SmartPick-AI

**Product choices, backed by catalogue evidence.**

SmartPick-AI is an incremental project for searching and comparing products against clear requirements such as budget, brand, rating, and category-specific specifications. The planned system combines retrieval-augmented generation (RAG) with deterministic filters so that recommendations can be traced to product records.

## Published checkpoint

**Phase 0: project foundation.** This repository currently includes Python packaging, environment configuration, logging, documentation, and smoke tests. Product data, search, storage, an agent workflow, and an interface are planned for later checkpoints; none is implemented in this published version. No search-quality results are claimed.

## What SmartPick aims to answer

> Compare products within my budget that meet my required specifications. Explain the trade-offs using the catalogue evidence.

This is a target use case, not a working feature at Phase 0. The planned workflow will:

1. Interpret the request and extract explicit shopping constraints.
2. Retrieve candidates using keyword, semantic, and hybrid search.
3. Enforce price, brand, rating, and category-specific requirements in Python.
4. Gather complete product records for comparison.
5. Check factual claims and product citations before returning an answer.
6. Explain when evidence is missing rather than inventing specifications.

## Run the foundation locally

Use Python 3.11 or newer. In PowerShell:

```powershell
git clone https://github.com/nityajainn/SmartPick-AI.git
cd SmartPick-AI
python -m venv .venv
.venv\Scripts\Activate.ps1
python -m pip install --upgrade pip
python -m pip install -e ".[dev]"
```

Run the foundation checks:

```powershell
python -m pytest
python -m ruff check .
python -m ruff format --check .
```

See [`.env.example`](.env.example) for configuration names. The foundation reads process environment variables directly; saving a local `.env` file does not automatically load it. Tests require no API credentials. Keep real secrets out of Git.

## Project map

| Path | Purpose |
|---|---|
| `src/searchrank_ai/` | Python foundation; original package name retained |
| `tests/` | Automated foundation checks |
| `docs/` | Architecture, decisions, build log, and evaluation notes |
| `data/` | Reserved for local datasets, ignored by Git |
| `artifacts/` | Reserved for generated indexes and outputs, ignored by Git |

## Development roadmap

| Phase | Deliverable | Status in this repository |
|---|---|---|
| 0 | Packaging, configuration, logging, and tests | Published |
| 1 | Dataset audit and cleaning | Planned for a later checkpoint |
| 2 | BM25 keyword retrieval | Planned for a later checkpoint |
| 3 | Semantic and hybrid retrieval with strict filters | Planned for a later checkpoint |
| 4 | PostgreSQL and pgvector storage | Planned for a later checkpoint |
| 5 | Bounded LangGraph workflow and evidence verification | Planned for a later checkpoint |
| 6 | API and demonstration interface | Planned |
| 7 | Evaluation, hardening, and Docker | Planned |
| 8 | Final documentation and release | Planned |

## Evidence and project history

SmartPick-AI starts its own repository history on September 8, 2026, from a Phase 0 foundation adapted from [SearchRank-AI](https://github.com/singlamohak16/SearchRank-AI). The original Python package name is retained. Earlier design notes are retained as background; they do not represent development performed today.

The project targets product catalogue search and comparison. A concrete category, dataset, and its supported filters will be selected and evaluated in the data phase. Multi-category retrieval is a design direction, not a capability of this foundation.

Product IDs, source evidence, missing values, and evaluation conditions will remain explicit as the project grows. Catalogue text is treated as untrusted input, and numerical constraints and factual checks belong in deterministic code.
