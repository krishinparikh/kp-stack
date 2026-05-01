# kp-stack
An opinionated boilerplate tech stack, including Claude Code infrastructure, for building AI-native full-stack web applications.

## Recommended Usage
There is no right or wrong way to use this, but here are some guidelines I'd recommend:

- Write a PRD with `/prd-creator` and an SRS with `/srs-creator`
- Iterate on these until satisfied, and only then start implementing code
- Add/update/remove skills/MCPs as fit for your project

## Tech Stack

| Technology | Description | Rationale |
|------------|-------------|-----------|
| Next.js | Full-stack React framework | Handles UI, routing, and API in one framework |
| TypeScript | Typed JavaScript | Compile-time safety and better editor tooling |
| Drizzle ORM | Type-safe SQL ORM | Lightweight with full SQL control and typed schemas |
| Neon | Serverless Postgres | Scales to zero and fits serverless deployments |
| LangChain | LLM application framework | Composable chains, agents, and tool use |
| TanStack Query | Async state management for React | Declarative data fetching with caching and sync |
| shadcn/ui | Pre-built Radix + Tailwind components | Copy-paste components with full customization |

## Claude Skills

| Skill | Description |
|-------|-------------|
| `prd-creator` | Guides structured product thinking from problem through solution to produce a PRD |
| `srs-creator` | Translates a PRD into a complete technical blueprint (SRS) |
| `frontend-eng` | Enforces frontend conventions — RSC by default, client component boundaries, shadcn/ui |
| `backend-eng` | Enforces backend conventions — queries, actions, route handlers, Drizzle, type safety |
| `neon-drizzle` | Provisions a Neon database and scaffolds a full Drizzle ORM setup |
| `mermaid-diagrams` | Creates software diagrams (sequence, ER, flowchart, C4, etc.) using Mermaid syntax |
| `claudemd-creator` | Generates or updates the project's CLAUDE.md file |
| `find-skills` | Discovers and installs new agent skills |

## MCP Servers

| MCP | Description | Scope |
|-----|-------------|-------|
| `shadcn` | Installs and configures shadcn/ui components directly from the registry | Project |
| `playwright` | Automates browser interactions for testing, screenshots, and UI debugging | Global |
| `figma` | Connects to Figma for design-to-code workflows | Global |
| `context7` | Fetches up-to-date library and framework documentation | Global |
| `Neon` | Manages Neon Postgres projects, branches, schemas, and queries | Global |

## Helpful Resources

| Resource | Description |
|----------|-------------|
| [skills.sh](https://skills.sh) | Directory of community-built Claude Code skills |
| [mcpmarket.com](https://mcpmarket.com) | Marketplace for discovering and installing MCP servers |
