# CareStream — Project Todo List

Implementation checklist for the CareStream capstone. See [carestream-spec.md](carestream-spec.md) for architecture and API details.

---

## 1. Architecture & Monorepo Setup

- [ ] Initialize monorepo with pnpm workspaces (`pnpm-workspace.yaml`)
- [ ] Create package structure: `apps/web`, `services/catalog-api`, `services/analytics-api`, `packages/shared-types`
- [ ] Configure root ESLint, Prettier, TypeScript project references
- [ ] Write ADR-001: Why Shaka Player over Video.js
- [ ] Write ADR-002: Microservices split (catalog-api vs analytics-api)
- [ ] Create MSSQL migration scripts: `Content`, `WatchProgress`, `Session`, `QoSEvent`
- [ ] Create shared TypeScript types package for API contracts
- [ ] Set up environment config per service (`.env.example` files)
- [ ] Document Azure Key Vault integration notes for production secrets

---

## 2. Backend — Catalog API

- [ ] Scaffold Node.js service (Express or Fastify) on port 3001
- [ ] Implement repository pattern with MSSQL driver (mssql or Prisma)
- [ ] `GET /api/v1/content` — list all content with optional progress
- [ ] `GET /api/v1/content/:id` — detail with HLS/DASH stream URLs
- [ ] `GET /api/v1/progress/:userId` — continue-watching list
- [ ] `PUT /api/v1/progress/:userId/:contentId` — upsert watch position
- [ ] Seed 10+ healthcare CME titles with public test stream URLs
- [ ] OpenAPI/Swagger spec at `/api/docs`
- [ ] CORS config allowing Next.js origin only
- [ ] Unit tests for repository layer (Jest)
- [ ] Integration test against local SQL Server instance

---

## 3. Backend — Analytics API

- [ ] Scaffold Node.js service on port 3002
- [ ] `POST /api/v1/events/batch` — ingest QoS events (max 50 per batch)
- [ ] Request validation: schema check all event types and payloads
- [ ] Rate limiting middleware (e.g. 100 req/min per IP)
- [ ] `GET /api/v1/metrics/summary` — avg startup time, rebuffer ratio, error rate
- [ ] `GET /api/v1/metrics/by-content/:contentId` — per-title aggregates
- [ ] MSSQL stored procedure or view for dashboard aggregates
- [ ] Index optimization on `QoSEvent` table
- [ ] Unit tests for event validation and aggregation logic
- [ ] Integration test: ingest events → query summary returns expected values

---

## 4. Front-end — Catalog & App Shell

- [ ] Initialize Next.js 14+ App Router with TypeScript strict mode
- [ ] Configure Tailwind CSS
- [ ] SSR home page (`app/page.tsx`) fetching from catalog-api
- [ ] `ContentGrid` and `ContentCard` components with poster images
- [ ] Content detail page (`app/content/[id]/page.tsx`) with CME metadata
- [ ] Continue-watching row from progress API
- [ ] Responsive layout: mobile, tablet, desktop breakpoints
- [ ] `next/image` for poster optimization
- [ ] Lazy load below-fold content cards
- [ ] Baseline Lighthouse audit on catalog page (record scores)

---

## 5. Front-end — Player Core

- [ ] Install and configure Shaka Player
- [ ] Create `useShakaPlayer` hook with typed event model (discriminated unions)
- [ ] `VideoPlayer` client component with `ssr: false` dynamic import
- [ ] Attach Shaka to `<video ref>` element
- [ ] Custom `PlayerControls`: play/pause, seek, volume, fullscreen
- [ ] `ProgressBar` with buffered ranges visualization
- [ ] Buffering spinner overlay on Shaka `buffering` event
- [ ] `ErrorOverlay` with retry button and Shaka error code mapping
- [ ] `QualityMenu` — list variant tracks, manual quality override
- [ ] Subtitle/caption track selector (if available on stream)
- [ ] Resume playback from watch-progress API on load
- [ ] Save progress periodically (every 30s) and on pause/unmount
- [ ] HLS.js or native HLS fallback for Safari if needed

---

## 6. Streaming Protocols & DRM

- [ ] Auto-select DASH (Shaka) on Chrome/Firefox/Edge
- [ ] Auto-select HLS (native or HLS.js) on Safari
- [ ] Display current quality: resolution + bitrate indicator
- [ ] Manual quality override with ABR disable/enable toggle
- [ ] Integrate ClearKey DRM demo stream (Shaka sintel asset)
- [ ] Write ADR-003: Manifest structure and CDN strategy
- [ ] Document each seed asset's manifest URL and format in README

---

## 7. Performance Engineering

- [ ] Dynamic import Shaka Player bundle (`next/dynamic`, `ssr: false`)
- [ ] `@next/bundle-analyzer` report — document player chunk size
- [ ] `preconnect` to CDN domains in `layout.tsx`
- [ ] Preload poster images only (not video segments)
- [ ] Instrument time-to-first-frame (TTFF) with `performance.now()`
- [ ] Document TTFF breakdown: manifest / first segment / decode
- [ ] Lighthouse audit: target LCP < 2.5s, INP < 200ms on catalog
- [ ] 45-minute playback memory profiling notes (heap snapshot comparison)
- [ ] Performance report markdown in `docs/performance-report.md`

---

## 8. Analytics & QoS Dashboard

- [ ] Implement `AnalyticsClient` class in front-end
- [ ] Generate sessionId on watch page mount
- [ ] Emit events: `playback_start`, `rebuffer_start`, `rebuffer_end`, `quality_change`, `error`, `heartbeat`
- [ ] Batch events every 10s or 20 events; send via `navigator.sendBeacon` or fetch
- [ ] Admin dashboard page (`app/admin/page.tsx`) — protected route (basic auth demo)
- [ ] Fetch `/metrics/summary` and display KPI cards
- [ ] Chart: startup time trend (7-day line chart)
- [ ] Chart: rebuffer ratio by content (bar chart)
- [ ] Chart: error rate heatmap or table by content
- [ ] Verify SQL aggregates match client-side event counts (accuracy check)

---

## 9. Cross-Platform & Smart TV

- [ ] Implement spatial focus navigation on catalog grid (arrow keys)
- [ ] Visible `:focus-visible` ring on all interactive elements
- [ ] Player keyboard shortcuts: Space, arrows, M, F
- [ ] TV-safe layout: 5% overscan margin on player controls
- [ ] Minimum 44px touch/focus targets on all buttons
- [ ] `playsInline` attribute on video for iOS
- [ ] Test matrix document: Chrome, Firefox, Safari, mobile Safari
- [ ] Tizen Web Simulator test notes (if simulator available)
- [ ] webOS / Tizen hosted web app packaging notes in README

---

## 10. Security

- [ ] CSP headers in Next.js `middleware.ts`
- [ ] Allow `media-src` for CDN domains and `blob:`
- [ ] Allow `connect-src` for catalog-api and analytics-api
- [ ] Input validation on analytics-api (reject malformed events)
- [ ] CORS locked to known origins (no wildcard in production)
- [ ] Audit codebase for `dangerouslySetInnerHTML` and XSS vectors
- [ ] Write security ADR: XSS, CSP, CORS, DRM threat model
- [ ] Ensure no license keys or secrets in client bundle

---

## 11. Testing & Quality

- [ ] Jest unit tests: catalog-api repository functions
- [ ] Jest unit tests: analytics-api event validation and aggregation
- [ ] Jest unit tests: player state reducer / error mapping helpers
- [ ] RTL tests: `PlayerControls` renders and responds to clicks
- [ ] RTL tests: keyboard shortcut handlers
- [ ] Mock Shaka Player in front-end unit tests
- [ ] Playwright E2E: home → content detail → watch → video playing
- [ ] Playwright E2E: error state and retry flow
- [ ] Configure Playwright to record video on failure
- [ ] Target 80%+ coverage on API services; document front-end coverage

---

## 12. CI/CD, Documentation & Portfolio

- [ ] Create `azure-pipelines.yml`: lint → test → build → E2E → deploy
- [ ] Separate pipeline jobs for web, catalog-api, analytics-api
- [ ] Lighthouse CI gate on front-end PRs (performance budget)
- [ ] Deploy web app to Vercel or Azure Static Web Apps (demo)
- [ ] Deploy APIs to Azure App Service or OpenShift (demo)
- [ ] Write GitHub Actions equivalent YAML in `docs/github-actions-equivalent.yml`
- [ ] README: architecture diagram, setup instructions, demo URL
- [ ] README: "Angular → React learning narrative" section
- [ ] Record demo GIF or short video of catalog + playback + admin dashboard
- [ ] Write `docs/interview-prep.md`: map .NET/Angular/healthcare experience to Player team
- [ ] Write "What I'd do at scale" section: multi-CDN, SSAI, client-side beacon at millions QPS
- [ ] CHANGELOG with semantic versioning

---

## Milestone tracker

| Milestone | Target week | Done |
|-----------|-------------|------|
| Monorepo + catalog-api returning seed data | Week 2 | [ ] |
| Next.js catalog page live | Week 2 | [ ] |
| Raw MSE / manifest parsing exercises | Week 4 | [ ] |
| Shaka player with custom controls | Week 6 | [ ] |
| QoS events in SQL + analytics-api | Week 8 | [ ] |
| Admin dashboard with charts | Week 9 | [ ] |
| Playwright E2E passing | Week 10 | [ ] |
| Azure DevOps pipeline green | Week 11 | [ ] |
| Portfolio README complete | Week 12 | [ ] |
