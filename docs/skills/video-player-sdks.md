# Video Player SDKs (Shaka, HLS.js, Video.js, Dash.js)

## What it is

Player SDKs are JavaScript libraries that handle manifest fetching, segment download, MSE buffer management, ABR decisions, DRM license exchange, and text track rendering. **Shaka Player** (Google) supports DASH and HLS with strong DRM. **HLS.js** polyfills HLS for browsers without native HLS (Chrome, Firefox). **Video.js** is a general HTML5 player with plugin ecosystem. **Dash.js** is the DASH reference player.

## Why it matters for this role

The job requires deep hands-on experience with Shaka Player, HLS.js, Video.js, and Dash.js — including UI customization, buffering logic, error handling, and ABR behavior. Shaka + HLS.js is the recommended primary stack for CareStream and interview demos.

## Core concepts checklist

- [ ] Shaka Player: `shaka.Player`, `attach(videoElement)`, `load(manifestUri)`
- [ ] Shaka configuration: `player.configure({ streaming, abr, drm })`
- [ ] Shaka events: `error`, `buffering`, `adaptation`, `trackschanged`, `textchanged`
- [ ] Shaka error codes: `shaka.util.Error` categories (NETWORK, MEDIA, DRM, MANIFEST)
- [ ] Shaka UI overlay (optional) vs fully custom UI
- [ ] HLS.js: `new Hls()`, `loadSource()`, `attachMedia()`, `Hls.Events`
- [ ] When to use native HLS (Safari/iOS) vs HLS.js
- [ ] Video.js plugin model; `videojs()` initialization
- [ ] Dash.js MediaPlayer factory pattern
- [ ] Custom UI pattern: hide native/SDK controls; wire your buttons to SDK API
- [ ] Text tracks / subtitles: `getTextTracks()`, `selectTextTrack()`
- [ ] Quality override: disable ABR, `selectVariantTrack()`
- [ ] Retry/recovery strategies on network errors

## SDK selection (CareStream default)

| Browser | Primary SDK | Fallback |
|---------|-------------|----------|
| Chrome, Firefox, Edge | Shaka Player (DASH) | Shaka (HLS) |
| Safari (desktop + iOS) | Native HLS or Shaka | HLS.js if needed |
| Smart TV (Tizen, webOS) | Shaka Player | Platform-specific constraints |

## Learning resources

| Resource | Type | URL |
|----------|------|-----|
| Shaka Player docs | Official | https://shaka-player-demo.appspot.com/docs/api/index.html |
| Shaka Player GitHub | Source | https://github.com/shaka-project/shaka-player |
| Shaka demo app | Live demo | https://shaka-player-demo.appspot.com/demo/ |
| HLS.js docs | Official | https://github.com/video-dev/hls.js/blob/master/docs/API.md |
| HLS.js demo | Live demo | https://hlsjs.video-dev.org/demo/ |
| Video.js docs | Official | https://videojs.com/guides/ |
| Dash.js wiki | Official | https://github.com/Dash-Industry-Forum/dash.js/wiki |

## Custom UI integration pattern (Shaka)

```typescript
// Simplified useShakaPlayer hook structure
export function useShakaPlayer(videoRef: RefObject<HTMLVideoElement>) {
  const playerRef = useRef<shaka.Player | null>(null);

  useEffect(() => {
    if (!videoRef.current) return;
    const player = new shaka.Player();
    player.attach(videoRef.current);
    player.addEventListener('error', onError);
    player.addEventListener('adaptation', onAdaptation);
    playerRef.current = player;
    return () => { player.destroy(); };
  }, [videoRef]);

  const load = (url: string) => playerRef.current?.load(url);
  const getVariantTracks = () => playerRef.current?.getVariantTracks() ?? [];
  return { load, getVariantTracks };
}
```

## Hands-on exercises

1. **Shaka hello world:** Load Angel One DASH demo in a React component with `<video ref={videoRef} />`.
2. **Custom controls:** Hide default UI; wire play/pause/seek to Shaka `videoRef.current` and `player.getMediaElement()`.
3. **Error mapping:** Trigger errors (bad URL, CORS block); map Shaka error codes to user messages.
4. **Quality menu:** List variant tracks; implement manual quality selection with ABR disable/enable.
5. **HLS.js fallback:** Detect Safari native HLS support; branch to native vs HLS.js vs Shaka.
6. **Compare SDKs:** Load same stream in Video.js and Shaka; document bundle size and API differences.

## Interview / on-the-job topics

- Why Shaka over Video.js for a DASH-first product
- How you customize buffering goals (`streaming.rebufferingGoal`, `bufferingGoal`)
- Shaka `load()` startup sequence and time-to-first-frame optimization
- Handling `QUOTA_EXCEEDED` and buffer eviction policies
- Text track rendering: in-band vs sidecar WebVTT
- Player memory leaks: ensuring `destroy()` on component unmount

## Related skills

- [html5-video-mse-eme.md](html5-video-mse-eme.md) — Underlying APIs
- [hls-dash-protocols.md](hls-dash-protocols.md) — Manifest formats
- [typescript-react-nextjs.md](typescript-react-nextjs.md) — React hook patterns
- [performance-qos-analytics.md](performance-qos-analytics.md) — Instrumenting SDK events
