# TypeScript, React & Next.js

## What it is

TypeScript adds static types to JavaScript. React is a component-based UI library; Next.js is a React framework providing App Router, Server Components, SSR/SSG, and API routes. Together they form the primary front-end stack for the Player team role.

## Why it matters for this role

The job requires expert-level TypeScript, React, and Next.js. You will build typed player event models, manifest parsers, reusable UI components, and SSR catalog pages — all while keeping the video player as a client-side `'use client'` island within a Next.js app shell.

## Core concepts checklist

- [ ] TypeScript strict mode (`strict: true` in tsconfig)
- [ ] Discriminated unions for player events (`{ type: 'error'; code: number } | { type: 'playing' }`)
- [ ] Generic hooks and utility types (`Partial<T>`, `Pick<T>`, `Record<K,V>`)
- [ ] React function components with typed props interfaces
- [ ] `useState`, `useEffect`, `useRef`, `useReducer`, `useCallback`, `useMemo`
- [ ] Custom hooks encapsulating player logic (`useShakaPlayer`)
- [ ] Next.js App Router: `app/layout.tsx`, `app/page.tsx`, dynamic `[id]` routes
- [ ] Server Components vs Client Components (`'use client'` directive)
- [ ] Data fetching in Server Components (`async` page components)
- [ ] Dynamic imports (`next/dynamic`) for code-splitting the player bundle
- [ ] Environment variables (`NEXT_PUBLIC_*` vs server-only)
- [ ] Middleware for auth, CSP headers, redirects

## Learning resources

| Resource | Type | URL |
|----------|------|-----|
| TypeScript Handbook | Official | https://www.typescriptlang.org/docs/handbook/ |
| React docs | Official | https://react.dev |
| Next.js App Router docs | Official | https://nextjs.org/docs/app |
| Next.js Learn course | Tutorial | https://nextjs.org/learn |
| Total TypeScript (Matt Pocock) | Course | https://www.totaltypescript.com |
| Epic React (Kent C. Dodds) | Course | https://epicreact.dev |

## Key patterns for streaming apps

### Typed player events (discriminated union)

```typescript
type PlayerEvent =
  | { type: 'loading'; contentId: string }
  | { type: 'playing'; currentTime: number }
  | { type: 'buffering'; isBuffering: boolean }
  | { type: 'quality_change'; height: number; bitrate: number }
  | { type: 'error'; code: number; message: string };

function handlePlayerEvent(event: PlayerEvent) {
  switch (event.type) {
    case 'error':
      console.error(event.code, event.message);
      break;
    case 'quality_change':
      trackMetric('quality_change', { height: event.height });
      break;
  }
}
```

### Server vs Client split

| Layer | Component type | Why |
|-------|--------------|-----|
| Catalog home, detail metadata | Server Component | SEO, fast LCP, no JS for static content |
| Video player, controls | Client Component | Needs `HTMLVideoElement`, MSE, Shaka |
| Analytics beacon | Client Component | Browser `fetch` / `sendBeacon` |
| Admin dashboard charts | Client Component | Interactive charts, real-time polling |

### Dynamic import for player bundle

```typescript
import dynamic from 'next/dynamic';

const VideoPlayer = dynamic(() => import('@/components/player/VideoPlayer'), {
  ssr: false,
  loading: () => <PlayerSkeleton />,
});
```

## Hands-on exercises

1. **Strict TS config:** Enable `strict`, `noUncheckedIndexedAccess`, fix all type errors in a starter Next.js app.
2. **Server + client pages:** Build catalog page as Server Component fetching from catalog-api; watch page as Client Component with player.
3. **Custom hook:** Implement `useContent(id)` with loading/error/data states (like Angular service + async pipe).
4. **Player state machine:** Use `useReducer` for player states with typed actions.
5. **Middleware:** Add CSP headers in `middleware.ts` for all routes except admin.

## Interview / on-the-job topics

- When to use Server Components vs Client Components in a streaming app
- How you type third-party SDK events (Shaka) with declaration merging or wrappers
- Code splitting strategy for player SDK (~200KB+); impact on startup time
- SSR hydration pitfalls when mixing server HTML with client player mount
- Next.js caching: `fetch` cache, `revalidate`, and implications for catalog freshness

## Related skills

- [angular-to-react-bridge.md](angular-to-react-bridge.md) — If coming from Angular
- [video-player-sdks.md](video-player-sdks.md) — Shaka integration in React
- [performance-qos-analytics.md](performance-qos-analytics.md) — Core Web Vitals in Next.js
- [web-security-streaming.md](web-security-streaming.md) — CSP in middleware
