# SmartPick-AI local setup

These commands target PowerShell on Windows from the repository root. Python 3.11 or newer is
required. The first dependency/model installation needs internet access. Product search does not
need an LLM key; the Ask / Compare workflow needs a configured real provider and product storage.

## Install

```powershell
git clone https://github.com/nityajainn/SmartPick-AI.git
Set-Location SmartPick-AI
python -m venv .venv
.\.venv\Scripts\python.exe -m pip install --upgrade pip
.\.venv\Scripts\python.exe -m pip install -e '.[dev]'
```

Explicit executable paths avoid needing to change PowerShell's activation policy. A CPU is
sufficient. Dependency ranges and container image tags are not a complete environment lock.

## Obtain and prepare the catalogue

Obtain the dataset separately from the publisher linked in the
[catalogue audit](MOBILE_CATALOGUE_AUDIT.md), after reviewing its usage conditions.
Keep the original ZIP at `data/raw/suresh_91mobiles_2008_2026/source.zip`.
Create the local directories if needed. Raw data is not distributed in this repository.

The recorded source archive SHA-256 is
`98604a14020c9e7e45bcf0b578540d7a20f8dad4be8ad31686a05720b438c297`.
A replacement dataset requires a new audit and corresponding evaluation judgments.

```powershell
.\.venv\Scripts\python.exe -X utf8 -m searchrank_ai.mobile_catalogue --archive data/raw/suresh_91mobiles_2008_2026/source.zip --audit-dir artifacts/audit/initial --catalogue data/processed/suresh_91mobiles_2008_2026/catalogue.csv
.\.venv\Scripts\python.exe -m searchrank_ai.bm25 build --catalogue data/processed/suresh_91mobiles_2008_2026/catalogue.csv --output artifacts/bm25/phase2-index.json
.\.venv\Scripts\python.exe -m searchrank_ai.retrieval build-semantic --catalogue data/processed/suresh_91mobiles_2008_2026/catalogue.csv --output artifacts/semantic/phase3-index.npz --device cpu
```

The matching snapshot produces 3,062 products with catalogue SHA-256
`c3428dd04e5d02c6a66ce00f5961f6b1f6ba9dcc30d86a37ffde84523619c7ea`.
MiniLM uses 384-dimensional vectors and pinned revision
`c21050a7ef692090620a6d037dd736908f9c7cf6`.
Writers refuse overwrites. Use fresh output paths for intentional rebuilds and keep application
paths consistent; never mix indexes from different catalogues.

## Run with Docker

Start Docker Desktop in Linux-container mode. For a new setup without live AI queries, use a shell
with no inherited real-provider settings and no credential-bearing Compose environment file:

```powershell
$env:SEARCHRANK_LLM_PROVIDER = 'mock'
$env:SEARCHRANK_LLM_MODEL = ''
$env:SEARCHRANK_LLM_API_KEY = ''
docker compose config --quiet
docker compose up -d database
docker compose --profile setup run --rm --build ingest
docker compose up --build -d api ui
docker compose ps
Invoke-RestMethod http://127.0.0.1:8000/health
```

Open the UI at `http://127.0.0.1:8501` or API docs at `http://127.0.0.1:8000/docs`.
Expect search and products ready, query unready, and overall degraded status with the mock provider.
The Search tab works; Ask / Compare requires the real provider. Ingestion synchronizes the entire
catalogue transactionally, including removing absent IDs, so use it for setup or deliberate updates.

The Compose project is `smartpick-ai`; its named volumes are separate from SearchRank-AI.
Ports 8000/8501 must be free. If another project uses them, change this copy's host mappings and
use the matching browser URLs. PostgreSQL is internal to the Compose network.
Container model caching is separate from host caching, so first startup may download weights.

For existing unchanged containers, use `docker compose start database api ui`.
`docker compose stop` preserves named volumes. Do not delete volumes to solve routine startup errors.

## Run without Docker

Follow [storage setup](STORAGE.md) for PostgreSQL/pgvector. Set configuration from
`.env.example` in the shell or an ignored local environment loader; Python does not load it
automatically. Run these in separate terminals:

```powershell
.\.venv\Scripts\python.exe -m uvicorn searchrank_ai.api:app --reload
```

```powershell
$env:SEARCHRANK_API_URL = 'http://127.0.0.1:8000'
.\.venv\Scripts\python.exe -m streamlit run src/searchrank_ai/streamlit_app.py
```

See [the workflow guide](AGENTIC_RAG.md) for the optional OpenAI adapter and environment settings.
Configure credentials privately. Queries can incur provider costs; health checks do not call the
provider or verify its quota. This checkpoint does not include Gemini or a shortlist comparison UI.

## Checks

```powershell
$env:SEARCHRANK_RUN_LLM_INTEGRATION = '0'
$env:SEARCHRANK_TEST_DATABASE_URL = ''
$env:SEARCHRANK_TEST_API_URL = ''
$env:SEARCHRANK_TEST_UI_URL = ''
.\.venv\Scripts\python.exe -m pytest -q
.\.venv\Scripts\python.exe -m ruff check .
.\.venv\Scripts\python.exe -m ruff format --check .
.\.venv\Scripts\python.exe -m pip check
.\.venv\Scripts\python.exe scripts/check_release.py
```

Normal tests use synthetic fixtures. Optional integrations skip without their configured services.
Compose configuration validation can run without starting containers. Evaluation commands are in
[results](RESULTS.md).

| Symptom | Check |
|---|---|
| Search unready | All catalogue/index paths, matching hashes, and model cache |
| Products unready | Database health and ingestion of the matching catalogue |
| Only query unready | Expected for mock; otherwise check provider configuration privately |
| Port occupied | Use free host ports without stopping another project's services |
| No results | Loosen explicit constraints; missing fields cannot satisfy required filters |
| Output already exists | Preserve it and select a new output filename |
| Test temporary directory error | Choose a fresh dedicated directory with an existing parent |

Do not share credentials, environment dumps, or resolved Compose output containing secrets.
This guide was checked against the current configuration; a fresh-machine installation was not
performed during Phase 8.
