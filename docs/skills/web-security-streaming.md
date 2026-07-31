# Web Security for Streaming

## What it is

Web security for streaming apps covers protecting users and content: **XSS** (Cross-Site Scripting) prevention, **CSP** (Content Security Policy) headers, **CORS** for cross-origin manifests and segments, and **DRM-related attack vectors** (license tampering, key extraction attempts). Secure media playback requires correct `crossOrigin` settings and trusted CDN origins.

## Why it matters for this role

The job requires strong understanding of XSS prevention, CSP, CORS, secure media playback, and DRM-related attack vectors. Player apps load scripts, fetch manifests from CDNs, and exchange license tokens — each is an attack surface.

## Core concepts checklist

- [ ] XSS types: stored, reflected, DOM-based
- [ ] React's default escaping; danger of `dangerouslySetInnerHTML`
- [ ] CSP directives: `default-src`, `script-src`, `connect-src`, `media-src`, `frame-src`
- [ ] CSP nonce/hash for inline scripts in Next.js
- [ ] CORS: `Access-Control-Allow-Origin` on manifest and segment servers
- [ ] `crossOrigin="anonymous"` on `<video>` for WebVTT and canvas capture
- [ ] Mixed content blocking (HTTPS page loading HTTP media)
- [ ] DRM: license server authentication; token expiry; domain binding
- [ ] EME security levels (L1 hardware vs L3 software)
- [ ] Tokenized manifest URLs (signed URLs, short TTL)
- [ ] Subresource Integrity (SRI) for CDN-hosted player SDK scripts
- [ ] `Referrer-Policy` for media requests

## CSP example for streaming app (Next.js middleware)

```
Content-Security-Policy:
  default-src 'self';
  script-src 'self' 'nonce-{NONCE}';
  connect-src 'self' https://catalog-api.example.com https://analytics-api.example.com https://*.cdn.example.com;
  media-src 'self' https://*.cdn.example.com blob:;
  img-src 'self' https: data:;
  style-src 'self' 'unsafe-inline';
  frame-ancestors 'none';
```

## Learning resources

| Resource | Type | URL |
|----------|------|-----|
| OWASP XSS Prevention | Guide | https://cheatsheetseries.owasp.org/cheatsheets/Cross_Site_Scripting_Prevention_Cheat_Sheet.html |
| MDN — CSP | Reference | https://developer.mozilla.org/en-US/docs/Web/HTTP/CSP |
| MDN — CORS | Reference | https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS |
| web.dev — CSP | Guide | https://web.dev/articles/csp |
| EME security model (W3C) | Spec | https://w3c.github.io/encrypted-media/ |
| Shaka — DRM configuration | Guide | https://shaka-player-demo.appspot.com/docs/api/tutorial-drm-config.html |

## Hands-on exercises

1. **CSP middleware:** Add CSP headers in Next.js `middleware.ts`; fix violations until player still loads.
2. **CORS debug:** Load cross-origin manifest without CORS; observe error; fix with proxy or CDN headers.
3. **XSS audit:** Search codebase for `dangerouslySetInnerHTML`, `eval`, dynamic script injection.
4. **DRM token flow:** Document where license tokens are stored (never localStorage for production keys).
5. **Security ADR:** Write ADR covering XSS, CSP, CORS, and DRM threat model for CareStream.

## Interview / on-the-job topics

- Why `media-src` must include CDN domains in CSP
- CORS preflight on segment requests vs manifest GET
- How Widevine L1 vs L3 affects content protection requirements
- Signed URL rotation and impact on player retry logic
- Preventing analytics endpoint abuse (event injection, rate limiting)
- CSP `'unsafe-inline'` for styles — trade-offs with Tailwind

## Related skills

- [html5-video-mse-eme.md](html5-video-mse-eme.md) — crossOrigin on video
- [hls-dash-protocols.md](hls-dash-protocols.md) — DRM and encryption
- [typescript-react-nextjs.md](typescript-react-nextjs.md) — CSP in middleware
- [testing-build-cicd.md](testing-build-cicd.md) — Security headers in CI checks
