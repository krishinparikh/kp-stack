# kp-stack File Architecture Reference

The canonical file structure for all projects built on kp-stack. The SRS Application Structure section MUST follow this layout.

## Base Structure

```
project-name/
├── drizzle.config.ts
├── next.config.ts
├── tailwind.config.ts
├── src/
│   ├── app/
│   │   ├── layout.tsx                 # Root layout (providers, fonts, metadata)
│   │   ├── page.tsx                   # Landing / home page
│   │   ├── (feature-name)/
│   │   │   ├── page.tsx               # Feature page (maps 1:1 to PRD feature)
│   │   │   └── components/            # Page-specific components
│   │   └── api/
│   │       └── feature/
│   │           └── route.ts           # Next.js Route Handler
│   ├── components/
│   │   ├── ui/                        # shadcn/ui primitives (Button, Dialog, etc.)
│   │   └── shared/                    # App-wide shared components
│   ├── db/
│   │   ├── schema.ts                  # All Drizzle table definitions
│   │   ├── index.ts                   # Drizzle client + Neon connection pool
│   │   └── migrations/                # drizzle-kit generated migrations
│   ├── lib/
│   │   ├── ai/                        # LangChain chains, agents, tools
│   │   │   └── [chain-or-agent].ts    # One file per chain/agent/tool
│   │   ├── queries/                   # Drizzle query helpers (reusable DB operations)
│   │   └── utils.ts                   # General utility functions
│   ├── hooks/                         # Custom React hooks
│   └── types/                         # TypeScript type definitions
│       └── [domain-entity].ts         # One file per domain entity
└── ...
```

## Rules

- **`src/` directory** is required — all source code lives under `src/`.
- **App Router only** — use `app/` directory routing, never `pages/`.
- **Route groups** — use `(groupName)` for logical page grouping without affecting URLs.
- **Colocated page components** — page-specific components live in `components/` subdirectories next to their `page.tsx`.
- **Shared UI** — shadcn/ui components in `src/components/ui/`; app-wide shared components in `src/components/shared/`.
- **Database** — all Drizzle schema in `src/db/schema.ts`; DB client/connection in `src/db/index.ts`.
- **AI layer** — all LangChain code in `src/lib/ai/` with one file per chain, agent, or tool.
- **Query helpers** — reusable Drizzle query functions in `src/lib/queries/`, not inlined in route handlers.
- **Types** — one file per domain entity in `src/types/`.
- **Small files** — prefer many small, focused files over few large ones. Constants, sub-components, utility modules, config, and types should each live in their own files.
- **Feature pages map 1:1** to the PRD's core features.
