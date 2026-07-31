# Performance, Core Web Vitals & QoS Analytics

## What it is

**Core Web Vitals** are Google's user-centric performance metrics: **LCP** (Largest Contentful Paint), **INP** (Interaction to Next Paint), and **CLS** (Cumulative Layout Shift). For streaming apps, additional **QoS (Quality of Service)** metrics matter: startup time (time-to-first-frame), rebuffer ratio, rebuffer frequency, bitrate switches, and playback failure rate. Analytics frameworks collect these events for data-driven optimization.

## Why it matters for this role

The job emphasizes optimizing startup time, rebuffering, memory, rendering, and Core Web Vitals. You must design analytics to track startup time, buffering ratio, rebuffer frequency, bitrate switches, and playback failures — enabling rollout decisions and production triage.

## Core concepts checklist

- [ ] LCP: largest visible element paint time; target < 2.5s
- [ ] INP: responsiveness to user input; target < 200ms
- [ ] CLS: unexpected layout shift; target < 0.1
- [ ] Time to First Frame (TTFF): manifest fetch → first decoded frame
- [ ] Startup time breakdown: manifest load, first segment, license (DRM), decode
- [ ] Rebuffer events: `waiting` on video element; duration and frequency
- [ ] Rebuffer ratio: rebuffer time / total watch time
- [ ] Bitrate switch events: quality up/down with reason
- [ ] Playback failure rate (PFR): sessions ending in error / total sessions
- [ ] Engagement heartbeat: periodic beacon while playing
- [ ] `PerformanceObserver` API for web vitals in code
- [ ] Lighthouse CI for automated audits in PR pipeline
- [ ] Code splitting impact on LCP and player startup
- [ ] Memory profiling: Chrome DevTools Memory tab during 30+ min playback

## QoS event schema (CareStream)

| Event | Fields | When fired |
|-------|--------|------------|
| `playback_start` | sessionId, contentId, timestamp, ttffMs | First frame rendered |
| `rebuffer_start` | sessionId, contentId, currentTime | `waiting` event begins |
| `rebuffer_end` | sessionId, durationMs | `playing` resumes after buffer |
| `quality_change` | sessionId, fromBitrate, toBitrate, reason | ABR or manual switch |
| `error` | sessionId, code, category, message | Unrecoverable or retried error |
| `heartbeat` | sessionId, currentTime, bitrate, bufferLength | Every 30s while playing |
| `playback_end` | sessionId, watchDurationMs, completed | User leaves or video ends |

## Learning resources

| Resource | Type | URL |
|----------|------|-----|
| web.dev — Core Web Vitals | Official | https://web.dev/vitals/ |
| web.dev — Optimize LCP | Guide | https://web.dev/articles/optimize-lcp |
| Chrome DevTools — Performance | Guide | https://developer.chrome.com/docs/devtools/performance |
| Shaka — Performance tips | Guide | https://shaka-player-demo.appspot.com/docs/api/tutorial-network-and-buffering.html |
| Lighthouse CI | Tool | https://github.com/GoogleChrome/lighthouse-ci |
| web-vitals JS library | npm | https://github.com/GoogleChrome/web-vitals |
| Bitmovin Analytics (concepts) | Reference | https://bitmovin.com/demos/analytics/ |

## Hands-on exercises

1. **Baseline Lighthouse:** Run Lighthouse on CareStream catalog page; document LCP, INP, CLS; set improvement targets.
2. **TTFF measurement:** Instrument Shaka `load()` with `performance.now()` marks; log manifest vs first-frame breakdown.
3. **Rebuffer tracking:** Listen to `waiting`/`playing` on video; emit `rebuffer_start`/`rebuffer_end` to analytics-api.
4. **Code split impact:** Compare Lighthouse scores with player bundled vs dynamically imported.
5. **SQL aggregates:** Write MSSQL query for avg startup time and rebuffer ratio by contentId over last 7 days.
6. **Memory soak test:** Play 45-minute stream; take heap snapshot at start and end; document growth.

## Interview / on-the-job topics

- How you define and measure startup time consistently across browsers
- Trade-offs: preloading manifest vs saving disabled segment vs poster-only preload
- What causes high INP on a player page (heavy main thread during segment parse)
- How QoS metrics inform A/B experiment rollout decisions
- Distinguishing CDN issues vs client-side buffer underrun in metrics
- RUM (Real User Monitoring) vs synthetic Lighthouse testing

## Related skills

- [video-player-sdks.md](video-player-sdks.md) — SDK events to instrument
- [typescript-react-nextjs.md](typescript-react-nextjs.md) — Dynamic imports, SSR impact
- [testing-build-cicd.md](testing-build-cicd.md) — Lighthouse CI in pipeline
- [hls-dash-protocols.md](hls-dash-protocols.md) — Bitrate switch context
