# Developer Guide

Everything needed to run, understand, extend, and ship **shader-svg**.
For environment variables and config files see `docs/ENVIRONMENT.md`;
for third-party services see `docs/INTEGRATIONS.md`.

---

## 1. Prerequisites

- **Node.js** 18.18+ or 20+ (Next.js 15 requirement)
- **pnpm** 8+ — the repo has a `pnpm-lock.yaml`; use pnpm, not npm/yarn
  (several deps are pinned to `latest` in `package.json`, so only the
  lockfile gives you reproducible installs)

Check:

```bash
node -v   # >= 18.18
pnpm -v
```

## 2. Setup & scripts

```bash
git clone https://github.com/girishlade111/shader-svg.git
cd shader-svg
pnpm install
```

| Script        | Command        | What it does |
|---------------|----------------|--------------|
| `pnpm dev`    | `next dev`     | dev server at http://localhost:3000, hot reload |
| `pnpm build`  | `next build`   | production build into `.next/` |
| `pnpm start`  | `next start`   | serves the production build (run `build` first) |
| `pnpm lint`   | `next lint`    | ESLint (note: `next.config.mjs` ignores lint errors during builds) |

No `.env` file is needed — the app runs with zero configuration.

## 3. Project structure

```
shader-svg/
├── app/
│   ├── layout.tsx        # root layout: Geist fonts, <Analytics/>, metadata
│   ├── page.tsx          # home route — renders <MeshGradientSVG/>
│   └── globals.css       # Tailwind + shadcn design tokens (ACTIVE)
├── components/
│   ├── mesh-gradient-svg.tsx  # the animated shader ghost (main feature)
│   └── theme-provider.tsx     # next-themes wrapper (not yet wired up)
├── lib/
│   └── utils.ts          # cn() — clsx + tailwind-merge helper
├── public/               # static assets (placeholder images from v0 scaffold)
├── styles/
│   └── globals.css       # byte-identical duplicate of app/globals.css (unused)
├── docs/
│   ├── ENVIRONMENT.md    # .env + config reference
│   ├── INTEGRATIONS.md   # third-party services
│   └── DEVELOPER_GUIDE.md# this file
├── components.json       # shadcn/ui CLI manifest
├── next.config.mjs       # build behavior (lint/TS errors ignored, images unoptimized)
├── tailwind.config.ts    # Tailwind v3 + shadcn tokens
├── postcss.config.mjs    # Tailwind PostCSS plugin
├── tsconfig.json         # strict TS, @/* path alias
└── package.json          # deps (see INTEGRATIONS.md)
```

**Routing.** This is the Next.js App Router. `app/page.tsx` is `/`.
There are no other routes. `app/layout.tsx` wraps everything.

## 4. How the main component works

`components/mesh-gradient-svg.tsx` is the entire product: an animated
"ghost" whose body is a live WebGL mesh gradient, with eyes that follow
your cursor. It is a client component (`"use client"`) because it uses
`window` listeners and animation state.

### 4.1 The silhouette (SVG + clipPath)

```tsx
<svg viewBox="0 0 231 289">
  <defs>
    <clipPath id="shapeClip">
      <path d="M230.809 115.385V249.411C…Z" />  {/* ghost/arch outline */}
    </clipPath>
  </defs>
  <foreignObject width="231" height="289" clipPath="url(#shapeClip)">
    <div className="w-full h-full">
      <MeshGradient colors={colors} className="w-full h-full" speed={1} />
    </div>
  </foreignObject>
  {/* eyes drawn on top */}
</svg>
```

Technique: the `<MeshGradient>` canvas renders inside a `<foreignObject>`
(HTML-in-SVG), and the SVG `clipPath` cuts it to the ghost shape. The
eyes are plain SVG ellipses drawn **after** the foreignObject so they sit
on top of the gradient.

> ⚠️ `id="shapeClip"` is a global SVG id. If you ever render two
> `<MeshGradientSVG/>` instances on one page, the ids collide. Fix by
> accepting an `id` prop or using React's `useId()`.

### 4.2 Eye tracking

```tsx
const [mousePosition, setMousePosition] = useState({ x: 0, y: 0 })
const [eyeOffset, setEyeOffset] = useState({ x: 0, y: 0 })

// 1. track the cursor globally
useEffect(() => {
  const handleMouseMove = (e: MouseEvent) =>
    setMousePosition({ x: e.clientX, y: e.clientY })
  window.addEventListener("mousemove", handleMouseMove)
  return () => window.removeEventListener("mousemove", handleMouseMove)
}, [])

// 2. convert cursor position → clamped eye offset
useEffect(() => {
  const rect = document.querySelector("svg")?.getBoundingClientRect()
  if (rect) {
    const deltaX = (mousePosition.x - (rect.left + rect.width / 2)) * 0.08
    const deltaY = (mousePosition.y - (rect.top + rect.height / 2)) * 0.08
    const maxOffset = 8
    setEyeOffset({
      x: Math.max(-maxOffset, Math.min(maxOffset, deltaX)),
      y: Math.max(-maxOffset, Math.min(maxOffset, deltaY)),
    })
  }
}, [mousePosition])
```

Tuning knobs: `0.08` = follow sensitivity, `8` = max pixel offset.
The `<motion.ellipse>` elements animate `cx`/`cy` with a spring
(`stiffness: 150, damping: 15`) — that's what makes the eyes lag softly
instead of snapping.

> ⚠️ `document.querySelector("svg")` grabs the **first** `<svg>` in the
> document. Fragile if the page grows more SVGs — scope it with a `ref`
> instead.

### 4.3 Blink animation

Styled-JSX keyframes squash the eyes vertically every 3 seconds:

```css
@keyframes blink {
  0%, 90%, 100% { ry: 30; }
  95%            { ry: 3; }   /* the blink */
}
```

`ry` is an SVG geometry property; animating it via CSS works in
Chromium/Firefox/Safari for ellipse elements.

### 4.4 Floating motion

The outer `<motion.div>` loops `y: [0, -8, 0]` and
`scaleY: [1, 1.08, 1]` over 2.8s (`easeInOut`, infinite) with
`transformOrigin: "top center"` — a gentle hover with squash-and-stretch
pivoting from the top, like a hanging figure.

### 4.5 Color palette

```ts
const colors = [
  "#FFB3D9", // pastel pink
  "#87CEEB", // sky blue
  "#4A90E2", // medium blue
  "#2C3E50", // dark blue-gray
  "#1A1A2E", // very dark blue
]
```

Passed straight to `<MeshGradient colors speed>`. To re-theme, just
replace this array.

## 5. Common tasks

### 5.1 Change the gradient colors or speed

Edit the `colors` array and the `speed` prop in
`components/mesh-gradient-svg.tsx`. `speed={1}` is default; `2` doubles
the animation rate, `0` freezes it.

### 5.2 Add a shadcn/ui component

```bash
pnpm dlx shadcn@latest add button dialog
```

This reads `components.json` and drops files into `components/ui/`
using the `@/components` alias. Then:

```tsx
import { Button } from "@/components/ui/button"
```

### 5.3 Enable dark/light theme toggle

`next-themes` is installed but not mounted. Wire it up:

1. Wrap children in `app/layout.tsx`:

```tsx
import { ThemeProvider } from '@/components/theme-provider'

<html lang="en" suppressHydrationWarning>
  <body>
    <ThemeProvider attribute="class" defaultTheme="system" enableSystem>
      {children}
    </ThemeProvider>
    <Analytics />
  </body>
</html>
```

2. Add a toggle using `useTheme()` from `next-themes` anywhere client-side.
   The `.dark` token set in `app/globals.css` and `darkMode: ['class']`
   in `tailwind.config.ts` are already in place.

### 5.4 Add a new page/route

Create `app/about/page.tsx` — it becomes `/about` automatically.
Shared UI goes in `app/layout.tsx`.

### 5.5 Use path aliases

`@/` maps to the repo root (`tsconfig.json` paths). Examples:
`@/components/...`, `@/lib/utils`, `@/app/globals.css`.

## 6. Build & deployment

### 6.1 Production build locally

```bash
pnpm build   # type/lint errors are ignored per next.config.mjs
pnpm start   # serves on http://localhost:3000
```

### 6.2 Deploy to Vercel (recommended)

The project is already linked to Vercel and auto-deploys from GitHub:

1. Push to `main` → Vercel builds and deploys automatically.
2. No environment variables to configure (see `docs/ENVIRONMENT.md`).
3. `@vercel/analytics` starts reporting once deployed.

Because this repo is **synced with v0.app**, deploying from the v0
interface also pushes code here, which then triggers the Vercel deploy.
Pick one source of truth (see §7).

### 6.3 Deploy elsewhere (Netlify / Cloudflare Pages)

It's a standard Next.js 15 app. Static export is possible but needs
`output: 'export'` in `next.config.mjs` — note `images.unoptimized`
is already set, which static export requires.

## 7. Working with the v0.app sync

This repo auto-receives pushes from the linked v0.app project. Rules of
engagement:

- **v0 is the source of truth for UI** as long as the sync is on. Local
  commits to `app/` or `components/` can be overwritten by the next v0
  push — v0 force-pushes its generated tree.
- **Safe local work:** `docs/`, `README.md`, and non-UI config are rarely
  touched by v0, but "rarely" ≠ "never". Keep local-only branches for
  experiments.
- **To take full local ownership:** disconnect the GitHub sync in the
  v0.app project settings. After that, this repo is a normal Next.js
  project.

## 8. Code quality notes & tech debt

Things a maintainer should know before scaling this up:

1. **`next.config.mjs` ignores TS and ESLint errors during builds.**
   Fine for a prototype; remove both flags before treating this as
   production code so failures surface in CI.
2. **Two deps float on `latest`** (`@paper-design/shaders-react`,
   `framer-motion`). Pin exact versions to avoid surprise visual changes.
3. **`styles/globals.css` is dead code** — byte-identical copy of
   `app/globals.css`, imported nowhere. Delete it.
4. **Global SVG id + global `querySelector("svg")`** in
   `mesh-gradient-svg.tsx` break with multiple instances — use `useId()`
   and a `ref`.
5. **Unused dependency surface** — form/chart/carousel/OTP libraries are
   installed but unimported. They don't bloat the client bundle (tree
   shaken / never imported), but they slow installs. Prune if the roadmap
   doesn't need them (`pnpm remove …`).
6. **`theme-provider.tsx` is unwired** — either mount it (see §5.3) or
   delete it to avoid confusion.
7. **Metadata still says "v0 App / Created with v0"** in
   `app/layout.tsx` — update title/description/OG tags before public
   launch.

## 9. Troubleshooting

| Symptom | Likely cause | Fix |
|---------|--------------|-----|
| `pnpm dev` port conflict | something on :3000 | `pnpm dev -- -p 3001` |
| Gradient renders black/blank | WebGL unavailable (old GPU, headless) | shaders-react needs a real GPU context; check browser console |
| Eyes don't follow cursor | another `<svg>` earlier in DOM | scope the query with a `ref` (see §4.2) |
| `pnpm install` resolves different versions | `latest` tags + stale lockfile | delete lockfile only as last resort; prefer `pnpm install --frozen-lockfile` in CI |
| Styles look unstyled | Tailwind content globs miss the file | check `tailwind.config.ts` `content` covers the new path |
| Hydration warning after adding ThemeProvider | next-themes mutates `<html>` class | add `suppressHydrationWarning` to `<html>` |
| Local edits vanish after a deploy | v0.app sync overwrote them | commit/push from v0, or disconnect the sync (§7) |
