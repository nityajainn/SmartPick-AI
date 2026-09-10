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
