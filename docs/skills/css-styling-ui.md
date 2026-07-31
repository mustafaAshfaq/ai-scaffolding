# CSS, Tailwind & Player UI

## What it is

Tailwind CSS is a utility-first CSS framework — you compose styles via class names like `flex`, `p-4`, `bg-slate-900` instead of writing separate CSS files. styled-components is a CSS-in-JS library (also listed in the job profile) that scopes styles to React components. For player UI, you need responsive layouts, accessible controls, and overlays that work on desktop, mobile, and TV.

## Why it matters for this role

The job requires CSS-in-JS (styled-components/Tailwind) and building reusable player chrome — controls, progress bars, quality menus, error overlays. You already know HTML/CSS; this skill doc focuses on utility-first patterns and accessibility specific to video players.

## Core concepts checklist

- [ ] Tailwind setup in Next.js (`tailwind.config.ts`, `globals.css` with `@tailwind` directives)
- [ ] Responsive breakpoints (`sm:`, `md:`, `lg:`, `xl:`)
- [ ] Flexbox and grid for player layout (video + controls overlay)
- [ ] Absolute positioning for control bar overlay on video
- [ ] Focus states (`focus-visible:ring-2`) for keyboard and TV remote navigation
- [ ] `prefers-reduced-motion` for accessibility
- [ ] ARIA roles: `role="slider"` for seek bar, `aria-label` on icon buttons
- [ ] Hidden native controls (`controls={false}` on `<video>`) with custom UI
- [ ] styled-components basics (optional): `styled.div`, theme provider
- [ ] Dark mode via Tailwind `dark:` variant or class strategy

## Player UI component patterns

### Control bar layout

```
┌─────────────────────────────────────────┐
│                                         │
│              VIDEO AREA                 │
│                                         │
├─────────────────────────────────────────┤
│ ▶  ───●────────────  12:34 / 45:00  🔊 ⛶ │
└─────────────────────────────────────────┘
```

- Video container: `relative w-full aspect-video bg-black`
- Controls overlay: `absolute bottom-0 inset-x-0 bg-gradient-to-t from-black/80 p-4`
- Buttons: min 44×44px touch target (`min-h-11 min-w-11`) for mobile/TV

### Accessible seek bar

```tsx
<input
  type="range"
  min={0}
  max={duration}
  value={currentTime}
  onChange={handleSeek}
  aria-label="Seek"
  aria-valuemin={0}
  aria-valuemax={duration}
  aria-valuenow={currentTime}
  className="w-full h-1 accent-red-600 cursor-pointer"
/>
```

## Learning resources

| Resource | Type | URL |
|----------|------|-----|
| Tailwind CSS docs | Official | https://tailwindcss.com/docs |
| Tailwind + Next.js guide | Official | https://tailwindcss.com/docs/guides/nextjs |
| styled-components docs | Official | https://styled-components.com/docs |
| WAI-ARIA Authoring Practices — Slider | Spec | https://www.w3.org/WAI/ARIA/apg/patterns/slider/ |
| Refactoring UI (tips) | Book | https://www.refactoringui.com |
| Headless UI (accessible components) | Library | https://headlessui.com |

## Hands-on exercises

1. **Player chrome:** Build `PlayerControls`, `ProgressBar`, `VolumeSlider`, `QualityMenu` with Tailwind only.
2. **Responsive:** Player full-width on mobile; max-width container on desktop with 16:9 aspect ratio.
3. **Keyboard:** Space = play/pause, arrows = seek ±10s, M = mute, F = fullscreen — visible focus rings.
4. **Loading/error states:** Buffering spinner overlay; error card with retry button styling.
5. **Compare styled-components:** Reimplement control bar with styled-components to understand CSS-in-JS trade-offs (runtime cost vs colocation).

## Interview / on-the-job topics

- Tailwind vs styled-components vs CSS Modules — when each fits in a large player codebase
- How you hide native video controls and maintain accessibility
- TV UI: 10-foot interface guidelines (font size, focus, safe zones)
- CSS performance: avoid layout thrashing during progress bar updates (use `transform` not `width` where possible)
- High-contrast mode and caption styling requirements

## Related skills

- [typescript-react-nextjs.md](typescript-react-nextjs.md) — Component structure
- [cross-platform-smart-tv.md](cross-platform-smart-tv.md) — TV-specific UI patterns
- [video-player-sdks.md](video-player-sdks.md) — Wiring controls to Shaka API
