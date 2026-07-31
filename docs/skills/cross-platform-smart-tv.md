# Cross-Platform & Smart TV Development

## What it is

Streaming player apps must work on desktop browsers, mobile web, and Smart TV platforms (Samsung Tizen, LG webOS, VIDAA). TVs use remote controls (D-pad) instead of mouse, have lower-powered hardware, older browser engines, and strict memory limits. "Cross-browser" in this role means handling platform-specific quirks, not just Chrome vs Firefox.

## Why it matters for this role

The job explicitly requires seamless experience across desktop, mobile web, and Smart TV platforms (Tizen, webOS). The Player team ships to millions of users on TV — spatial navigation and 10-foot UI are as important as responsive CSS.

## Core concepts checklist

- [ ] Desktop: Chrome, Firefox, Safari, Edge — MSE/EME support matrix
- [ ] Mobile web: iOS Safari native HLS; `playsInline` to prevent fullscreen hijack
- [ ] Samsung Tizen: WebKit-based; Tizen Studio simulator; hosted web app packaging
- [ ] LG webOS: Chromium-based; webOS TV SDK; pointer vs key events
- [ ] VIDAA (Hisense): Chromium variant; similar patterns to webOS
- [ ] Spatial navigation: focus moves with arrow keys; no mouse hover on TV
- [ ] `:focus-visible` styling for remote navigation feedback
- [ ] `tabindex` management on interactive controls
- [ ] 10-foot UI: min 28–32px text; 64px+ touch/ focus targets
- [ ] Safe zone / overscan: keep critical UI 5% inset from edges
- [ ] Autoplay policies: muted autoplay often required on mobile
- [ ] Memory limits on TV: shorter buffer goals; avoid large JS bundles
- [ ] Platform detection vs feature detection (prefer latter)

## Platform comparison

| Platform | Engine | HLS native | MSE/DASH | Remote input |
|----------|--------|------------|----------|--------------|
| Chrome desktop | Blink | No | Yes | Mouse + keyboard |
| Safari desktop/iOS | WebKit | Yes | Limited | Touch |
| Tizen TV | WebKit | Varies | Yes | D-pad keys |
| webOS TV | Chromium | Varies | Yes | D-pad + pointer |
| Firefox desktop | Gecko | No | Yes | Mouse + keyboard |

## Learning resources

| Resource | Type | URL |
|----------|------|-----|
| Samsung Tizen TV docs | Official | https://developer.samsung.com/smarttv/develop/overview.html |
| LG webOS TV docs | Official | https://webostv.developer.lge.com/develop/overview/web-app-overview |
| W3C Spatial Navigation spec | Spec | https://w3c.github.io/csswg-drafts/css-nav-1/ |
| Shaka — Browser support | Reference | https://github.com/shaka-project/shaka-player#browser-support |
| Can I use — MSE | Reference | https://caniuse.com/mediasource |
| BBC TAL (TV app patterns) | Reference | https://github.com/bbc/tal |

## Hands-on exercises

1. **Focus ring audit:** Tab through all player controls; ensure visible focus on every interactive element.
2. **Keyboard map:** Implement Space, arrows, M, F shortcuts; test without mouse.
3. **iOS test:** Verify `playsInline` and native HLS on iPhone Safari.
4. **Tizen simulator:** Install Tizen Studio; load CareStream in TV Web Simulator; document issues.
5. **Throttled TV test:** Chrome DevTools CPU 4× slowdown + Slow 3G; verify playback still starts.
6. **Safe zone overlay:** Add dev-only overlay showing 5% overscan margin; verify controls stay inside.

## Interview / on-the-job topics

- How spatial navigation differs from tab order on desktop
- Safari HLS native path vs MSE — when you skip Shaka entirely
- TV memory constraints and buffer size tuning
- Packaging a hosted web app for Tizen vs webOS store submission
- Testing matrix strategy when you cannot own every physical device
- Pointer events on webOS magic remote vs pure D-pad

## Related skills

- [css-styling-ui.md](css-styling-ui.md) — TV-safe UI sizing
- [video-player-sdks.md](video-player-sdks.md) — Platform-specific player config
- [html5-video-mse-eme.md](html5-video-mse-eme.md) — Browser API support
- [web-security-streaming.md](web-security-streaming.md) — CSP on embedded TV runtimes
