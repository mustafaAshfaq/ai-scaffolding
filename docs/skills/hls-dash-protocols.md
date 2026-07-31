# HLS, DASH, ABR & DRM

## What it is

**HLS (HTTP Live Streaming)** and **MPEG-DASH** are adaptive streaming protocols. Content is split into small segments at multiple quality levels (bitrates/resolutions). A manifest file (M3U8 for HLS, MPD for DASH) tells the player what segments exist. **ABR (Adaptive Bitrate)** algorithms choose which quality to fetch based on network conditions. **DRM** encrypts segments; players use EME to obtain decryption keys from a license server.

## Why it matters for this role

The job requires expert knowledge of HLS and MPEG-DASH including manifest structure, segmenting, DRM, and adaptive bitrate. You will customize ABR behavior, debug manifest issues, and explain bitrate switches during interviews and production triage.

## Core concepts checklist

- [ ] HLS manifest types: Master playlist (`.m3u8`) vs Media playlist
- [ ] Key HLS tags: `#EXTM3U`, `#EXT-X-VERSION`, `#EXT-X-STREAM-INF`, `#EXT-X-TARGETDURATION`, `#EXT-X-MEDIA-SEQUENCE`, `#EXT-X-KEY`, `#EXT-X-ENDLIST`
- [ ] HLS segment formats: MPEG-TS (`.ts`) vs fMP4 (`.m4s`)
- [ ] DASH MPD structure: `<Period>`, `<AdaptationSet>`, `<Representation>`, `<SegmentTemplate>`
- [ ] Segment addressing: `$Number$`, `$Time$` templates
- [ ] ABR ladder design: 240p → 360p → 480p → 720p → 1080p with bitrates
- [ ] ABR goals: maximize quality without rebuffering; react to bandwidth estimates
- [ ] Buffer-based vs throughput-based ABR algorithms
- [ ] DRM systems: Widevine (Chrome/Android), FairPlay (Safari), PlayReady (Edge/legacy)
- [ ] ClearKey (test DRM), AES-128 (HLS sample encryption)
- [ ] CORS on manifest and segment origins
- [ ] Live vs VOD manifests (`#EXT-X-PLAYLIST-TYPE:VOD` vs live sliding window)

## Manifest anatomy (HLS master playlist example)

```
#EXTM3U
#EXT-X-VERSION:6
#EXT-X-STREAM-INF:BANDWIDTH=800000,RESOLUTION=640x360,CODECS="avc1.4d401f,mp4a.40.2"
360p/index.m3u8
#EXT-X-STREAM-INF:BANDWIDTH=2800000,RESOLUTION=1280x720,CODECS="avc1.4d401f,mp4a.40.2"
720p/index.m3u8
#EXT-X-STREAM-INF:BANDWIDTH=5000000,RESOLUTION=1920x1080,CODECS="avc1.4d401f,mp4a.40.2"
1080p/index.m3u8
```

## Learning resources

| Resource | Type | URL |
|----------|------|-----|
| Apple HLS Authoring Spec | Spec | https://developer.apple.com/documentation/http-live-streaming |
| MPEG-DASH spec overview | Spec | https://mpeg.org/standards/MPEG-DASH.html |
| HLS RFC 8216 | RFC | https://datatracker.ietf.org/doc/html/rfc8216 |
| Shaka Player — DASH/HLS tutorial | Guide | https://shaka-player-demo.appspot.com/docs/api/tutorial-manifest.html |
| Bitmovin — What is ABR streaming | Article | https://bitmovin.com/video-streaming/what-is-adaptive-bitrate-streaming |
| Axinom DRM test vectors | Test | https://github.com/Axinom/public-test-vectors |
| Unified Streaming — HLS/DASH examples | Samples | https://demo.unified-streaming.com |

## Test stream URLs (public)

| Name | HLS | DASH |
|------|-----|------|
| Big Buck Bunny | https://test-streams.mux.dev/x36xhzz/x36xhzz.m3u8 | https://dash.akamaized.net/akamai/bbb_30fps/bbb_30fps.mpd |
| Shaka demo | https://storage.googleapis.com/shaka-demo-assets/angel-one-hls/hls.m3u8 | https://storage.googleapis.com/shaka-demo-assets/angel-one/dash.mpd |
| ClearKey DRM (Shaka) | — | https://storage.googleapis.com/shaka-demo-assets/sintel-mp4-widescreen/dash.mpd |

## Hands-on exercises

1. **Parse a master playlist:** Fetch an M3U8 URL; extract all `#EXT-X-STREAM-INF` bandwidth and resolution values into a table.
2. **Compare HLS vs DASH:** Load the same content in both formats; document structural differences in a one-page ADR.
3. **ABR observation:** Play Big Buck Bunny; throttle network in DevTools (Slow 3G); log quality switches from Shaka events.
4. **DRM flow:** Load Shaka ClearKey demo; trace license request in Network tab; document request/response headers.
5. **Broken manifest:** Introduce a bad segment URL in a test manifest; observe player error behavior and recovery options.

## Interview / on-the-job topics

- Master vs media playlist in HLS; when each is fetched
- How ABR estimates bandwidth (throughput vs buffer occupancy)
- Why `#EXT-X-KEY` matters for encrypted HLS
- DASH `$Time$` vs `$Number$` segment templates
- Multi-period DASH for ad insertion (SSAI context)
- CDN failover when manifest fetch fails mid-playback

## Related skills

- [html5-video-mse-eme.md](html5-video-mse-eme.md) — Browser APIs consuming segments
- [video-player-sdks.md](video-player-sdks.md) — SDK manifest parsing
- [web-security-streaming.md](web-security-streaming.md) — DRM attack vectors
- [performance-qos-analytics.md](performance-qos-analytics.md) — Bitrate switch metrics
