# Architecture

## Current architecture

SmartPick-AI targets evidence-grounded product catalogue search and comparison. Phase 1 uses
smartphones as the first implemented category because each category requires its own validated
schema and strict-filter semantics. The Phase 0 foundation contains a Python package boundary,
validated environment configuration, shared logging, and smoke tests. Phase 1 adds local
suitability-audit paths:

```text
Unchanged local ZIP -> streamed CSV inventory -> exact laptop-category records
                    -> deterministic title/numeric/ID probes -> local review artifacts
```

`data_audit.py` handles streaming, validation, counts, provenance, and artifact output.
`audit_titles.py` estimates title-field coverage and flags uncertainty; it is not a production
cleaner or product schema. Original records remain separate from derived findings. No audit step
executes catalogue text, calls an LLM, downloads images, or checks live product pages.

The retained flow above belongs to the rejected laptop source. The rejected Amazon smartphone
candidate follows this audit-only path:

```text
Unchanged phone ZIP -> all six-column CSV records -> phone_titles evidence probes
                    -> numeric / direct-or-sponsored URL checks / duplicate groups
                    -> report.json + records.jsonl + deterministic samples.json
```

`phone_audit.py` preserves all records and reports duplicate-field conflicts without selecting or
deleting rows. `phone_titles.py` separates explicit capacities from unlabelled pair hypotheses,
flags expansion/virtual-RAM uncertainty, and assigns review categories. Unknown remains unknown.
These modules are audit tools, not the final product schema or production cleaning pipeline.

The adopted 91mobiles path is:

```text
Unchanged 17-column ZIP -> schema and source-hash validation -> explicit field parsers
                        -> evidence-based eligibility reasons -> 18-field core catalogue CSV
                        -> report.json + records.jsonl + deterministic samples.json
```

`mobile_catalogue.py` validates all rows before writing, derives stable IDs from unique source URL
slugs, normalizes only explicit units/scales, rejects capacity-evidenced feature phones and
announced products, and refuses output overwrites or duplicate product IDs. The audit JSONL retains
every raw value. The generated catalogue excludes `spec_score`, `antutu_score`, `awards`,
`expert_rating`, and `store` as instructed. See [the catalogue audit](MOBILE_CATALOGUE_AUDIT.md).

## Phase 2 keyword search

The validated 18-field catalogue feeds a weighted Unicode tokenizer and a versioned BM25 JSON
index. Queries return product IDs, names, source URLs, and scores with deterministic tie-breaking.
The index records the catalogue hash, scoring parameters, and document term frequencies.

`bm25.py` handles building, loading, searching, and binary-relevance evaluation using the Python
standard library. Product names have weight 3, brands weight 2, and specification evidence weight 1.
The reviewed benchmark pins the catalogue hash so changed data cannot silently reuse judgments.

This component provides lexical retrieval; Phase 3 composes it with the semantic index and filters.

## Phase 3 semantic and hybrid retrieval

Labelled catalogue evidence is encoded using the pinned MiniLM model as normalized 384-dimensional
vectors. The versioned NPZ artifact retains the model identity, catalogue hash, and ordered IDs.
`semantic.py` owns encoding and artifact validation; `retrieval.py` aligns this artifact with
the catalogue and BM25 index and supports independent BM25, semantic, and hybrid search.

Strict maximum-price, minimum-RAM/storage/rating, and included/excluded-brand checks run across
the complete catalogue before normalization and ranking. Missing ratings fail the rating filter;
contradictory brand requirements are rejected. Hybrid scoring combines max-normalized eligible
BM25 scores with shifted cosine similarity. The selected alpha is 0.25.

The hybrid matrix stays in memory; Phase 4 adds a separate PostgreSQL/pgvector storage boundary.
Phase 5 adds request interpretation and an optional LLM provider; Phase 6 adds the API. Current search
text and structured fields cover smartphones; other categories require validated schemas and evidence.

## Phase 4 persistence

`storage.py` validates the 18-field catalogue and aligned semantic vectors before synchronizing
complete product records, embeddings, and provenance metadata in PostgreSQL. Ingestion uses an
advisory lock and one explicit transaction for upserts, stale-ID removal, and metadata replacement.
Standalone reads and type registration use autocommit so they cannot leave an outer transaction
that silently discards later writes when the connection closes.

Product lookup preserves requested order and reports missing IDs separately. Exact pgvector cosine
search accepts a validated query embedding and returns complete evidence with deterministic ties.
Database-backed hybrid scoring and connection pooling remain deferred. Phase 6 exposes stored
product lookup through the API.

## Phase 5 workflow and evidence boundary

`workflow.py` owns one bounded LangGraph with separate search, comparison, clarification,
unsupported, conflict, no-result, and verification-failure routes. `agent_models.py` defines
typed request, claim, citation, and result contracts; `agent_tools.py` wraps retrieval, ordered
product lookup, and deterministic verification. `evidence_policy.py` binds comparison criteria
to allowed numeric fields and directions.

The provider may interpret requests, reformulate an unsuccessful query once, and propose
structured facts. It cannot override numerical constraints or citation checks. Stored products
are rechecked against the original constraints before drafting. Missing-information claims must
refer to recognized fields and retrieved IDs, with null stored values or explicitly unsupported
attributes. Only verified facts and unavailable items are rendered.

The maximum is four recorded tool calls per request. `llm.py` supplies the network-free mock
and optional OpenAI adapter. Normal tests use synthetic evidence and scripted provider responses.
Phase 6 connects this workflow to the API and interface shown below.

## Implemented request flow

```text
User
  -> Streamlit demonstration
  -> FastAPI backend
  -> one bounded LangGraph workflow
       -> catalogue search tool (BM25, semantic, or hybrid + strict filters)
       -> product details tool
       -> evidence verification tool
  -> grounded answer with product-ID citations

Storage: PostgreSQL + pgvector
```

The workflow uses the provider only for request understanding, essential clarification, one bounded
query reformulation, and a structured answer draft. Price, brand, rating, category-specific
filters, evidence checks, and final rendering remain deterministic Python logic.

## Phase 6 application boundary

`api_models.py` validates requests and response contracts. `api.py` exposes health, direct search,
grounded queries, and stored product lookup with consistent error responses. Blocking work runs
in FastAPI's thread pool. `services.py` loads dependencies once and reports readiness separately
for search, product lookup, and agent queries; embedding startup uses cached files by default.

`streamlit_app.py` uses only the HTTP client in `api_client.py`. The interface displays ranked
results, extracted constraints, verified answers, and workflow evidence without duplicating
retrieval or verification logic. Catalogue labels are escaped before controlled Markdown rendering.
The local demonstration does not provide authentication, pooling, or production deployment controls.
See [the API and interface guide](API_AND_UI.md).

## Phase 7 evaluation and local containers

`evaluation.py` runs reviewed scripted-agent scenarios and warmed in-process search timing.
The agent runner records per-case routes, status, tools, retries, verification outcomes, and metric
denominators. Metrics without applicable cases are null with a zero denominator; an empty suite is
invalid. These controls test behavior around supplied model decisions, not real-model understanding.

Storage readiness uses a read-only ingestion/readability probe. The health endpoint refreshes its
state on every request; incomplete or mismatched storage also marks dependent queries unready.
The Docker API health check requires both retrieval and products to be ready.

The two-stage image runs as an unprivileged user. Compose defines the database, API, UI, and
optional ingestion service; local data mounts are read-only. The `smartpick-ai` project name
separates its named volumes from SearchRank-AI. Host ports still need to be free when starting it.

## Component boundaries

- **Data:** schema, cleaning, provenance, and reproducible ingestion.
- **Retrieval:** independently measurable BM25, semantic, and hybrid implementations.
- **Storage:** product records and embeddings with traceable identifiers.
- **Workflow:** state and conditional routes with explicit retry/tool-call limits.
- **Providers:** configurable LLM interface plus a deterministic test double.
- **API:** validation and orchestration without presentation logic.
- **UI:** a single demonstration page that calls the API.
- **Evaluation:** reviewed scenarios, reproducible metrics, and documented failure cases.

Data, retrieval, storage, workflow, provider, API, and interface boundaries are implemented
through Phase 6. Phase 7 adds containers and controlled retrieval/agent evaluation. Broad live-model
quality, production deployment, and concurrent load remain outside the measured scope.

## Phase 8 documentation and release checks

The application boundaries remain unchanged. [Setup](SETUP.md), [demo](DEMO.md),
[results](RESULTS.md), and [interview notes](INTERVIEW.md) describe the implemented checkpoint.
`scripts/check_release.py` checks candidate files and local Markdown links without publishing,
deleting, or modifying them. Its tests cover generated paths, redacted credential findings,
file size/encoding, and missing link targets.

[Release preparation](RELEASE.md) separates phase completion from version/tag creation and
production deployment. The package remains `0.1.0.dev0`. Historical source additions beyond
SmartPick-AI's imported Phase 7 snapshot do not become capabilities through documentation.
