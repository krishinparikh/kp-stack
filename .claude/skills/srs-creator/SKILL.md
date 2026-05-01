---
name: srs-creator
description: "Use when the user asks to write, create, or draft an SRS (Software Requirements Specification) or system architecture document. Translates a PRD into a complete technical blueprint."
---

<overview>
Write SRS documents that translate a PRD into a complete technical blueprint. The SRS should give any engineer enough detail to start building without further design discussions. It defines **how** to build what the PRD describes.
</overview>

## Prerequisites

Before writing, you MUST read the project's PRD (`docs/PRD.md`). The SRS is derived from it. If no PRD exists, tell the user to create one first (or use the `prd-creator` skill).

Also ask the user about any tech stack preferences or constraints they have. If they have none, make opinionated choices and justify them.

## Process

1. Read the PRD thoroughly
2. Identify all entities, relationships, and data flows from the user stories and core features
3. Choose technologies for each layer
4. Design the database schema from the entities
5. Map core features to API routes
6. Define the application structure (file tree)

## SRS Document Structure

Write the document in this exact order. 

### 1. Tech Stack
A table with two columns: **Layer** and **Technology**. Cover every layer of the system:
- Frontend (framework, UI library, styling)
- Backend API (language, framework)
- Database
- Any specialized frameworks (agent frameworks, ML pipelines, etc.)
- External services (PDF parsing, search APIs, etc.)
- Containerization / deployment

Be specific — include the actual library names, not just categories. For example: "React + Vite + shadcn/ui + Tailwind CSS", not "JavaScript frontend."

### 2. Architecture
Describe the system components and how they connect. Cover:
- The frontend: what framework serves it, how it communicates with the backend, and any client-side state management
- The backend: its internal layering (routes, services, data access), how requests flow through these layers, and where business logic lives
- External dependencies: database, third-party APIs, file storage, message brokers
- Communication protocols between layers (REST, SSE, WebSocket, etc.) and why each was chosen
- How async/background work is triggered and managed (e.g., pipeline kicked off by an API call, runs in a background thread, publishes status via Redis)
- Data flow: trace a typical request end-to-end from user action through frontend → API → backend processing → database → response

### 3. Database Schema
For each table/collection:
- Table name as a sub-heading
- One-sentence description of what it stores
- A table with columns: **Column**, **Type**, **Notes**
- Mark PKs and FKs explicitly
- Include status enums inline (e.g., `parsing` → `specialists` → `complete`)
- Use appropriate types (UUID, TEXT, JSONB, etc.)

Derive tables directly from the PRD's entities. Every noun that gets created, stored, or referenced should map to a table or a column.

### 4. Application Structure
A file tree showing every directory and file in the project. Use this format:
```
project-name/
├── docker-compose.yml
├── backend/
│   ├── app/
│   │   ├── __init__.py
│   │   ├── routes/
│   │   │   └── upload.py          # POST /api/upload — description
│   │   └── ...
└── frontend/
    └── src/
        └── ...
```

Rules:
- Maintain separation of concerns — break code into folders and separate files when opportune. Constants, sub-components, utility modules, config, and types should each live in their own files rather than being inlined into larger files. Prefer many small, focused files over few large ones.
- Include inline comments (`# description`) for non-obvious files
- Group by feature/domain, not by type, where possible
- Show the full tree — don't use "..." unless a directory contains only standard boilerplate
- Frontend pages should map 1:1 to the PRD's core features
- Each page gets its own `components/` subdirectory for page-specific components

### 5. API Routes
A table with columns: **Method**, **Path**, **Description**. Cover every endpoint the frontend needs. Prefix all routes consistently (e.g., `/api/`). Include:
- CRUD endpoints for each entity
- Any streaming/SSE endpoints
- File upload endpoints

### 6. Pipelines
If the application has major pipelines (data processing, AI agent workflows, ETL, multi-step async processes, etc.), describe each one here:
- Name the pipeline and explain what it does in one sentence
- List the stages/steps in order, describing inputs, processing, and outputs for each
- Note where stages run in parallel vs. sequentially
- Describe any conditional branching or loops
- Show the state/data that flows between stages

For each pipeline, create a separate Mermaid diagram file in `docs/pipelines/` (e.g., `docs/pipelines/debate-pipeline.md`). Use the `mermaid-diagrams` skill to produce these diagrams. Each file should contain the Mermaid diagram plus a brief legend or explanation of the nodes.

### 7. Testing
Do NOT strive for full coverage. Instead, identify the critical paths and write tests only where failure would be costly or hard to debug. Focus on:
- Core business logic (e.g., pipeline orchestration, data transformations, verdict/scoring logic)
- API route contracts (correct status codes, response shapes, error cases)
- Edge cases in parsing or data processing where silent failures are likely
- Integration points with external services (mocked)

Skip tests for: boilerplate CRUD, UI layout, straightforward getters/setters, and anything where a bug would be immediately obvious. Every test should justify its existence — if you can't explain what breakage it catches, don't write it.

Use the `python-testing-patterns` skill when implementing tests.

### 8. Environment & Config
List all environment variables the application requires. For each variable:
- Name (e.g., `DATABASE_URL`, `ANTHROPIC_API_KEY`)
- What it's for
- Whether it's required or optional
- A sensible default if one exists

Group by service if the app has multiple (backend, frontend, database, etc.).

## Writing Guidelines

- **Show code patterns**: When describing chains, state schemas, or API structures, include code snippets showing the actual pattern to follow (not full implementations).
- **Be specific about libraries**: Don't say "an ORM" — say "SQLAlchemy with Flask-Migrate for migrations."
- **Match the PRD exactly**: Every core feature in the PRD should map to frontend pages + API routes + data models here. Nothing should be invented that the PRD doesn't call for.
- **Keep schema lean**: Only model what the PRD requires. No "created_at/updated_at on everything" unless there's a stated need.

## Output

Save the SRS to `docs/SRS.md`. After writing, update `CLAUDE.md` if a new doc entry is needed.
