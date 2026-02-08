## Project Status
- A production-style, distributed e-commerce semantic search platform built incrementally with a focus on scalability and correctness.


### v0.1 – Data Ingestion & Distributed Storage (Completed)
- Dockerized preprocessing pipeline for large-scale Amazon datasets
- 3-node CockroachDB cluster running locally
- Schema design for products and reviews
- Bulk data ingestion using CockroachDB IMPORT
- Verified replication and range distribution across nodes

### v0.2 – Product Search Infrastructure (Completed)

- Elasticsearch-based BM25 product search
- Official Elasticsearch (ARM64-compatible) running via Docker Compose
- Product index with optimized mappings
- ~10,000 products indexed successfully
- Verified relevance ranking using multi_match queries and field boosting

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

