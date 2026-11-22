# documentMind
# docIS — Document Intelligence System

An AI-powered document intelligence platform for uploading, processing, extracting, reviewing, and searching information across PDFs, scanned documents, and spreadsheets.

## Features

- **Document Management** — Upload, organize, view, and delete documents.
- **Document Processing** — Extract text using PDF parsing and OCR.
- **Invoice Intelligence** — Extract invoice data, validate fields, detect issues, and review corrections.
- **Search** — Find document content using keyword and semantic search.
- **Background Processing** — Run asynchronous jobs with progress tracking and retry support.
- **Evaluation** — Measure extraction quality against a curated dataset.

## Tech Stack

| Layer | Technologies |
|---|---|
| Backend | Node.js, TypeScript, Express.js|
| Database | PostgreSQL, Prisma ORM, Prisma Migrate |
| API | REST API, Zod (validation), OpenAPI/Swagger|
| Processing |OCR, PDF parsing, spreadsheet extraction |
| Background Jobs | BullMQ, Redis |
| Search | Text chunking, embeddings, semantic search, Qdrant |
| Frontend | React, TypeScript, Vite |
| Styling | Tailwind CSS |
| Data Fetching | TanStack Query |
| Testing | Vitest, React Testing Library, Supertest, Playwright |
| Code Quality |	ESLint, Prettier, TypeScript |
| Infrastructure | Docker Compose |
|CI/CD|	GitHub Actions|

# Project Structure
```text
docIS/
├── backend/
│   ├── src/
│   │   ├── config/              # Environment variables and app configuration
│   │   ├── api/
│   │   │   ├── routes/           # Express API routes
│   │   │   ├── controllers/      # HTTP request/response handlers
│   │   │   ├── middlewares/      # Authentication, validation, error handling
│   │   │   └── validators/       # Zod request validation schemas
│   │   │
│   │   ├── database/
│   │   │   ├── prisma/           # Prisma client initialization
│   │   │   └── repositories/     # Database access layer
│   │   │
│   │   ├── modules/
│   │   │   ├── documents/        # Document upload and management
│   │   │   ├── extraction/       # Structured data extraction
│   │   │   ├── processing/       # OCR, PDF and spreadsheet processing
│   │   │   ├── search/           # Chunking, embeddings and semantic search
│   │   │   └── evaluations/      # Extraction evaluation logic
│   │   │
│   │   ├── services/             # Shared business logic
│   │   ├── storage/              # Local and cloud file storage
│   │   ├── queues/
│   │   │   ├── producers/        # Background job producers
│   │   │   ├── workers/          # BullMQ job processors
│   │   │   └── definitions/      # Queue and job definitions
│   │   │
│   │   ├── utils/                # Shared utility functions
│   │   ├── types/                # Shared TypeScript types
│   │   ├── app.ts                # Express application setup
│   │   └── server.ts             # HTTP server entry point
│   │
│   ├── prisma/
│   │   ├── schema.prisma         # Database schema
│   │   ├── migrations/           # Prisma migrations
│   │   └── seed.ts               # Database seed data
│   │
│   ├── tests/
│   │   ├── unit/
│   │   ├── integration/
│   │   └── api/
│   │
│   ├── evaluation/
│   │   ├── datasets/             # Ground-truth datasets
│   │   ├── fixtures/             # Sample documents
│   │   └── reports/              # Evaluation results
│   │
│   ├── docs/                     # Backend architecture and API docs
│   ├── package.json
│   ├── tsconfig.json
│   └── .env.example
│
├── frontend/
│   ├── src/
│   │   ├── api/
│   │   │   ├── client.ts         # HTTP client configuration
│   │   │   ├── endpoints/        # API endpoint definitions
│   │   │   └── hooks/            # TanStack Query hooks
│   │   │
│   │   ├── components/
│   │   │   ├── ui/               # Reusable UI components
│   │   │   ├── layout/           # Application layouts
│   │   │   └── documents/        # Document-specific components
│   │   │
│   │   ├── pages/                # Application pages
│   │   ├── features/             # Feature-specific components and logic
│   │   ├── hooks/                # Custom React hooks
│   │   ├── lib/                  # Shared utilities and configurations
│   │   ├── types/                # Frontend TypeScript types
│   │   ├── styles/               # Global styles and Tailwind CSS
│   │   ├── test/
│   │   │   ├── fixtures/
│   │   │   └── setup.ts
│   │   ├── App.tsx
│   │   └── main.tsx
│   │
│   ├── public/
│   ├── index.html
│   ├── package.json
│   ├── tsconfig.json
│   └── vite.config.ts
│
├── docker/
│   ├── backend.Dockerfile
│   ├── frontend.Dockerfile
│   └── docker-compose.yml
│
├── scripts/                      # Development and deployment scripts
├── docs/                          # System architecture and documentation
├── .github/
│   └── workflows/                 # GitHub Actions CI/CD
│
├── .gitignore
├── package.json                   # Optional monorepo root scripts
├── README.md
└── .env.example
```
## Application Pages
- Documents (/) — Browse, upload, filter, and manage documents.
- Document Intelligence (/documents/:id) — View extracted content, review invoice data, validate structured fields, and inspect processing history.
- Search (/search) — Search across document content using keyword and semantic search.

## Development
The backend and frontend are maintained as separate applications within the project.
- Backend dependencies and scripts are managed with npm.
- Backend development uses Node.js, Express.js, and TypeScript.
- Frontend dependencies and scripts are managed with npm.
- Frontend development uses React, TypeScript, Vite, and Tailwind CSS.
- PostgreSQL and Prisma ORM manage database access and schema migrations.
- BullMQ and Redis handle asynchronous document-processing jobs.
- Docker Compose provides the supporting infrastructure.
- Zod handles request validation and schema definitions.
Refer to backend/docs/, frontend/, and the respective package.json files for detailed setup and development instructions.
## Testing
- Backend: Vitest, Supertest — unit, API, and integration tests.
- Frontend: Vitest and React Testing Library.
- End-to-End: Playwright — application workflows and user interactions.
- Evaluation: Ground-truth datasets and extraction evaluation reports for measuring document extraction accuracy.
## Project Goal
Make unstructured business documents easier to process and understand by combining automated document extraction, structured data validation, human-in-the-loop review, and intelligent search in one platform.
