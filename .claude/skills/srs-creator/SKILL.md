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
Describe the system components and how they connect. The architecture MUST follow Next.js App Router conventions:
- **Frontend**: Next.js App Router with React Server Components by default. Client components (`"use client"`) only when interactivity is needed. TanStack Query for client-side server state (caching, mutations, optimistic updates). shadcn/ui for all UI primitives.
- **Backend services**: Default to **queries** and **actions** over API Route Handlers. Only create a Route Handler (`app/api/`) when you need an externally callable endpoint (webhooks, third-party integrations, streaming SSE).
  - **Queries** (`src/queries/`) — server-side read functions. Each file exports async functions that use Drizzle to fetch data. Called directly from Server Components or via TanStack Query.
  - **Actions** (`src/actions/`) — Server Actions for mutations. Each file uses `"use server"` and exports async functions that write/update/delete via Drizzle. Called from client components via form actions or `useMutation`.
  - Drizzle ORM for all database access — no raw SQL unless Drizzle can't express the query.
- **Database**: Neon serverless Postgres via Drizzle. Schema defined in `src/db/schema.ts`. Migrations via `drizzle-kit`.
- **AI layer**: LangChain (TypeScript) for any LLM orchestration, chains, agents, or tool use. Prefer LangChain abstractions over raw API calls.
- Communication: Server Actions for mutations, direct query calls for reads, Route Handlers only when an HTTP endpoint is required. Server-Sent Events via Route Handlers for streaming LLM responses.
- Data flow: trace a typical request end-to-end from user action → React component → query/action (or TanStack Query → action) → Drizzle → Neon → response.
- **File structure**: Follow the canonical layout in `file-architecture.md` (in this skill's directory). Show the full project tree in the SRS, expanding with project-specific files as needed.

### 3. Database Schema
Schema is defined using **Drizzle ORM** table declarations (not raw SQL). For each table:
- Table name as a sub-heading
- One-sentence description of what it stores
- A table with columns: **Column**, **Type**, **Notes**
- Mark PKs and FKs explicitly
- Include status enums inline (e.g., `parsing` → `specialists` → `complete`)
- Use Postgres-native types via Drizzle helpers: `uuid`, `text`, `jsonb`, `timestamp`, `pgEnum`, etc.
- Show a Drizzle schema snippet for each table so the engineer can copy it directly into `src/db/schema.ts`

Derive tables directly from the PRD's entities. Every noun that gets created, stored, or referenced should map to a table or a column.

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