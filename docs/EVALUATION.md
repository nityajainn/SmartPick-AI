# Evaluation

## 2026-09-15 — SmartPick-AI Phase 4 validation

The isolated Phase 4 copy passed 191 tests in 3.13 seconds, with one optional live-database
integration test skipped because no disposable test database was configured for this run.
Ruff lint, formatting for 33 files, and dependency consistency checks passed.

The storage loader validated 3,062 real catalogue records aligned with the 384-dimensional
semantic artifact, catalogue SHA-256
`c3428dd04e5d02c6a66ce00f5961f6b1f6ba9dcc30d86a37ffde84523619c7ea`, and the pinned
`all-MiniLM-L6-v2@c21050a7ef692090620a6d037dd736908f9c7cf6` encoder.
This verifies local input compatibility; it is not evidence of live database ingestion.

Tests cover conversion, schema SQL, parameterized ingestion, missing-ID handling, dimensions,
cosine result mapping, and the explicit-transaction/autocommit regression. No new retrieval
quality or live database performance metric is claimed. See [the storage guide](STORAGE.md).

## 2026-09-14 — SmartPick-AI Phase 3 validation

The isolated Phase 3 copy passed 182 tests in 1.90 seconds. Ruff lint and formatting checks passed
for 29 files; the reused Python environment passed dependency consistency checks.

The actual offline CPU evaluation used the cached pinned MiniLM encoder, the existing
3,062-product catalogue, BM25 index, semantic vectors, and 20 reviewed cases. The semantic index
was validated and reused, not rebuilt during this publication.

| Configuration | Recall@10 | MRR@10 | NDCG@10 | Constraint satisfaction |
|---|---:|---:|---:|---:|
| BM25 | 0.811111 | 0.812500 | 0.808979 | 1.000000 |
| Semantic | 0.937500 | 0.887500 | 0.899768 | 1.000000 |
| Hybrid, alpha 0.25 | 0.925000 | 0.950000 | 0.928558 | 1.000000 |
| Hybrid, alpha 0.50 | 0.859722 | 0.827222 | 0.826990 | 1.000000 |
| Hybrid, alpha 0.75 | 0.815278 | 0.816250 | 0.811935 | 1.000000 |

The recorded selection rule chooses alpha 0.25. These results reproduce the source benchmark;
the 20-case set is too small for broad product-search claims. Semantic-only recall is higher,
while the selected hybrid improves top-rank quality on this set. No LLM or network call was used.
The generated report stays in ignored local artifacts. Original model-build timings remain
historical and are labelled as such in [the hybrid report](HYBRID_RETRIEVAL.md).

## 2026-09-13 — SmartPick-AI Phase 2 validation

The isolated SmartPick-AI Phase 2 copy passed 162 tests, lint, and formatting checks. Using the
unchanged local catalogue with SHA-256
`c3428dd04e5d02c6a66ce00f5961f6b1f6ba9dcc30d86a37ffde84523619c7ea`,
two independent builds and evaluations produced byte-identical indexes and reports.

On 3,062 products and the 12 reviewed exact-model queries, Recall@10, MRR@10, and NDCG@10
were all 1.000. Every judged target was ranked first. This narrow benchmark does not measure broad
shopping needs, unjudged variants, misspellings, semantic concepts, or numeric filter compliance.
Generated artifacts are excluded from Git. See [the BM25 report](BM25_RETRIEVAL.md).

## Historical source evaluation records

Phase 2 provides a reviewed exact-model BM25 benchmark, expanded by the Phase 3 comparison above.
Agent evaluation remains a later phase. Earlier entries below retain the source audit dates.

On 2026-09-02, the Phase 0 suite ran on Windows with Python 3.12.13 and pytest 8.4.2:
3 smoke tests passed in 0.03 seconds. This is an engineering validation result, not a search-quality
metric.

Later phases will record measured results for BM25, semantic, and hybrid retrieval separately,
including Recall@10, NDCG@10, hybrid-alpha ablation, constraint satisfaction, citation correctness,
unsupported-claim behavior, tool-call counts, and latency. Every report will include the dataset
size, evaluation-set size, hardware, embedding model, caching conditions, and whether the LLM was
mocked or real.

This file must never contain estimated or fabricated metrics presented as results.

## 2026-09-03 — Dataset suitability checks, not retrieval evaluation

The [Amazon India audit](DATASET_AUDIT.md) records actual full-file counts, source hash, title
coverage, fixed-seed spot-check conditions, and limitations. Two runs of the final audit rules
produced identical non-timing results and byte-identical review/sample/category artifacts.

At the laptop-audit checkpoint, the synthetic suite had 56 passing tests. Coverage includes Hindi/English capacity
patterns, GPU-vs-RAM separation, ambiguous and missing evidence, accessories/desktops, numeric
sentinels, URL/ASIN agreement, source preservation, malformed input, and deterministic reruns.

The tentative candidate counts are **not** extraction accuracy, a manually verified usable-product
count, retrieval metrics, or proof of catalogue quality. No embedding model or LLM was used.
No paid API or network call is needed for tests or re-auditing an existing local archive.

## 2026-09-03 — Smartphone audit checkpoint

The [smartphone source audit](PHONE_DATASET_AUDIT.md) covers all 3,529 records with 2,805 unique IDs.
Its final rules identify 450 unique smartphone candidates, 260 explicit-core candidates, and 360
if inferred unlabelled capacity pairs are included. These are heuristic coverage counts, not verified
usable products, extraction accuracy, or retrieval results. The dataset is not recommended for adoption.

The combined synthetic suite produced **110 passed in 0.41 seconds** on Windows 11 build 26200,
Python 3.12.13. Ruff lint/format and dependency checks passed. Full-source repeat runs matched on all
non-run report fields and byte-identical JSONL/sample/duplicate artifacts, and preserved the source hash.
Fixed-seed category and explicit-core samples were inspected without an independent ground-truth set.
No LLM, embedding model, retailer access, or retrieval evaluation was used.

## 2026-09-04 — Adopted catalogue reproducibility checkpoint

The [91mobiles catalogue audit](MOBILE_CATALOGUE_AUDIT.md) measured 4,000 source rows and 3,062
eligible core-catalogue rows. The final set has 2,983 normalized ratings, 70 derived brands, unique
product IDs/source URLs, and no exact source duplicates. These are dataset and cleaning measurements,
not retrieval quality or independent specification-accuracy results.

The combined synthetic suite produced **142 passed in 0.50 seconds** on Windows 11 build 26200,
Python 3.12.13 on the final check. Tests cover field parsing, rating-scale
normalization, feature-phone and announcement exclusion, URL/ID validation, missing-value
preservation, raw-row retention, core-only schema enforcement, malformed sources, overwrite
refusal, duplicate-ID refusal, and deterministic reruns.

Two complete source runs matched on every report field except run metadata. Their cleaned CSV,
records JSONL, and sample JSON files were byte-identical. The source hash remained unchanged. No
LLM, embedding model, retailer request, image download, manually labelled ground truth, or search
evaluation was used.
