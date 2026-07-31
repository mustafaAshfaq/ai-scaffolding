# CareStream — Capstone Project Specification

Healthcare CME (Continuing Medical Education) streaming platform. Demonstrates React/Next.js player skills while leveraging Node.js, MSSQL, microservices, and Azure DevOps experience.

## Overview

| Attribute | Value |
|-----------|-------|
| Domain | Healthcare continuing education video streaming |
| Users | Clinicians (watch), Admins (QoS monitoring) |
| Front-end | Next.js 14+ App Router, TypeScript strict, Tailwind, Shaka Player |
| Back-end | Node.js microservices (catalog-api, analytics-api) |
| Database | SQL Server |
| CI/CD | Azure DevOps (primary); GitHub Actions documented for interview |
| Monorepo | pnpm workspaces |

## Architecture

```mermaid
flowchart TB
  subgraph client [apps/web — Next.js]
    Catalog[Catalog SSR]
    WatchPage[Watch Client Page]
    AdminDash[QoS Dashboard]
    PlayerUI[Custom Player UI]
    AnalyticsSDK[Analytics Client]
  end
  subgraph media [Browser Media]
    Shaka[Shaka Player]
    HLSjs[HLS.js Fallback]
  end
  subgraph services [Node.js Services]
    CatalogAPI[catalog-api :3001]
    AnalyticsAPI[analytics-api :3002]
  end
  subgraph data [SQL Server]
    DB[(carestream DB)]
  end
  Catalog --> CatalogAPI
  WatchPage --> PlayerUI
  PlayerUI --> Shaka
  PlayerUI --> HLSjs
  AnalyticsSDK --> AnalyticsAPI
  AdminDash --> AnalyticsAPI
  CatalogAPI --> DB
  AnalyticsAPI --> DB
```

## Monorepo structure

```
carestream/
├── apps/
│   └── web/                         # Next.js front-end
│       ├── app/
│       │   ├── layout.tsx
│       │   ├── page.tsx             # Catalog home (SSR)
│       │   ├── content/[id]/page.tsx
│       │   ├── watch/[id]/page.tsx  # Client player page
│       │   └── admin/page.tsx       # QoS dashboard
│       ├── components/
│       │   └── player/
│       ├── hooks/
│       │   └── useShakaPlayer.ts
│       ├── lib/
│       │   └── analytics-client.ts
│       └── middleware.ts            # CSP headers
├── services/
│   ├── catalog-api/                 # Express/Fastify REST API
│   └── analytics-api/               # QoS ingest + aggregates
├── packages/
│   └── shared-types/                # Shared TS interfaces
├── db/
│   ├── migrations/
│   │   ├── 001_create_content.sql
│   │   ├── 002_create_watch_progress.sql
│   │   ├── 003_create_qos_events.sql
│   │   └── 004_create_sessions.sql
│   └── seed/
│       └── seed_content.sql
├── docs/
│   └── adr/
│       ├── ADR-001-shaka-over-videojs.md
│       ├── ADR-002-microservices-split.md
│       └── ADR-003-manifest-cdn-strategy.md
├── azure-pipelines.yml
├── pnpm-workspace.yaml
└── README.md
```

## Database schema

### Content

```sql
CREATE TABLE Content (
  Id            UNIQUEIDENTIFIER PRIMARY KEY DEFAULT NEWID(),
  Title         NVARCHAR(200) NOT NULL,
  Description   NVARCHAR(MAX),
  Specialty     NVARCHAR(100),          -- e.g. Cardiology, Oncology
  CmeCredits    DECIMAL(4,2),
  DurationSec   INT NOT NULL,
  PosterUrl     NVARCHAR(500),
  HlsUrl        NVARCHAR(500) NOT NULL,
  DashUrl       NVARCHAR(500) NOT NULL,
  DrmScheme     NVARCHAR(50) NULL,    -- NULL | ClearKey | Widevine
  CreatedAt     DATETIME2 DEFAULT GETUTCDATE()
);
```

### WatchProgress

```sql
CREATE TABLE WatchProgress (
  Id            UNIQUEIDENTIFIER PRIMARY KEY DEFAULT NEWID(),
  UserId        NVARCHAR(100) NOT NULL,  -- demo: localStorage UUID
  ContentId     UNIQUEIDENTIFIER NOT NULL REFERENCES Content(Id),
  PositionSec   INT NOT NULL DEFAULT 0,
  UpdatedAt     DATETIME2 DEFAULT GETUTCDATE(),
  UNIQUE (UserId, ContentId)
);
```

### Session

```sql
CREATE TABLE Session (
  Id            UNIQUEIDENTIFIER PRIMARY KEY DEFAULT NEWID(),
  UserId        NVARCHAR(100),
  ContentId     UNIQUEIDENTIFIER NOT NULL REFERENCES Content(Id),
  StartedAt     DATETIME2 DEFAULT GETUTCDATE(),
  EndedAt       DATETIME2 NULL,
  DeviceType    NVARCHAR(50)           -- desktop | mobile | tv
);
```

### QoSEvent

```sql
CREATE TABLE QoSEvent (
  Id            BIGINT IDENTITY PRIMARY KEY,
  SessionId     UNIQUEIDENTIFIER NOT NULL REFERENCES Session(Id),
  ContentId     UNIQUEIDENTIFIER NOT NULL,
  EventType     NVARCHAR(50) NOT NULL,  -- playback_start | rebuffer_start | rebuffer_end | quality_change | error | heartbeat
  Payload       NVARCHAR(MAX),          -- JSON
  ClientTs      DATETIME2 NOT NULL,
  ReceivedAt    DATETIME2 DEFAULT GETUTCDATE()
);

CREATE INDEX IX_QoSEvent_ContentId_EventType ON QoSEvent(ContentId, EventType);
CREATE INDEX IX_QoSEvent_SessionId ON QoSEvent(SessionId);
```

## API contracts

### catalog-api (port 3001)

#### GET /api/v1/content

List all content items (catalog grid).

**Response 200:**

```json
{
  "items": [
    {
      "id": "uuid",
      "title": "Cardiology Fundamentals: ECG Interpretation",
      "specialty": "Cardiology",
      "cmeCredits": 1.5,
      "durationSec": 3600,
      "posterUrl": "https://...",
      "progressSec": 120
    }
  ]
}
```

#### GET /api/v1/content/:id

Single content detail with stream URLs.

**Response 200:**

```json
{
  "id": "uuid",
  "title": "Cardiology Fundamentals: ECG Interpretation",
  "description": "...",
  "specialty": "Cardiology",
  "cmeCredits": 1.5,
  "durationSec": 3600,
  "posterUrl": "https://...",
  "streams": {
    "hls": "https://test-streams.mux.dev/x36xhzz/x36xhzz.m3u8",
    "dash": "https://storage.googleapis.com/shaka-demo-assets/angel-one/dash.mpd",
    "drm": null
  },
  "subtitles": []
}
```

#### GET /api/v1/progress/:userId

Continue-watching list.

#### PUT /api/v1/progress/:userId/:contentId

Body: `{ "positionSec": 450 }` — upsert watch position.

### analytics-api (port 3002)

#### POST /api/v1/events/batch

Ingest QoS events from player client.

**Request:**

```json
{
  "sessionId": "uuid",
  "events": [
    {
      "type": "playback_start",
      "contentId": "uuid",
      "clientTs": "2026-06-30T10:00:00Z",
      "payload": { "ttffMs": 1240 }
    },
    {
      "type": "heartbeat",
      "contentId": "uuid",
      "clientTs": "2026-06-30T10:00:30Z",
      "payload": { "currentTimeSec": 30, "bitrate": 2800000, "bufferLengthSec": 12 }
    }
  ]
}
```

**Response 202:** `{ "accepted": 2 }`

#### GET /api/v1/metrics/summary

Query params: `from`, `to` (ISO dates).

**Response 200:**

```json
{
  "avgStartupTimeMs": 1180,
  "rebufferRatio": 0.02,
  "errorRate": 0.005,
  "totalSessions": 142,
  "period": { "from": "...", "to": "..." }
}
```

#### GET /api/v1/metrics/by-content/:contentId

Per-title playback health aggregates.

## Shared TypeScript types (packages/shared-types)

```typescript
export interface ContentSummary {
  id: string;
  title: string;
  specialty: string;
  cmeCredits: number;
  durationSec: number;
  posterUrl: string;
  progressSec?: number;
}

export interface StreamInfo {
  hls: string;
  dash: string;
  drm: 'ClearKey' | 'Widevine' | null;
}

export type QoSEventType =
  | 'playback_start'
  | 'rebuffer_start'
  | 'rebuffer_end'
  | 'quality_change'
  | 'error'
  | 'heartbeat'
  | 'playback_end';

export interface QoSEvent {
  type: QoSEventType;
  contentId: string;
  clientTs: string;
  payload: Record<string, unknown>;
}
```

## Test stream URLs (seed data)

Use these public streams for seed content:

| Title | HLS | DASH |
|-------|-----|------|
| Cardiology Fundamentals | https://test-streams.mux.dev/x36xhzz/x36xhzz.m3u8 | https://dash.akamaized.net/akamai/bbb_30fps/bbb_30fps.mpd |
| Surgical Safety Protocol | https://storage.googleapis.com/shaka-demo-assets/angel-one-hls/hls.m3u8 | https://storage.googleapis.com/shaka-demo-assets/angel-one/dash.mpd |
| Pharmacology Update (DRM demo) | — | https://storage.googleapis.com/shaka-demo-assets/sintel-mp4-widescreen/dash.mpd |

## Front-end key components

| Component | Type | Responsibility |
|-----------|------|----------------|
| `ContentGrid` | Server | Render catalog from catalog-api |
| `ContentCard` | Server/Client | Poster, title, progress bar |
| `VideoPlayer` | Client | Shaka attach, load manifest |
| `PlayerControls` | Client | Custom UI overlay |
| `QualityMenu` | Client | Variant track selection |
| `AnalyticsClient` | Client | Batch and send QoS events |
| `QoSDashboard` | Client | Charts from analytics-api |

## Environment variables

### apps/web

```
CATALOG_API_URL=http://localhost:3001
ANALYTICS_API_URL=http://localhost:3002
NEXT_PUBLIC_APP_URL=http://localhost:3000
```

### services/catalog-api

```
DATABASE_URL=mssql://user:pass@localhost:1433/carestream
PORT=3001
CORS_ORIGIN=http://localhost:3000
```

### services/analytics-api

```
DATABASE_URL=mssql://user:pass@localhost:1433/carestream
PORT=3002
CORS_ORIGIN=http://localhost:3000
RATE_LIMIT_MAX=100
```

## Security requirements

- CSP via Next.js middleware (see [web-security-streaming.md](../skills/web-security-streaming.md))
- CORS locked to known origins on both APIs
- Input validation on `POST /events/batch` (max batch size, schema validation)
- No sensitive DRM keys in client code; ClearKey demo only
- Rate limiting on analytics-api ingest endpoint

## Performance targets

| Metric | Target |
|--------|--------|
| Catalog LCP | < 2.5s |
| Catalog INP | < 200ms |
| Player TTFF | < 2s on broadband |
| Player bundle (gzipped) | < 250KB excluding Shaka |
| Shaka lazy-loaded | Separate chunk |

## CI/CD (Azure DevOps)

Pipeline stages: **Build** (lint + unit test all packages) → **E2E** (Playwright) → **Deploy** (main branch only).

See [testing-build-cicd.md](../skills/testing-build-cicd.md) for sample `azure-pipelines.yml`.

GitHub Actions equivalent documented in `docs/adr/` or README for interview discussion.

## ADRs to write

1. **ADR-001:** Why Shaka Player over Video.js for DASH-first healthcare content
2. **ADR-002:** Microservices split (catalog vs analytics) vs monolith API
3. **ADR-003:** Manifest CDN strategy and test stream selection

## Related documents

- [carestream-todos.md](carestream-todos.md) — Implementation checklist
- [../study-plan.md](../study-plan.md) — Learning timeline
- [../skills/](../skills/) — Skill learning guides
