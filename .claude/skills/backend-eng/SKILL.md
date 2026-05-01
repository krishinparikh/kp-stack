---
name: backend-eng
description: "INVOKE THIS SKILL when writing ANY backend code — queries, actions, route handlers, database schema, or AI/LLM integrations."
---

For deeper guidance, reference the `next-best-practices` skill as needed.

## Queries and Actions over Route Handlers

Default to **queries** and **actions**. Only create a Route Handler (`app/api/`) when you need an externally callable HTTP endpoint (webhooks, third-party integrations, streaming SSE).

### Queries (`src/queries/`)
- Server-side **read** functions. Each file exports async functions that use Drizzle to fetch data.
- Called directly from Server Components or via TanStack Query in client components.
- One file per domain entity (e.g., `src/queries/users.ts`, `src/queries/projects.ts`).
- Never put read logic inside Route Handlers or Server Actions — keep reads in queries.

### Actions (`src/actions/`)
- Server Actions for **mutations** (create, update, delete).
- Each file uses `"use server"` at the top and exports async functions that write via Drizzle.
- Called from client components via form actions or TanStack Query's `useMutation`.
- One file per domain entity (e.g., `src/actions/users.ts`, `src/actions/projects.ts`).

### Route Handlers (`src/app/api/`)
Only use when you genuinely need an HTTP endpoint:
- Webhooks from external services
- SSE streams (e.g., streaming LLM responses via `ReadableStream`)
- Third-party API callbacks
- Endpoints consumed by non-browser clients

## Database Access

- **Drizzle ORM for everything.** No raw SQL unless Drizzle literally cannot express the query.
- Schema lives in `src/db/schema.ts`. Drizzle client and Neon connection pool in `src/db/index.ts`.
- Migrations via `drizzle-kit` — never write migration SQL by hand.
- Use Postgres-native types via Drizzle helpers: `uuid`, `text`, `jsonb`, `timestamp`, `pgEnum`, etc.

## Type Safety

The Drizzle schema is the **single source of truth** for all types. Never manually redefine types that Drizzle already provides.

- Use `typeof table.$inferSelect` and `typeof table.$inferInsert` to derive row types. Export these from `src/db/schema.ts`.
- Extend inferred types with `&` or `Omit`/`Pick` — don't duplicate fields into a hand-written type.
- Use `type`, not `interface`.
- API request/response shapes should be derived from or composed of Drizzle-inferred types so the contract between client and server stays in sync with the schema automatically.

## AI Layer

- **LangChain (TypeScript)** for all LLM orchestration, chains, agents, and tool use.
- Prefer LangChain abstractions over raw API calls to LLM providers.
- AI code lives in `src/lib/ai/` — one file per chain, agent, or tool.
- Stream LLM responses to the client via SSE in a Route Handler using `ReadableStream`.

## Data Flow

The canonical request path:

1. **Read**: React Server Component → query function (`src/queries/`) → Drizzle → Neon → rendered HTML
2. **Mutation**: Client component → Server Action (`src/actions/`) → Drizzle → Neon → revalidate
3. **Streaming AI**: Client component → Route Handler (`app/api/`) → LangChain → SSE stream → client
