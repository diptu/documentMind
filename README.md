# EvidenceOS

> **Multi-Tenant SaaS for Reliable AI Document Intelligence**
>
> **INGEST → EXTRACT → RETRIEVE → VERIFY → TRUST**
>
> Multi-tenant by design. Evidence-grounded by default. Built for
> reliable AI at scale.

EvidenceOS is a multi-tenant SaaS platform that transforms unstructured
business documents into structured information, searchable evidence, and
verifiable AI-generated insights.

Organizations can securely manage their teams and workspaces, upload
business documents, extract and validate information, search across
permitted content, and ask questions using an AI assistant that grounds
its answers in source evidence.

> **Core principle:** Every important AI-generated claim should be
> traceable to verifiable evidence in its source documents, and every
> tenant must remain isolated from other tenants' data.

EvidenceOS explores the intersection of **multi-tenant SaaS engineering,
distributed systems, applied machine learning, information retrieval,
and reliable AI infrastructure.**

------------------------------------------------------------------------

## Product Goals

EvidenceOS is designed around five product goals:

1.  **Secure collaboration:** Organizations manage their own users,
    workspaces, documents, and permissions.
2.  **Document intelligence:** Convert PDFs, office documents,
    spreadsheets, and images into structured, validated information.
3.  **Evidence-first retrieval:** Find relevant source passages using
    lexical and semantic search.
4.  **Verifiable AI:** Generate answers with traceable citations and
    flag unsupported claims.
5.  **SaaS operations:** Manage tenant provisioning, plans, quotas,
    usage, audit history, and operational health.

## Key Features

### Multi-Tenant SaaS

-   **Organization onboarding:** Create and configure customer
    organizations (tenants).
-   **Membership management:** Invite users, manage memberships, and
    assign roles.
-   **Workspace collaboration:** Organize documents and team workflows
    into workspaces.
-   **Role-based access control:** Enforce organization and workspace
    permissions.
-   **Tenant isolation:** Scope application data, files, jobs, search,
    caches, and AI context to authorized tenants.
-   **Usage metering:** Track storage, document processing, and AI
    usage.
-   **Subscription foundations:** Support plan entitlements, quotas, and
    subscription lifecycle management.
-   **Audit history:** Record security-sensitive and administrative
    actions.

### Document Intelligence

-   **Multi-format ingestion:** Process PDF, DOCX, XLSX, CSV, JPG, PNG,
    and TIFF files.
-   **OCR and parsing:** Extract text, tables, and document structure.
-   **Structured extraction:** Identify entities, fields, and business
    information.
-   **Automated validation:** Detect missing fields, duplicates,
    financial inconsistencies, and low-confidence extractions.
-   **Document comparison:** Identify discrepancies and contradictions
    across documents.
-   **Human review:** Allow authorized users to correct, review, and
    approve extracted information.

### Evidence-Grounded AI

-   **Hybrid RAG:** Combine lexical and semantic retrieval using
    Reciprocal Rank Fusion (RRF).
-   **Reranking:** Improve evidence selection with cross-encoder models.
-   **Grounded generation:** Generate answers, summaries, reports, and
    business drafts from authorized evidence.
-   **Citation verification:** Check whether cited passages support
    generated claims.
-   **Evidence inspection:** Link answers to source documents, pages,
    and passages.
-   **Uncertainty handling:** Flag insufficient or conflicting evidence
    instead of presenting unsupported output as fact.

### Distributed Processing

-   Asynchronous document-processing jobs.
-   Idempotent task execution.
-   Retry with exponential backoff and jitter.
-   Dead-letter queues and failure recovery.
-   Worker health monitoring and stuck-job recovery.
-   Backpressure and admission control.
-   Independent scaling of application and ML workloads.

------------------------------------------------------------------------

## SaaS Roles and Access Model

EvidenceOS separates platform administration, organization membership,
and workspace collaboration.

  -----------------------------------------------------------------------
  Scope                   Role                    Responsibilities
  ----------------------- ----------------------- -----------------------
  Platform                Platform Admin          Manage tenants, plans,
                                                  and platform operations

  Organization            Owner                   Manage organization,
                                                  administrators,
                                                  subscription, and
                                                  members

  Organization            Admin                   Manage members,
                                                  settings, and
                                                  workspaces

  Workspace               Manager                 Manage workspace
                                                  members and content

  Workspace               Member                  Upload, search, and
                                                  work with permitted
                                                  documents

  Workspace               Reviewer                Review and approve
                                                  permitted AI outputs
  -----------------------------------------------------------------------

The initial implementation should use a small, understandable role set.
More granular permissions can be introduced as product requirements
evolve.

### Tenant Isolation Principles

-   Resolve tenant context from the authenticated user's verified
    membership.
-   Never trust a client-supplied `tenantId` as proof of access.
-   Apply tenant and workspace authorization on the server for every
    tenant-owned operation.
-   Scope database queries, object access, jobs, vector searches, cache
    keys, and conversation history.
-   Re-check membership and permissions when processing asynchronous
    jobs.
-   Test cross-tenant and cross-workspace access at API and service
    boundaries.
-   Treat tenant isolation as a security invariant, not a frontend
    feature.

------------------------------------------------------------------------

## System Architecture

EvidenceOS uses a tenant-aware application architecture with
independently deployable ML processing services. The initial
implementation favors a modular NestJS backend and separate Python
workers. Event-driven infrastructure can be introduced as processing
requirements grow.

``` mermaid
flowchart TD
    U[Users from Multiple Organizations] --> WEB[Next.js SaaS Frontend]
    WEB --> API[NestJS API]
    API --> AUTH[Authentication]
    AUTH --> CTX[Tenant Context and RBAC]
    CTX --> APP[Tenant-Aware Application Modules]

    APP --> ORG[Organizations and Workspaces]
    APP --> BILL[Plans and Usage Metering]
    APP --> DOC[Document Management]
    APP --> AUDIT[Audit Logging]

    DOC --> QUEUE[Job Queue / Kafka or Redpanda]
    QUEUE --> WORKERS[Python OCR and Extraction Workers]

    WORKERS --> OBJ[MinIO Object Storage]
    WORKERS --> DB[MongoDB]
    WORKERS --> VEC[Qdrant]

    APP --> RET[Permission-Aware Retrieval]
    RET --> DB
    RET --> VEC
    RET --> RERANK[Hybrid Ranking and Reranking]
    RERANK --> LLM[Local LLM - Ollama]
    LLM --> VERIFY[Citation Verification]
    VERIFY --> RESULT[Grounded Tenant-Scoped Response]

    CACHE[Redis Tenant-Aware Cache] -.-> APP
    OBS[OpenTelemetry / Prometheus / Grafana] -.-> API
    OBS -.-> WORKERS
```

### Architecture Principles

-   Tenant context is established and validated at the backend boundary.
-   Authorization is enforced in application services and data-access
    layers.
-   Application data and search results are tenant- and
    permission-scoped.
-   Original documents and derived artifacts have controlled access
    paths.
-   Asynchronous jobs carry validated tenant and workspace context.
-   CPU-intensive and ML workloads can scale independently.
-   Processing is observable, retryable, and idempotent.
-   Local-first LLM inference avoids mandatory dependence on external
    LLM APIs.
-   The system starts simple and introduces distributed infrastructure
    when justified.

------------------------------------------------------------------------

## **Technology Stack**

| Layer               | Technology                         |
| ------------------- | ---------------------------------- |
| Frontend            | Next.js, React, TypeScript         |
| Backend             | NestJS, Node.js, TypeScript        |
| API                 | REST, WebSockets (where needed)    |
| ML Services         | Python, FastAPI                    |
| LLM Inference       | Ollama, Configurable Local LLMs    |
| Primary Database    | MongoDB                            |
| Vector Database     | Qdrant                             |
| Message Broker      | Kafka / Redpanda (Planned)         |
| Cache               | Redis                              |
| Object Storage      | MinIO                              |
| OCR                 | Tesseract, PaddleOCR               |
| Document Parsing    | PyMuPDF, pdfplumber, python-docx   |
| Embeddings          | FastEmbed, Sentence Transformers   |
| Reranking           | Cross-Encoder Models               |
| ML Framework        | PyTorch, Hugging Face              |
| Experiment Tracking | MLflow                             |
| Testing             | Jest, pytest, Hypothesis           |
| Load Testing        | k6, Locust                         |
| Observability       | OpenTelemetry, Prometheus, Grafana |
| Infrastructure      | Docker, Docker Compose             |
| CI/CD               | GitHub Actions                     |

Technology choices are implementation targets and may evolve as the
system is benchmarked.

------------------------------------------------------------------------

## Core Modules

  -----------------------------------------------------------------------
  Module                              Responsibilities
  ----------------------------------- -----------------------------------
  Identity                            Authentication, sessions, password
                                      reset

  Tenant Management                   Tenant provisioning, tenant
                                      context, organization settings

  Membership & RBAC                   Invitations, memberships, roles,
                                      permissions

  Workspace Management                Workspace creation, membership,
                                      access policies

  Subscription & Usage                Plans, entitlements, quotas, usage
                                      metering

  Audit                               Security-sensitive and
                                      administrative event history

  Document Ingestion                  Upload, validation, metadata,
                                      storage

  OCR & Extraction                    Text, tables, entities, structured
                                      fields

  Validation                          Schema checks, financial
                                      calculations, anomaly detection

  Indexing                            Chunking, embeddings, vector
                                      indexing

  Retrieval                           Keyword search, semantic search,
                                      hybrid ranking

  Generation                          Evidence-grounded answers and
                                      document drafts

  Verification                        Citation resolution and
                                      claim-support checks

  Comparison                          Document differences and
                                      reconciliation

  Feedback                            Human corrections, approvals,
                                      review history

  Evaluation                          Model benchmarks and performance
                                      analytics
  -----------------------------------------------------------------------

------------------------------------------------------------------------

## Multi-Tenancy and Data Isolation

### Initial tenancy model

The initial SaaS deployment uses shared application and database
infrastructure with logical tenant isolation. This keeps development and
operations manageable while preserving a path toward dedicated resources
for enterprise customers.

  -----------------------------------------------------------------------
  Layer                   Initial strategy        Future scaling option
  ----------------------- ----------------------- -----------------------
  Application             Shared Next.js and      Independently scaled
                          NestJS applications     services

  MongoDB                 Shared database;        Dedicated database for
                          tenant-scoped records   selected enterprise
                          and queries             tenants

  Qdrant                  Shared collections with Separate collections or
                          mandatory tenant and    clusters
                          permission filters      

  MinIO                   Tenant-scoped object    Dedicated buckets or
                          keys and controlled     storage
                          access                  

  Redis                   Tenant-aware cache keys Dedicated instances

  Kafka                   Shared topics with      Dedicated topics or
                          tenant identity in      clusters
                          validated event context 

  Workers                 Shared worker pools     Dedicated worker pools
                          with tenant-aware jobs  
  -----------------------------------------------------------------------

### Tenant-owned data

Every tenant-owned record should carry an explicit tenant relationship.
Workspace-owned resources should also carry a workspace relationship
where applicable.

Example document record:

``` typescript
interface DocumentRecord {
  tenantId: string;
  workspaceId: string;
  documentId: string;
  filename: string;
  status: "PENDING" | "PROCESSING" | "COMPLETED" | "FAILED";
  createdAt: Date;
}
```

Example MinIO object key:

``` text
tenants/{tenantId}/documents/{documentId}/original.pdf
```

Example Qdrant payload:

``` json
{
  "tenant_id": "tenant_123",
  "workspace_id": "workspace_456",
  "document_id": "doc_789",
  "chunk_id": "chunk_001"
}
```

These examples illustrate the data model; identifiers must be resolved
and authorized by trusted backend services.

### Important security note

A tenant ID in a database record, object key, or vector payload does not
by itself provide isolation. The application must authorize the
requesting user and enforce the corresponding tenant and workspace
filters on every access path. Use defense in depth, including scoped
repositories, object access controls, service-level checks, and
automated isolation tests.

------------------------------------------------------------------------

## Subscription Plans and Usage Metering

EvidenceOS is intended to support plan-based SaaS access. The initial
MVP can use manually assigned or simulated plans before payment
processing is integrated.

  Capability                 Free            Pro            Enterprise
  -------------------------- --------------- -------------- ------------
  Workspaces                 Limited         Expanded       Custom
  Members                    Limited         Expanded       Custom
  Document storage           Limited quota   Higher quota   Custom
  Processing jobs            Limited quota   Higher quota   Custom
  AI requests                Limited quota   Higher quota   Custom
  Audit history              Basic           Advanced       Advanced
  Dedicated infrastructure   No              No             Optional

These are proposed product tiers, not published prices or finalized
entitlements.

Suggested usage dimensions:

-   Stored document bytes.
-   Number of documents.
-   Processing jobs and processing time.
-   AI requests and token usage.
-   Vector storage.
-   Active members and workspaces.

Quota checks should happen before accepting expensive operations. Usage
records should be auditable and safe against duplicate job events.

------------------------------------------------------------------------

## Document Ingestion Pipeline

``` mermaid
flowchart TD
    A[Authenticated User Upload] --> B[Resolve Tenant and Workspace]
    B --> C[Authorize Upload and Check Quota]
    C --> D[Validate File]
    D --> E[Store Original in MinIO]
    E --> F[Persist Metadata in MongoDB]
    F --> G[Create Processing Job]
    G --> H[Queue Job]
    H --> I[Python Processing Worker]
    I --> J[OCR and Parsing]
    J --> K[Structured Extraction]
    K --> L[Validation and Confidence]
    L --> M[Persist Tenant-Scoped Results]
    M --> N[Update Job Status]
    N --> O[Notify Authorized Users]
```

Processing jobs must carry a validated tenant and workspace context.
Workers must not rely on untrusted tenant identifiers from arbitrary
event payloads. They should load the job, verify its ownership and
authorization context, and ensure all outputs are written to the correct
tenant scope.

------------------------------------------------------------------------

## Retrieval & RAG Pipeline

EvidenceOS combines lexical and semantic retrieval to improve evidence
discovery while enforcing tenant and workspace permissions.

``` mermaid
flowchart TD
    A[Authenticated User Query] --> B[Resolve Tenant and Workspace Permissions]
    B --> C[Query Understanding]
    C --> D[BM25 Lexical Search]
    C --> E[Qdrant Dense Vector Search]
    D --> F[Reciprocal Rank Fusion]
    E --> F
    F --> G[Permission and Metadata Filtering]
    G --> H[Cross-Encoder Reranking]
    H --> I[Evidence Selection]
    I --> J[Local LLM Generation]
    J --> K[Citation Verification]
    K --> L[Grounded Response]
```

Key retrieval experiments:

-   BM25 versus dense retrieval.
-   Hybrid retrieval with RRF.
-   Fixed-size versus semantic chunking.
-   Embedding model comparisons.
-   Cross-encoder reranking.
-   Query rewriting and decomposition.
-   Retrieval quality and latency under tenant filters.
-   Permission-filtering correctness under mixed-tenant workloads.

**Security invariant:** Unauthorized chunks must never enter the model
context, even temporarily. Filtering only the final response is
insufficient.

------------------------------------------------------------------------

## Local LLM Architecture

EvidenceOS uses locally hosted language models through **Ollama**,
avoiding mandatory dependence on external LLM APIs.

Planned capabilities:

-   Structured JSON extraction.
-   Evidence-grounded question answering.
-   Contract and invoice summarization.
-   Document comparison.
-   Claim verification.
-   Configurable model routing and fallback.
-   Quantized model inference.
-   Response caching with tenant-aware cache keys.
-   Token-budget and usage management.

The model layer will remain provider-agnostic to support future
inference backends. Local model selection will depend on available
hardware, latency, quality, and memory requirements.

------------------------------------------------------------------------

## Human Review and Feedback

``` mermaid
flowchart TD
    A[AI Prediction] --> B[Authorized Human Review]
    B --> C[Correction or Approval]
    C --> D[Store Tenant-Scoped Feedback]
    D --> E[Data Quality Checks]
    E --> F[Versioned Evaluation Dataset]
    F --> G[Offline Model Evaluation]
    G --> H[Candidate Release and Regression Tests]
```

Feedback must retain tenant ownership, actor identity, timestamps, and
relevant model or prompt versions. Do not automatically retrain models
on every user edit. Approved corrections should pass data-quality checks
before entering evaluation or training datasets.

------------------------------------------------------------------------

## Distributed Systems & Reliability

Engineering challenges addressed:

-   Asynchronous document processing.
-   Kafka-based event-driven communication when justified.
-   Idempotent task execution.
-   Retry with exponential backoff and jitter.
-   Dead-letter queues.
-   Worker health monitoring.
-   Stuck-job recovery.
-   Backpressure and admission control.
-   Per-tenant quotas and fair resource allocation.
-   Independent worker scaling.
-   Distributed tracing and observability.
-   Tenant-aware failure handling and operational metrics.

Tenant identity and authorization context must be propagated through
every asynchronous boundary. A worker must not be able to write results
to a tenant simply because an event contains that tenant's ID.

------------------------------------------------------------------------

## Security and Privacy

Security is a core product requirement, not a final-stage enhancement.

Planned controls:

-   Verified authentication and server-side authorization.
-   Organization and workspace RBAC.
-   Tenant-scoped database access.
-   Controlled object-storage access.
-   Permission-aware vector retrieval.
-   Tenant-aware caches and background jobs.
-   Audit logging for sensitive actions.
-   Secure secrets and environment configuration.
-   Rate limits and resource quotas.
-   Document retention, export, and deletion workflows.
-   Protection against cross-tenant data leakage through prompts,
    retrieval, logs, and caches.

### Tenant isolation test cases

-   A member of Tenant A cannot read Tenant B's document by guessing its
    ID.
-   Tenant A cannot retrieve Tenant B's vector records or snippets.
-   Tenant A cannot access Tenant B's conversations or generated
    outputs.
-   A worker cannot process or persist a job under the wrong tenant.
-   A cached response cannot be returned to another tenant.
-   A removed member loses access to the organization and its
    workspaces.
-   Workspace restrictions are enforced within the same tenant.
-   Tenant deletion removes or schedules deletion of associated records,
    objects, vectors, and cached data.

------------------------------------------------------------------------

## ML Evaluation Framework

EvidenceOS includes an offline benchmarking framework.

  -----------------------------------------------------------------------
  Component                           Evaluation Metrics
  ----------------------------------- -----------------------------------
  OCR                                 Character Error Rate, Word Error
                                      Rate

  Extraction                          Precision, Recall, F1

  Validation                          Precision, Recall

  Retrieval                           Recall@K, MRR, nDCG@K

  Reranking                           nDCG improvement

  Generation                          Faithfulness, Answer Correctness

  Citations                           Citation Precision, Evidence
                                      Coverage

  Confidence                          Brier Score, Expected Calibration
                                      Error

  Processing                          Throughput, P50/P95/P99 Latency

  Feedback                            Correction and Acceptance Rates

  SaaS Isolation                      Cross-tenant and cross-workspace
                                      authorization test results

  Usage Metering                      Quota accuracy and duplicate-event
                                      handling
  -----------------------------------------------------------------------

Potential datasets:

-   FUNSD
-   SROIE
-   CORD
-   DocVQA
-   CUAD
-   MS MARCO

Respect dataset licenses and document any transformations. Benchmark
results should be reproducible and should not mix tenant-isolation tests
with claims about model quality.

------------------------------------------------------------------------

## Observability and Operations

Monitor the platform at both platform and tenant levels while avoiding
exposure of sensitive document content in telemetry.

Suggested operational metrics:

-   Request rate, errors, and latency.
-   Upload and processing throughput.
-   Queue wait time and job age.
-   OCR and extraction failures.
-   Retry and dead-letter queue rates.
-   Retrieval latency and result counts.
-   LLM inference latency and token usage.
-   Storage and processing usage by tenant.
-   Quota rejections.
-   Worker health and resource utilization.

Use tenant identifiers in operational metrics only with care. Avoid
high-cardinality metric labels; use logs and traces with appropriate
access controls for detailed tenant-level investigation.

------------------------------------------------------------------------

## Project Structure

``` text
evidenceos/
│
├── apps/
│   ├── web/                      # Next.js SaaS frontend
│   └── api/                      # NestJS application/API
│
├── services/
│   ├── tenant/                   # Tenant provisioning and context
│   ├── identity/                 # Authentication and authorization
│   ├── billing/                  # Plans, subscriptions, usage
│   ├── ingestion/                # Tenant-aware document ingestion
│   ├── ocr/                      # OCR processing
│   ├── extraction/               # Structured extraction
│   ├── validation/               # Business rule validation
│   ├── retrieval/                # Permission-aware hybrid retrieval
│   ├── generation/               # Local LLM integration
│   ├── verification/             # Citation verification
│   ├── feedback/                 # Human feedback
│   └── audit/                    # Audit history
│
├── ml/
│   ├── embeddings/
│   ├── reranking/
│   ├── calibration/
│   └── evaluation/
│
├── packages/
│   ├── shared-types/
│   ├── api-contracts/
│   └── tenant-context/
│
├── infrastructure/
│   ├── docker/
│   ├── kafka/
│   ├── mongodb/
│   ├── qdrant/
│   ├── minio/
│   └── monitoring/
│
├── benchmarks/
├── tests/
│   ├── unit/
│   ├── integration/
│   ├── e2e/
│   └── tenant-isolation/
│
├── docs/
│   ├── architecture/
│   ├── security/
│   ├── system-design/
│   └── decisions/
│
├── docker-compose.yml
└── README.md
```

The directory structure is a target organization. Start with the modules
needed for the MVP and split services when independent deployment or
scaling provides a concrete benefit.


------------------------------------------------------------------------

## Getting Started

### Prerequisites

-   Node.js 20+
-   Python 3.11+
-   Docker and Docker Compose
-   Ollama (for local LLM inference)
-   Git

### 1. Clone the repository

``` bash
git clone https://github.com/YOUR_USERNAME/evidenceos.git
cd evidenceos
```

### 2. Configure environment

``` bash
cp .env.example .env
```

Configure secrets, database credentials, object storage, and local
service endpoints. Never commit production secrets.

### 3. Start infrastructure

``` bash
docker compose up -d
```

### 4. Start a local LLM

Install Ollama and pull a model compatible with your hardware:

``` bash
ollama pull qwen2.5:3b
```

The model is an initial example, not a fixed production requirement.
Validate memory use, latency, and answer quality on your machine.

### 5. Start the backend

``` bash
cd apps/api
npm install
npm run start:dev
```

### 6. Start the frontend

``` bash
cd apps/web
npm install
npm run dev
```

Detailed setup instructions, environment variables, migrations, and
model requirements will be maintained as implementation progresses.

------------------------------------------------------------------------

## Engineering Priorities

EvidenceOS is designed to demonstrate practical engineering across:

-   **SaaS Engineering:** Multi-tenancy, organization management, RBAC,
    subscriptions, quotas.
-   **Software Engineering:** TypeScript, NestJS, API design, testing,
    modular architecture.
-   **Distributed Systems:** Asynchronous jobs, event-driven
    communication, fault tolerance, idempotency.
-   **Machine Learning:** OCR, embeddings, reranking, confidence
    calibration, evaluation.
-   **AI Engineering:** Local LLMs, RAG, citation verification, grounded
    generation.
-   **Data Engineering:** Document pipelines, NoSQL storage, indexing,
    data quality.
-   **Security Engineering:** Tenant isolation, workspace permissions,
    auditability, data lifecycle.
-   **Performance Engineering:** Benchmarking, load testing, caching,
    inference optimization.

## Future Engineering and Research Directions

-   Dedicated infrastructure for enterprise tenants.
-   Confidence-aware selective prediction.
-   Adaptive retrieval and query routing.
-   Layout-aware document chunking.
-   Efficient embedding and reranking models.
-   Quantized local LLM inference.
-   Dynamic batching and model optimization.
-   Feedback-driven retrieval improvement.
-   Evidence conflict detection.
-   Multimodal document understanding.
-   Tenant-aware resource scheduling and fair usage policies.

------------------------------------------------------------------------

## Author

**Nazmul Alam**

Computer Science & Engineering \| Applied Machine Learning \| ML Systems
\| AI Infrastructure

-   GitHub: https://github.com/diptu
-   Portfolio: https://diptuverse.com

------------------------------------------------------------------------

