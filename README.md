# StyleMate AI

*Your Wardrobe. Your Style. Your Everyday Confidence.*

An AI-assisted personal wardrobe and daily outfit recommendation platform: ASP.NET Core (modular monolith)
+ PostgreSQL + Next.js.

**Status:** MVP (phases 1–5) implemented: API + web app. See [`docs/development-roadmap.md`](docs/development-roadmap.md).

## Documentation
- [Product requirements & MVP scope](docs/product-requirements.md)
- [Architecture](docs/architecture.md)
- [Database design](docs/database-design.md)
- [API design](docs/api-design.md)
- [Recommendation engine](docs/recommendation-engine.md)
- [Security model](docs/security-model.md) · [Threat model](docs/threat-model.md)
- [Testing strategy](docs/testing-strategy.md)
- [Deployment strategy](docs/deployment-guide.md)
- [Decision log](docs/decisions.md)

## Local setup

Prerequisites: .NET 10 SDK, Node.js 22 LTS, PostgreSQL 16+ (local install or container).

1. **Database connection** (kept out of the repo, in user-secrets):
   ```
   dotnet user-secrets set "Database:ConnectionString" "Host=localhost;Port=5432;Database=stylemate_dev;Username=…;Password=…" --project src/StyleMate.Api
   ```
2. **API** — applies migrations on start in Development and listens on http://localhost:5080:
   ```
   dotnet run --project src/StyleMate.Api --launch-profile http
   ```
   Health: `GET /health/ready`. Emails are captured by the dev mailbox (`/dev/mailbox?to=you@example.com` in the web app).
3. **Web app** — proxies `/api/*` to the API (same origin, decision D4) and serves http://localhost:3000:
   ```
   cd frontend/stylemate-web
   npm install
   npm run dev
   ```
   Set `API_ORIGIN` if the API is not on `http://localhost:5080`.

### API contract → frontend client
The typed client is generated from the API's OpenAPI document (exposed in Development only). After changing an
endpoint or DTO, with the API running: `cd frontend/stylemate-web && npm run api:sync`, then commit
`openapi/v1.json` and `src/lib/api/schema.d.ts` together with the change.

### Tests
```
dotnet test StyleMate.slnx                 # needs STYLEMATE_TEST_POSTGRES (see docs/testing-strategy.md)
# .NET 10 RC SDKs: run each test project instead, e.g. dotnet run --project tests/StyleMate.Api.Tests
# (stop a running API first; it locks the DLLs the test build copies)
cd frontend/stylemate-web
npm run lint && npm run typecheck && npm test
npx playwright install chromium && npm run test:e2e   # smoke tests; builds and starts the app
```
