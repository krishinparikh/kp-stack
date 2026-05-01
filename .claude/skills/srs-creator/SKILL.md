---
name: srs-creator
description: "Use when the user asks to write, create, or draft an SRS (Software Requirements Specification) or system architecture document. Translates a PRD into a complete technical blueprint."
---

<overview>
Write SRS documents that translate a PRD into a complete technical blueprint. The SRS should give any engineer enough detail to start building without further design discussions. It defines **how** to build what the PRD describes.
</overview>

## Prerequisites

Before writing, you MUST read the project's PRD (`docs/prd.md`). The SRS is derived from it. If no PRD exists, tell the user to create one first (or use the `prd-creator` skill).

Also read the project's `README.md` to confirm the canonical tech stack. The SRS MUST use the kp-stack defaults listed below unless there is a strong, documented reason to deviate.

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
A table with two columns: **Layer** and **Technology**. The following are **non-negotiable defaults** from the kp-stack. Always start from these and only add to them:

| Layer | Technology |
|-------|-----------|
| Framework | Next.js (App Router) with TypeScript |
| UI | shadcn/ui + Radix primitives + Tailwind CSS |
| Client State / Data Fetching | TanStack Query |
| ORM | Drizzle ORM |
| Database | Neon (serverless Postgres) |
| AI / Agents | LangChain (TypeScript) |

Add rows for any **project-specific** layers the PRD requires (file storage, search, email, payments, etc.), but never replace a default. If the PRD demands something a default doesn't cover (e.g., real-time via WebSockets), add it alongside the defaults rather than swapping them out.

### 2. Architecture
Describe the system components and how they connect. Follow the conventions in the `frontend-eng` and `backend-eng` skills for detailed rules. The SRS architecture section should cover:
- **Frontend**: Next.js App Router, RSC by default, TanStack Query, shadcn/ui
- **Backend**: Queries for reads, Actions for mutations, Route Handlers only when an HTTP endpoint is needed
- **Database**: Neon serverless Postgres via Drizzle. Schema in `src/db/schema.ts`, migrations via `drizzle-kit`
- **AI layer**: LangChain (TypeScript) for LLM orchestration. SSE via Route Handlers for streaming
- **Data flow**: Trace a typical request end-to-end from user action through to database and back
- **File structure**: Follow the canonical layout in `file-architecture.md` (in the srs-creator skill directory). Show the full project tree, expanding with project-specific files as needed

### 3. Database Schema
Reference `db-reference.md` (in this skill's directory) for the format. Derive tables directly from the PRD's entities — every noun that gets created, stored, or referenced should map to a table or a column. If the schema has many tables, extract it into a separate `docs/db.md` file and reference it from the SRS. Otherwise, keep it inline in `docs/srs.md`.

### 4. Type Safety
The Drizzle schema is the **single source of truth** for all types. Never manually redefine types that Drizzle already provides. Follow these rules:
- Use `typeof table.$inferSelect` and `typeof table.$inferInsert` to derive row types from Drizzle tables. Export these from `src/db/schema.ts`.
- Extend inferred types with `&` or `Omit`/`Pick` when you need subsets or additions — don't duplicate fields into a hand-written type.
- Use `type`, not `interface`. Interfaces encourage declaration merging and inheritance patterns that add unnecessary complexity.
- API request/response shapes should be derived from or composed of the Drizzle-inferred types so the contract between client and server stays in sync with the schema automatically.
- For frontend-only concerns (form state, UI flags), create minimal `type` aliases in the relevant component file or `src/types/` — but never re-declare database columns.

### 5. API Routes
A table with columns: **Method**, **Path**, **Description**. All routes are Next.js Route Handlers under `src/app/api/`. Include:
- CRUD endpoints for each entity
- Any streaming/SSE endpoints (for LLM responses, use SSE via `ReadableStream`)
- File upload endpoints
- Note which operations should use Server Actions instead of Route Handlers (simple form mutations, optimistic updates)

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

Use Vitest for unit/integration tests and Playwright for E2E tests.

### 8. Environment & Config
List all environment variables the application requires. Always include these kp-stack baseline variables:

| Variable | Purpose | Required |
|----------|---------|----------|
| `DATABASE_URL` | Neon Postgres connection string (used by Drizzle) | Yes |
| `NEXT_PUBLIC_APP_URL` | Public app URL for client-side use | Yes |

Add any project-specific variables (API keys for LLM providers, external services, etc.). For each variable:
- Name
- What it's for
- Whether it's required or optional
- A sensible default if one exists

## Output

Save the SRS to `docs/srs.md`. After writing, update `CLAUDE.md` if a new doc entry is needed.