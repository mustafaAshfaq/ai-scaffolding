# WebDev Player Team — Study Plan

Tailored 12-week learning path for a senior .NET/Angular engineer pivoting to React/Next.js streaming front-end development.

## References

| Document | Purpose |
|----------|---------|
| [webdev_jobprofile.txt](webdev_jobprofile.txt) | Target role requirements |
| [candidate_profile.txt](candidate_profile.txt) | Your background and strengths |
| [project/carestream-spec.md](project/carestream-spec.md) | Capstone architecture |
| [project/carestream-todos.md](project/carestream-todos.md) | Capstone checklist |

---

## Candidate Gap Matrix

### Strengths to leverage

| Area | Your experience | How it maps to the role |
|------|-----------------|-------------------------|
| Senior engineering | 15 yrs enterprise (.NET, C#, Angular, Node) | Architecture, ADRs, code review, production triage |
| TypeScript / JS / HTML / CSS | Hands-on | TS syntax familiar; gap is React idioms |
| Angular | Primary front-end framework | Component thinking → ReactDOM hooks + Zustand |
| Backend & APIs | ASP.NET Web API, Node.js | Real metrics/catalog APIs in capstone |
| Database | 10 yrs MSSQL (DDL/DML, SSIS) | QoS events, watch history in SQL Server |
| DevOps | Azure DevOps, OpenShift, microservices | CI/CD in familiar tooling |
| Domains | Travel, healthcare | CareStream CME capstone narrative |

### Gaps to close

| Gap | Priority | Skill doc |
|-----|----------|-----------|
| React + Next.js | High | [angular-to-react-bridge.md](skills/angular-to-react-bridge.md), [typescript-react-nextjs.md](skills/typescript-react-nextjs.md) |
| HTML5 video / MSE / EME | Critical | [html5-video-mse-eme.md](skills/html5-video-mse-eme.md) |
| HLS / DASH / DRM / ABR | Critical | [hls-dash-protocols.md](skills/hls-dash-protocols.md) |
| Player SDKs (Shaka, HLS.js) | Critical | [video-player-sdks.md](skills/video-player-sdks.md) |
| Core Web Vitals / front-end perf | High | [performance-qos-analytics.md](skills/performance-qos-analytics.md) |
| Smart TV (Tizen, webOS) | Medium | [cross-platform-smart-tv.md](skills/cross-platform-smart-tv.md) |
| Tailwind / CSS-in-JS | Low–Medium | [css-styling-ui.md](skills/css-styling-ui.md) |
| Jest / RTL / Playwright | Medium | [testing-build-cicd.md](skills/testing-build-cicd.md) |
| Web security (streaming) | Medium | [web-security-streaming.md](skills/web-security-streaming.md) |

**Time allocation:** ~55% streaming/player, ~25% React/Next.js, ~20% capstone integration (backend, CI/CD, testing).

---

## 12-Week Timeline

| Week | Phase | Focus | Skill docs | Practice |
|------|-------|-------|------------|----------|
| 1 | 0 — Stack mapping | Angular→React mental model | [angular-to-react-bridge.md](skills/angular-to-react-bridge.md) | Monorepo skeleton; read job + candidate profiles |
| 1–2 | 1 — React + Next.js | Hooks, App Router, Server Components | [typescript-react-nextjs.md](skills/typescript-react-nextjs.md) | Catalog page calling Node catalog-api |
| 2 | 2 — Styling | Tailwind, accessible controls | [css-styling-ui.md](skills/css-styling-ui.md) | PlayerControls, ProgressBar, QualityMenu |
| 3–4 | 3 — Browser media | MSE, EME, video lifecycle | [html5-video-mse-eme.md](skills/html5-video-mse-eme.md) | Raw MSE segment append; EME flow diagram |
| 4–6 | 4 — Protocols | HLS, DASH, ABR, DRM | [hls-dash-protocols.md](skills/hls-dash-protocols.md) | Parse manifests; write ABR ADR |
| 6–8 | 5 — Player SDKs | Shaka, HLS.js, custom UI | [video-player-sdks.md](skills/video-player-sdks.md) | `useShakaPlayer` hook with typed events |
| 8–9 | 6 — Perf + QoS | Core Web Vitals, analytics-api | [performance-qos-analytics.md](skills/performance-qos-analytics.md) | Batch QoS to SQL; admin dashboard |
| 9–10 | 7 — Platform + security + tests | TV nav, CSP, Playwright | [cross-platform-smart-tv.md](skills/cross-platform-smart-tv.md), [web-security-streaming.md](skills/web-security-streaming.md), [testing-build-cicd.md](skills/testing-build-cicd.md) | E2E playback test; CSP middleware |
| 10–12 | 8 — Capstone | CareStream integration | All | Full project + portfolio README |

---

## Angular → React Bridge (Quick Reference)

| Angular | React / Next.js |
|---------|-----------------|
| NgModule | File-based composition; no modules required |
| Component + template | Function component + JSX |
| `@Input()` / `@Output()` | Props + callback props |
| `*ngIf` / `*ngFor` | `{condition && ...}` / `.map()` |
| Services + DI | Custom hooks + Context / Zustand |
| RxJS Observables | `useEffect` + event handlers; optional TanStack Query |
| NgRx store | Zustand or React Context |
| Angular Router | Next.js App Router file routes |
| `ngOnInit` / lifecycle hooks | `useEffect` with dependency array |
| Two-way `[(ngModel)]` | Controlled components (`value` + `onChange`) |
| Angular Material | Tailwind + headless UI / Radix |
| Karma + Jasmine | Jest + React Testing Library |
| `ng serve` / Angular CLI | `next dev` / Next.js CLI |

Full guide: [skills/angular-to-react-bridge.md](skills/angular-to-react-bridge.md)

---

## Skill Documents

| # | Document | Topic |
|---|----------|-------|
| 1 | [skills/angular-to-react-bridge.md](skills/angular-to-react-bridge.md) | Angular→React concept map |
| 2 | [skills/typescript-react-nextjs.md](skills/typescript-react-nextjs.md) | TS strict, React, Next.js App Router |
| 3 | [skills/css-styling-ui.md](skills/css-styling-ui.md) | Tailwind, player UI accessibility |
| 4 | [skills/html5-video-mse-eme.md](skills/html5-video-mse-eme.md) | HTML5 video, MSE, EME |
| 5 | [skills/hls-dash-protocols.md](skills/hls-dash-protocols.md) | HLS, DASH, ABR, DRM |
| 6 | [skills/video-player-sdks.md](skills/video-player-sdks.md) | Shaka, HLS.js, Video.js |
| 7 | [skills/performance-qos-analytics.md](skills/performance-qos-analytics.md) | Core Web Vitals, QoS metrics |
| 8 | [skills/cross-platform-smart-tv.md](skills/cross-platform-smart-tv.md) | Mobile, Tizen, webOS |
| 9 | [skills/web-security-streaming.md](skills/web-security-streaming.md) | XSS, CSP, CORS, DRM security |
| 10 | [skills/testing-build-cicd.md](skills/testing-build-cicd.md) | Jest, RTL, Playwright, CI/CD |

---

## Capstone: CareStream

Healthcare CME streaming platform — microservices (Node.js + MSSQL) + Next.js front-end with Shaka Player.

- **Spec:** [project/carestream-spec.md](project/carestream-spec.md)
- **Todos:** [project/carestream-todos.md](project/carestream-todos.md)

### Milestones

| Week | Milestone |
|------|-----------|
| 2 | Monorepo + catalog-api returning seeded content |
| 4 | Raw MSE demo + manifest parsing exercise complete |
| 6 | Shaka player playing HLS/DASH with custom controls |
| 8 | QoS events flowing to analytics-api + SQL |
| 10 | Admin dashboard + Playwright E2E passing |
| 12 | Azure DevOps pipeline green; portfolio README done |

---

## Job Requirements Checklist

Use this when reviewing progress against [webdev_jobprofile.txt](webdev_jobprofile.txt):

- [ ] React.js + Next.js + TypeScript (expert-level portfolio demo)
- [ ] HTML5, MSE, EME integration
- [ ] Shaka Player / HLS.js customization (UI, buffering, errors, ABR)
- [ ] HLS and MPEG-DASH protocol knowledge
- [ ] Core Web Vitals optimization
- [ ] Cross-browser + Smart TV (Tizen, webOS)
- [ ] Analytics & QoS (startup time, rebuffer, bitrate switches, failures)
- [ ] XSS, CSP, CORS, secure media playback
- [ ] Webpack/Vite concepts; Jest, RTL, Playwright
- [ ] CI/CD pipeline; production debugging narrative
