## 25-Week Development Roadmap

### Phase 1: Core Tenancy, Auth & Skeleton Ingestion
* **Timeline:** Weeks 1–4 (Dec 1 – Dec 28, 2026)
* **Deliverables:** Establish monorepo layout, auth, organization/workspace RBAC, `TenantPool` data-plane abstraction, fail-closed routing, MinIO upload, MongoDB metadata, and an end-to-end async worker stub with automated isolation tests.

### Phase 2: OCR, Parsing, Extraction & Validation
* **Timeline:** Weeks 5–9 (Dec 29, 2026 – Feb 1, 2027)
* **Deliverables:** Implement multi-format ingestion (PDF, DOCX, XLSX, CSV, images), PyMuPDF/pdfplumber/Tesseract/PaddleOCR pipelines, structured JSON extraction via Ollama, rule/financial validation, retries, and dead-letter queues.

### Phase 3: Hybrid Retrieval & Verifiable RAG
* **Timeline:** Weeks 10–14 (Feb 2 – Mar 8, 2027)
* **Deliverables:** Build chunking, FastEmbed/Sentence Transformers indexing in Qdrant with tenant/workspace filters, BM25 + dense RRF, cross-encoder reranking, citation verification, and uncertainty flagging.

### Phase 4: Human Review, Quotas & TenantBridge
* **Timeline:** Weeks 15–19 (Mar 9 – Apr 12, 2027)
* **Deliverables:** Add review/correction workflows, curated evaluation datasets, usage metering, quota enforcement, audit logging, and `TenantBridge` provisioning plus controlled pool-to-bridge migration.

### Phase 5: Enterprise Silo, Hardening & Benchmarks
* **Timeline:** Weeks 20–25 (Apr 13 – May 23, 2027)
* **Deliverables:** Implement `TenantSilo` isolation boundaries, run k6/Locust load and chaos/stuck-job tests, benchmark ML metrics (CER/WER, F1, nDCG@K, faithfulness, ECE), and finalize observability/CI/CD.
