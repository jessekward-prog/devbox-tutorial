# Professional Touches

**Trigger:** you say "add professional touches", "make it feel more native", "polish it up", or similar, once the core functionality of an app already works.

**Goal:** make a web app feel like a native mobile app.

## Before touching anything

1. **Read the codebase first.** Check `package.json` for the stack (React? Vue? Vanilla? Vite? Next?). Check existing components for what's already there.
2. **Only apply what fits.** A vanilla JS app doesn't get Framer Motion. A Next.js app handles routing differently. A Tailwind app uses utility classes, not custom CSS. Match the patterns already in the code.
3. **Don't add dependencies that aren't needed.** If there are only 2 buttons, a CSS `:active` scale is fine. If there's real list animation happening, then reach for a motion library.
4. **Skip items already done well.** If toasts exist, don't rebuild them. If tap targets are already 44px, move on.
5. After reading, **pick the 3–5 highest-impact items** for this specific app and do those. Don't blindly run the whole list top to bottom.

---

## 1. Motion — the baseline

If the app doesn't already have a motion library and genuinely needs orchestrated animation (staggered lists, shared-element transitions), add one (e.g. `framer-motion` for React). For anything simpler, plain CSS transitions are enough — don't reach for a dependency to animate one button.

### Page transitions
```js
const pageVariants = {
  initial: { opacity: 0, y: 8 },
  animate: { opacity: 1, y: 0, transition: { duration: 0.25, ease: [0.16, 1, 0.3, 1] } },
  exit:    { opacity: 0, y: -8, transition: { duration: 0.15 } },
};
```

### List stagger
```js
const staggerContainer = { animate: { transition: { staggerChildren: 0.06 } } };
const staggerItem = { initial: { opacity: 0, y: 10 }, animate: { opacity: 1, y: 0 } };
```

### Button tap feedback
Every interactive element gets a subtle press-down scale (`0.95` for normal buttons, `0.97` for large cards) — either via a motion library's tap gesture or a plain `:active { transform: scale(0.95); }`.

### Modals
Spring or eased entrance (scale 0.95 → 1, opacity 0 → 1). Backdrop fades separately from the modal content.

### Number/value changes
Animate counters in/out on change rather than snapping — a `popLayout`-style crossfade reads as intentional, not glitchy.

---

## 2. Touch targets

Every tappable element must be at least **44×44px** (iOS HIG minimum):
```css
.tap-target { min-height: 44px; min-width: 44px; }
```

Remove the default mobile tap flash:
```css
* { -webkit-tap-highlight-color: transparent; }
```

Select-all on numeric input focus so re-entering a value doesn't require manual selection:
```js
onFocus={e => e.target.select()}
```

---

## 3. Viewport & scroll

```css
body {
  min-height: 100dvh;        /* dynamic viewport — not the buggy 100vh on mobile browsers */
  overscroll-behavior: none; /* no rubber-band scroll bleeding into the page background */
}
```

---

## 4. Haptic feedback

```js
const useHaptics = () => ({
  tap:     () => navigator.vibrate?.(15),
  success: () => navigator.vibrate?.([20, 40, 30, 40, 60]),
  error:   () => navigator.vibrate?.([80, 30, 80]),
});
```

Fire `tap()` on form submits, set completions, toggles. Fire `success()` on milestone events. Only fires on devices/browsers that support the Vibration API — the `?.` guard makes it a silent no-op everywhere else (notably iOS Safari).

---

## 5. Toast / notification system

Slide-up toasts from the bottom, with easing rather than a hard cut. Three tiers, roughly:
- **Toast** — slides up, auto-dismisses after ~3s
- **Center modal** — important confirmations (success/error/warning)
- **Celebration moment** — milestones worth a bigger flourish (streaks, level-ups, first success)

**Never use `alert()`, `confirm()`, or `prompt()`** — they block the main thread, can't be styled, and look like a browser error rather than part of the app.

---

## 6. Loading states

Never show a blank screen. Options:
- Skeleton screens — pulsing grey shapes matching the content layout
- Spinner with a smooth linear rotation, not a stepped one
- Optimistic UI — update state instantly, revert only if the request actually fails

---

## 7. Micro-interactions

- **Checkmarks** — spring/bounce entrance: scale 0 → 1, slight rotation settle
- **Progress bars** — animate width on change (`transition: width 0.6s ease-out`), consider a colour shift at 100%
- **Completion states** — border colour change + subtle background tint, not just a text label flip
- **Delete confirmations** — always an explicit confirm step with a brief overlay, never instant on first tap

---

## 8. PWA shell (if not already there)

- `manifest.json` — `"display": "standalone"`, sensible orientation lock, icon sizes 192 + 512 + a maskable variant
- Service worker — network-first is the safest default for most content; explicitly skip caching `/api/` and `/auth/` routes so data never goes stale silently
- Meta tags in `<head>`:
  ```html
  <meta name="mobile-web-app-capable" content="yes">
  <meta name="apple-mobile-web-app-capable" content="yes">
  <meta name="apple-mobile-web-app-status-bar-style" content="black-translucent">
  <link rel="apple-touch-icon" href="/apple-touch-icon.png">
  ```

---

## 9. Typography

Pick one display face and one body/mono face and stay consistent — mixing arbitrary font sizes and families across a small app reads as unfinished. `IBM Plex Mono` + `Space Grotesk` is a solid, fast-loading default pairing if you don't already have a brand direction.

Utility classes for the two roles, so nothing hardcodes a font family inline:
```css
.font-mono    { font-family: var(--font-mono); }
.font-display { font-family: var(--font-display); }
```

---

## 10. CSS theme foundation

CSS custom properties on `:root` for every colour, so a theme swap or dark/light toggle is a token change, not a find-and-replace:
```css
:root {
  --color-bg-0: ...;   /* page background */
  --color-bg-1: ...;   /* card surface */
  --color-bg-2: ...;   /* raised card / input */
  --color-bg-3: ...;   /* hover states */
  --color-accent: ...; /* primary action colour */
  --color-border: ...; /* subtle dividers */
  --color-text-primary: ...;
  --color-text-secondary: ...;
  --color-text-tertiary: ...;
}
```

Never hardcode hex values inside components — always reference a variable.

---

## 11. Android back button / swipe-back navigation

Two-layer approach for apps with modals and in-app page history:

### Modal/drawer intercept
```js
useEffect(() => {
  if (isOpen) {
    history.pushState(null, '');
    const handler = () => onClose();
    window.addEventListener('popstate', handler);
    return () => window.removeEventListener('popstate', handler);
  }
}, [isOpen]);
```

### In-app page history (navigate back through pages to root)

Maintain a nav stack and push a history entry on every page change. The `popstate` listener walks the stack; when it empties, the browser handles back naturally (exits the PWA or goes to the previous site).

```js
// nav-stack.js — singleton
const stack = [];

export function pushPage(pageId) {
  stack.push(pageId);
  history.pushState({ pageId }, '');
}

export function initBackHandler(goToPage, rootPageId) {
  window.addEventListener('popstate', () => {
    stack.pop();
    const prev = stack[stack.length - 1] ?? rootPageId;
    goToPage(prev, { skipHistory: true }); // don't pushState again
  });
}
```

**Swipe-back on Android's predictive back gesture** works automatically once the History API stack is set up correctly and the PWA is in `"display": "standalone"` — no extra code needed.

---

## 12. Persistent global UI

Keep these mounted in the app shell, not inside individual pages, so they survive navigation:
- Timers (rest/countdown/etc.)
- Audio elements
- A global toast container

---

## 13. Icon / logo generation (placeholder)

If the app has no icon yet, generate two files before wiring up the PWA manifest.

### `public/icon.svg` — source of truth
Simple: accent-coloured square, rounded corners, the app's initial(s) centred in a mono font.

```svg
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 512 512">
  <rect width="512" height="512" rx="96" fill="#YOUR_ACCENT"/>
  <text x="256" y="340" text-anchor="middle"
        font-family="monospace" font-size="260"
        font-weight="700" fill="#ffffff">X</text>
</svg>
```

### PNG exports for the manifest (192 + 512 + maskable)

Prefer whatever image tool is already on the machine over adding a new dependency — `rsvg-convert`, `inkscape --export-*`, or ImageMagick's `convert` can all rasterize an SVG in one command. Reach for a Node library like `sharp` only if none of those are available.

```bash
rsvg-convert -w 192 -h 192 public/icon.svg -o public/icon-192.png
rsvg-convert -w 512 -h 512 public/icon.svg -o public/icon-512.png
# maskable: keep the important content inside a ~10% safe-zone margin
```

Wire into `manifest.json`:
```json
"icons": [
  { "src": "/icon-192.png", "sizes": "192x192", "type": "image/png" },
  { "src": "/icon-512.png", "sizes": "512x512", "type": "image/png" },
  { "src": "/icon-maskable.png", "sizes": "512x512", "type": "image/png", "purpose": "maskable" }
]
```

These are **placeholders** — mark the SVG with a `<!-- TODO: replace with real logo -->` comment so it's obviously not final.

---

## What NOT to do

- Don't use `alert()`, `confirm()`, or `prompt()` — replace with in-app modals/toasts
- Don't use full-page navigation (`window.location.href`) for in-app routing once you have client-side routing set up
- Don't let inputs zoom on focus — set `font-size: 16px` minimum on inputs for iOS
- Don't hardcode colours — always go through the theme tokens
- Don't skip loading states — a blank screen reads as broken, even when it's just slow
