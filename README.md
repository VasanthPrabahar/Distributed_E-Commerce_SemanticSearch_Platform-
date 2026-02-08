## Project Status
- A production-style, distributed e-commerce semantic search platform built incrementally with a focus on scalability and correctness.


---

## v0.1 — Data Ingestion & Distributed Storage (Completed)

**Goal:** Build a reliable, reproducible data ingestion and distributed storage foundation using real-world, large-scale Amazon datasets.

This milestone focuses entirely on correctness, scalability, and infrastructure hygiene before introducing any search or retrieval layers.

**What’s included**
- Streaming preprocessing pipeline:
  - `scripts/convert_meta_reviews.py`
  - Two-pass, memory-safe design (no full dataset loading)
  - Deterministic reservoir sampling (seeded) for reproducibility
    - ~10,000 products
    - ~50,000 reviews
  - Cleans HTML artifacts, malformed text, whitespace, and missing fields
- Cleaned outputs for downstream systems:
  - `data/out/products_small.csv`
  - `data/out/reviews_small.csv`
  - `data/out/asin_sample_list.txt`
- Distributed SQL storage:
  - 3-node CockroachDB cluster running locally via Docker Compose
  - Schema design for `products` and `reviews`
  - Bulk ingestion using CockroachDB `IMPORT`
- Verification and validation:
  - Row counts verified after import
  - Cluster health checked
  - Replication and range distribution across nodes validated
- Dockerized environment:
  - Fully reproducible local setup
  - No dependency on host Python environment
- Git & repository hygiene:
  - Removed virtual environments and large binaries from Git history
  - Clean `.gitignore` and `.dockerignore`
  - Repository kept recruiter-grade and lightweight

**Design decisions**
- Raw Amazon datasets (20+ GB total) are intentionally excluded from the repository.
- All preprocessing is streaming to avoid memory pressure.
- No search, embeddings, or APIs introduced at this stage by design.

**Outcome**
- A stable, distributed storage layer ready to support search and retrieval workloads.
- Deterministic sample datasets that can be regenerated at any time.

**How to run (short)**
```bash
# preprocess raw data (raw JSON paths configured in script)
python scripts/convert_meta_reviews.py

# start distributed CockroachDB cluster
docker compose up -d

# bulk import CSVs into CockroachDB
cockroach sql --insecure --host=localhost:2625
```
## v0.2 — Product Search Infrastructure (Completed)

**Goal:** Introduce a robust lexical search layer for products using Elasticsearch (BM25) while keeping APIs and semantic retrieval intentionally out of scope.

This milestone validates search relevance, indexing correctness, and distributed system integration.

**What’s included**
- Elasticsearch integration:
  - Official Elasticsearch 8.x image (ARM64-compatible)
  - Runs locally via Docker Compose alongside CockroachDB
- Product indexing pipeline:
  - `scripts/index_products_to_elasticsearch.py`
  - Bulk indexing of ~10,000 products from `products_small.csv`
- Index design and relevance tuning:
  - Explicit mappings for product fields
  - BM25 similarity (default Elasticsearch ranking)
  - Field boosting (e.g., `title^3`) to improve relevance
- Verification and validation:
  - Index document counts validated
  - Search correctness checked via `_search`
  - Ranking behavior inspected via `_score`
- Infrastructure-first approach:
  - No FastAPI endpoints added yet (intentional)
  - No semantic embeddings or vector search at this stage

**Design decisions**
- Lexical search is introduced before semantic retrieval to:
  - Establish a strong baseline
  - Validate indexing and relevance independently
- API layer is deferred until both product and review retrieval are ready.

**Outcome**
- A production-style product search layer capable of fast, relevant keyword-based retrieval.
- Clean separation between storage (CockroachDB) and search (Elasticsearch).

**How to run (short)**
```bash
# start services (CockroachDB + Elasticsearch)
docker compose up -d

# index products into Elasticsearch
python scripts/index_products_to_elasticsearch.py

# test a sample query
curl "http://localhost:9200/products/_search?q=wireless"
```
---

## v0.2 — Product Search Infrastructure (Completed)

**Goal:** Introduce a robust lexical search layer for products using Elasticsearch (BM25) while keeping APIs and semantic retrieval intentionally out of scope.

This milestone validates search relevance, indexing correctness, and distributed system integration.

**What’s included**
- Elasticsearch integration:
  - Official Elasticsearch 8.x image (ARM64-compatible)
  - Runs locally via Docker Compose alongside CockroachDB
- Product indexing pipeline:
  - `scripts/index_products_to_elasticsearch.py`
  - Bulk indexing of ~10,000 products from `products_small.csv`
- Index design and relevance tuning:
  - Explicit mappings for product fields
  - BM25 similarity (default Elasticsearch ranking)
  - Field boosting (e.g., `title^3`) to improve relevance
- Verification and validation:
  - Index document counts validated
  - Search correctness checked via `_search`
  - Ranking behavior inspected via `_score`
- Infrastructure-first approach:
  - No FastAPI endpoints added yet (intentional)
  - No semantic embeddings or vector search at this stage

**Design decisions**
- Lexical search is introduced before semantic retrieval to:
  - Establish a strong baseline
  - Validate indexing and relevance independently
- API layer is deferred until both product and review retrieval are ready.

**Outcome**
- A production-style product search layer capable of fast, relevant keyword-based retrieval.
- Clean separation between storage (CockroachDB) and search (Elasticsearch).

**How to run (short)**
```bash
# start services (CockroachDB + Elasticsearch)
docker compose up -d

# index products into Elasticsearch
python scripts/index_products_to_elasticsearch.py

# test a sample query
curl "http://localhost:9200/products/_search?q=wireless"
```
---

## v0.3 — FAISS-based Semantic Review Search (Completed)

**Goal:** Add a semantic retrieval layer for product reviews using sentence-transformer embeddings + FAISS HNSW.

**What’s included**
- `scripts/generate_embeddings.py` — streaming, memory-safe embedding generation for `data/out/reviews_small.csv` using `sentence-transformers/all-MiniLM-L6-v2`.
- `scripts/build_faiss_index.py` — builds an HNSW FAISS index (`IndexHNSWFlat`) and writes `data/faiss/reviews_hnsw.index` and a small JSON config.
  - Editable build-time HNSW params at top of file: `M` (connectivity) and `ef_construction` (build depth).
- `scripts/query_faiss.py` — CLI for semantic queries with:
  - `--k` (number of results),
  - `--efSearch` (runtime efSearch to trade recall/latency),
  - `--asin` (optional per-product filtering with oversampling).
- `data/faiss/faiss_metadata.db` — SQLite mapping `faiss_id → asin,reviewText,summary,unixReviewTime`.
- `scripts/e2e_smoke_test.sh` and `scripts/e2e_test.py` — smoke & programmatic E2E checks (Cockroach, Elasticsearch, FAISS artifacts, sample queries).
- Manifest & reproducibility notes:
  - Model: `sentence-transformers/all-MiniLM-L6-v2` (384d)
  - Index: `IndexHNSWFlat`, default `M=32`, `ef_construction=200`
  - Runtime `efSearch` recommended: `200` for demos

**Notes / Best practices**
- Do **not** commit `data/faiss/*` (index/embeddings) to git. Ensure `data/faiss/` is in `.gitignore`. Add `data/faiss/README.md` containing model & index metadata if you want to distribute reproducible artifacts.
- The FAISS pipeline is deterministic (queries produce identical results across runs).
- For per-product retrieval, `--asin` in `query_faiss.py` performs oversampling + filter; for high traffic, precompute per-asin id lists.

**How to run (short)**
```bash
# generate embeddings
python scripts/generate_embeddings.py

# build FAISS index
python scripts/build_faiss_index.py

# query
python scripts/query_faiss.py "long battery life" --k 5 --efSearch 200

- v0.4 – Unified FastAPI query gateway

