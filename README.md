## Project Status
- A production-style, distributed e-commerce semantic search platform built incrementally with a focus on scalability and correctness.


### v0.1 – Data Ingestion & Distributed Storage (Completed)
- Dockerized preprocessing pipeline for large-scale Amazon datasets
- 3-node CockroachDB cluster running locally
- Schema design for products and reviews
- Bulk data ingestion using CockroachDB IMPORT
- Verified replication and range distribution across nodes

### Upcoming
- v0.2 – Elasticsearch product indexing (BM25)
- v0.3 – FAISS-based semantic review search
- v0.4 – FastAPI query gateway

