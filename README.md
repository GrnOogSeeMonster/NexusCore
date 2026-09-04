# NexusCore

An internal developer portal built around an MCP server: a service catalog, workflow
engine, knowledge base and analytics layer that an AI coding agent can query directly
over the Model Context Protocol, with a Next.js portal on top.

The interesting part is the MCP server, not the web UI. Most developer portals expose
their catalog to humans through a dashboard; this one also exposes it to an agent as
eleven typed tools, so "which services depend on the payments API and what's their
error rate" is answerable from inside an editor.

---

## Status

**Prototype — the MCP server is the built part; the portal is a shell.** Last worked
on 10 September 2025. Not currently running anywhere.

| | |
|---|---|
| Size | ~4,800 lines TypeScript, plus a 620-line Prisma schema |
| Built and working | MCP server (11 tools across 4 modules), database schema, auth + setup flow |
| Scaffolded only | Web portal has one page (`/`); no catalog, workflow or analytics screens |
| Not started | The `ai-service` and `realtime` services have Dockerfiles but no source |
| Tests | None |
| Verified | Never run end to end. `pnpm install` and `docker compose up` are untested from a clean checkout |

### What actually exists

**`apps/mcp-server`** — ~2,600 lines. A stdio MCP server exposing eleven tools:

| Module | Tools |
|---|---|
| `knowledge/knowledge-base.ts` | `search_knowledge`, `add_knowledge` |
| `services/service-catalog.ts` | `get_service_info`, `list_services`, `get_service_dependencies` |
| `workflows/workflow-engine.ts` | `execute_workflow`, `get_workflow_status` |
| `analytics/analytics.ts` | `get_service_metrics`, `get_incidents`, `analyze_platform_health`, `natural_language_query` |

**`packages/database`** — a 620-line Prisma schema covering 16 models: `User`, `Team`,
`TeamMember`, `Service`, `Dependency`, `Workflow`, `WorkflowExecution`, `Action`,
`Scorecard`, `ServiceScorecard`, `Metric`, `Deployment`, `Knowledge`, `KnowledgeRelation`,
`Incident`, `Integration`. This is the most thought-through artifact in the repo — the
data model for a full portal is there even where the screens are not.

**`apps/web`** — ~2,200 lines. Next.js 14 App Router. One page (`/`) composed of a hero,
a stats overview, a quick-actions panel, a Three.js "service galaxy" and a recent-activity
feed. Four API routes: NextAuth, health, and a two-step setup flow. NextAuth with a
credentials provider, and middleware gating routes behind a session.

### What does not exist

The `/services`, `/workflows`, `/analytics` and `/knowledge` screens implied by the
data model. Deployment integrations. Any test.

---

## Stack

Next.js 14 · React 18 · TypeScript · Tailwind · Three.js / React Three Fiber ·
NextAuth · Prisma · PostgreSQL · Redis · Model Context Protocol SDK ·
Turborepo + pnpm workspaces · Docker Compose · Caddy

---

## Running it

```bash
pnpm install
cp .env.example .env          # set DATABASE_URL and NEXTAUTH_SECRET
docker compose -f docker-compose.dev.yml up -d postgres redis
pnpm run db:generate && pnpm run db:push
pnpm run dev
```

Helper scripts wrap the same steps: `./scripts/dev-start.sh`, `dev-stop.sh`,
`dev-restart.sh`.

The MCP server runs over stdio and is registered with an MCP client rather than
started directly:

```bash
pnpm --filter mcp-server build
node apps/mcp-server/dist/index.js
```

---

## Layout

```
apps/
  web/           Next.js portal — 1 page, 4 API routes, 8 components
  mcp-server/    MCP stdio server — 11 tools over 4 domain modules
packages/
  database/      Prisma schema (16 models) + client export
docker/          Dockerfiles per service (ai-service and realtime are empty shells)
scripts/         dev-start / dev-stop / dev-restart / setup
```

---

## Notes

Some strings inside the repo still say "DevForge" — an earlier working name, and a
different project from the `devforge` repo alongside this one.

MIT licensed.
