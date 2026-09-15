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
Natural-language constraint extraction, an LLM, and the API remain later phases. Current search
text and structured fields cover smartphones; other categories require validated schemas and evidence.

## Phase 4 persistence

`storage.py` validates the 18-field catalogue and aligned semantic vectors before synchronizing
complete product records, embeddings, and provenance metadata in PostgreSQL. Ingestion uses an
advisory lock and one explicit transaction for upserts, stale-ID removal, and metadata replacement.
Standalone reads and type registration use autocommit so they cannot leave an outer transaction
that silently discards later writes when the connection closes.

Product lookup preserves requested order and reports missing IDs separately. Exact pgvector cosine
search accepts a validated query embedding and returns complete evidence with deterministic ties.
Database-backed hybrid scoring, connection pooling, and an API are not part of this checkpoint.

## Target request flow

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

The workflow will use an LLM only for request understanding, essential clarification, one bounded
query reformulation, and grounded response generation. Price, brand, rating, category-specific
filters, and deterministic evidence checks will remain ordinary Python logic.

## Planned component boundaries

- **Data:** schema, cleaning, provenance, and reproducible ingestion.
- **Retrieval:** independently measurable BM25, semantic, and hybrid implementations.
- **Storage:** product records and embeddings with traceable identifiers.
- **Workflow:** state and conditional routes with explicit retry/tool-call limits.
- **Providers:** configurable LLM interface plus a deterministic test double.
- **API:** validation and orchestration without presentation logic.
- **UI:** a single demonstration page that calls the API.
- **Evaluation:** reviewed scenarios, reproducible metrics, and documented failure cases.

These are target boundaries, not claims about completed functionality.
