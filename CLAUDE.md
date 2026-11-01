# NoteMind

Chat app over the user's own notes and documents (RAG). Answers stream in with citations pointing to the source chunk. A side panel shows suggested questions that reorder based on user behavior (clicks, dismissals, opened citations).

The user's main goal is to brush up on React 19 and modern JS. The frontend is the focus; the backend is a thin supporting layer.

## Working preferences

- The user is an experienced engineer (MERN, React Native, Electron, databases). Don't explain basics.
- Use AI to increase productivity, not to replace the user's thinking. If the user says "I'll figure that out" or "no code now", give structure and reasoning, not code.
- Before changing existing code, analyze impact and state what depends on the thing being changed.
- Be concise and decision-oriented: a recommendation with trade-offs, not a menu.
- Ask before generating large amounts of code. Small, targeted snippets are fine when asked for.

## Git

- Make atomic commits, one logical change each.
- **Do NOT add a `Co-Authored-By` tag (or any attribution line) to commit messages or PR descriptions.**
- Only commit when asked.
- Main branch is `main`.

## Stack

- pnpm workspace monorepo (`apps/*`, `packages/*`)
- Web: Vite, React 19, TypeScript, TanStack Router + Query, Zustand, Tailwind v4, shadcn/ui
- API: Express modular monolith now; NestJS later as a new `api-nest` app implementing the same contract
- `packages/contract`: Zod schemas and inferred types, shipped as TS source with no build step. `main`, `types` and `exports` all point at `./src/index.ts`. Only re-export files that have content.
- LLM: Anthropic API behind a single `providers/llm.ts`
- Data: in-memory/JSON in Phase 1; Postgres + pgvector from Phase 2
- Deploy: Vercel (web), Railway/Render (API), Neon/Supabase (DB). Verify free-tier terms on each provider's page.

## Layout

```
apps/web/src: app/ routes/ features/<name>/ components/{ui,layout}/ lib/ mocks/ test/
apps/api-express/src: index.ts config/ common/ providers/llm.ts modules/<name>/ (db/, workers/ from Phase 2)
packages/contract/src: common, chat, threads, documents (P2), signals (P4), index
docs/: decisions.md (decision log), architecture.md
```

Do not create empty placeholder folders for later phases.

## Architecture rules

State ownership:
- Server data lives in TanStack Query. Never copy it into Zustand.
- Client-only shared state (draft, model setting, signal buffer) lives in Zustand.
- URL state lives in TanStack Router.
- Ephemeral UI lives in `useState` / `useOptimistic`.

Dependencies:
- `routes/` imports features, components, lib. Not `app/` internals.
- `features/*` imports other features only via their `index.ts`, plus components, lib, contract. Never `routes/`, `app/`, or another feature's internals.
- `components/` imports lib and contract. Never `features/`.
- `lib/` imports contract only.
- Backend: routes call their own service. Services call repos, providers, and other modules' services. Modules never reach into another module's tables, repos, or routes.
- All network calls go through `lib/api-client.ts`; responses are parsed with `contract` schemas.
- Query keys are defined once per feature (`*.queries.ts`).
- Enforce boundaries with ESLint from the start.

Naming: lowercase-hyphen folders, PascalCase component files, camelCase or dotted names elsewhere (`chat.service.ts`). Role suffixes: `.routes`, `.service`, `.repo`, `.store`, `.api`, `.queries`. Tests sit next to the file they test.

Security: the Anthropic key stays server-side. Every row carries `user_id`, and every vector query filters by it.

## Phases

0. Foundation: monorepo, contract package, bare web + API, walking skeleton (styled page + `/api/health` validated by contract), deployed early
1. Chat: streaming, optimistic messages, threads, Zustand draft/settings, `POST /api/chat`, threads endpoints, `llm.ts`; build against MSW mocks first
2. Library and RAG: upload, ingest status, pgvector, documents module
3. Citations: chips, source drawer, `GET /api/chunks/:id`
4. Personalization: signal buffer, suggestions panel, signals + suggestions modules
5. Hardening: error/loading states, tests, real auth, rate limiting, cost logging
6. `api-nest` implementing the same contract, with no frontend changes

## Current progress

Done: root workspace, `.gitignore`, `packages/contract` scaffold (empty domain files, zod installed), placeholder README and docs, first commits on `main`.

Next (Phase 0):
1. Scaffold `apps/web` with the Vite React TS template (package name `web`).
2. Add Tailwind v4 (`tailwindcss`, `@tailwindcss/vite`, `@types/node`), the `@` alias in `vite.config.ts`, `tsconfig.json` and `tsconfig.app.json`.
3. shadcn init and the button component (use `tw-animate-css` if an animate import errors).
4. Init `apps/api-express` (Express, tsx, tsup, types) with a minimal `/api/health`.
5. Link `@notemind/contract` into both apps with `workspace:*`; verify with `pnpm ls -r --depth -1`.
6. Add a `HealthResponse` schema to contract, use it in both apps, deploy the skeleton.

Phase 0 is done when `pnpm dev` runs both apps, the page shows a styled button and the API health status, and both are deployed and working together.

## Deployment notes

- Vercel root directory is `apps/web`; set `VITE_API_URL` in production.
- API host builds from the repo root and bundles `@notemind/contract` (e.g. tsup `--noExternal @notemind/contract`).
- Allow the Vercel domain in the API's CORS settings.
- Cost guardrails: Anthropic spend limit, host usage alert, per-user rate limits, caps on upload size and chunk count.

Keep `docs/decisions.md` updated with each significant decision and the reason.
