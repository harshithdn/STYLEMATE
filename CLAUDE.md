# CLAUDE.md — StyleMate AI

## Current phase
**MVP phases 1–5 implemented (API + Next.js web app, 2026-10-24).** Next: verify MVP acceptance criteria end to end
(Playwright journeys against a running stack), then Phase 6 (AI cataloging).

## Read first
- `docs/development-roadmap.md` — phases & exit criteria
- `docs/architecture.md` — layers, dependency rules, auth/storage/jobs design
- `docs/decisions.md` — accepted/proposed decisions (don't re-decide accepted ones)
- `docs/security-model.md`, `docs/threat-model.md` — controls every feature must respect

## Non-negotiables (short form)
- Modular monolith: Domain ← Application ← Infrastructure ← Api. Domain has no dependencies.
- Thin controllers; business logic in Application handlers / Domain rules. No MediatR, no AutoMapper,
  no generic repository.
- User ID only from the authenticated principal. Every user-scoped query filters by `user_id`;
  foreign resources → 404. Every user-scoped endpoint gets a cross-user isolation test.
- Private images only; signed URLs never logged; images re-encoded on upload.
- Mock providers are labelled (`IsMock`) and never used in Production.
- Migrations for every schema change; never `EnsureCreated` in app code.
- UTC `timestamptz` for instants; `date` + IANA time zone for planner days. Use `TimeProvider`.
- Never log passwords, tokens, cookies, signed URLs, or images.
- Never claim tests pass without running them. Never disable tests to go green.

## Commands
- API: `dotnet run --project src/StyleMate.Api --launch-profile http` (http://localhost:5080; DB connection in user-secrets)
- Backend tests: `dotnet test StyleMate.slnx` (needs `STYLEMATE_TEST_POSTGRES`). On a .NET 10 RC SDK, `dotnet test`
  rejects the MTP opt-in; run each project instead: `dotnet run --project tests/StyleMate.<X>.Tests`. Stop a running
  API first — it locks the DLLs the Api/Architecture test builds copy.
- Web: `cd frontend/stylemate-web && npm run dev` (http://localhost:3000, proxies `/api/*` to the API)
- Web checks: `npm run lint && npm run typecheck && npm test`; smoke E2E: `npm run test:e2e`
- Contract changed? With the API running: `npm run api:sync` and commit `openapi/v1.json` + `src/lib/api/schema.d.ts`.
- Frontend uses Next.js 16 (Cache Components on): read `node_modules/next/dist/docs/` before using Next APIs;
  client hooks reading the URL (`usePathname`, `useSearchParams`, `useParams`) need a `<Suspense>` boundary.

## Per-feature workflow
Objective → files → design/security notes → DB changes → tests → implement → build + test → docs → summary
of done/not done. Small, focused commits.
