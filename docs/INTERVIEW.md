# Explaining SmartPick-AI

Describe SmartPick-AI as an adaptation of the SearchRank-AI foundation with its own phased history,
product-search presentation, validation records, and documentation. Be specific about work you
personally understand and performed; the [README provenance](../README.md) records the source.

## Points you can substantiate

- The validated first catalogue contains 3,062 smartphone records with stable IDs and source links.
- BM25, semantic, and hybrid retrieval share deterministic price, capacity, rating, and brand filters.
- The selected hybrid alpha 0.25 reached MRR@10 0.950 and NDCG@10 0.928558 on 20 reviewed
  development queries. The same set selected the weight; it is not held-out performance.
- Sixteen scripted agent scenarios exercise the real workflow and verifier around supplied model
  decisions. They do not establish real-model accuracy or a universal hallucination guarantee.

## Questions to prepare

**Why combine BM25 and embeddings?** BM25 makes exact names and terms inspectable. Embeddings
help related wording. The small benchmark favors hybrid top-rank quality while semantic-only
retrieval has higher recall. Larger independent judgments are needed before generalizing.

**Why keep constraints outside the model?** Budgets and minimum capacities are strict requirements.
Python rejects ineligible or missing values before ranking. Correct enforcement does not prove
that a model interpreted the request correctly; extraction needs separate evaluation.

**What does verification establish?** Accepted field values, citations, missing-information claims,
and supported comparison directions match retrieved catalogue evidence. The catalogue itself may
be incomplete or wrong, and tests cover particular adversarial cases rather than every attack.

**Does pgvector serve hybrid ranking?** The measured API hybrid path uses local BM25 and NumPy
artifacts. PostgreSQL stores evidence, vectors, and metadata, and exposes a separate exact vector
search capability. Do not describe the serving path as database-backed hybrid ranking.

**What remains?** Independent retrieval and live-model evaluation, a locked runtime, fresh-machine
setup verification, authentication, concurrency tests, accessibility, and deployment hardening.
Additional categories need their own dataset/schema/filter validation. Live retailer feeds would
also require permitted access, freshness tracking, stable identity, and update/re-index policies.

Use [measured results](RESULTS.md) and [architecture](ARCHITECTURE.md) to support explanations.
