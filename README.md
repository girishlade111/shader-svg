# Shader SVG

An animated WebGL mesh-gradient "ghost" rendered inside an SVG silhouette —
built with Next.js 15, `@paper-design/shaders-react`, and Framer Motion.
The ghost floats, blinks, and its eyes follow your cursor.

> Built by **Girish Lade** · [ladestack.in](https://ladestack.in)

---

## Table of contents

- [Demo](#demo)
- [Features](#features)
- [Tech stack](#tech-stack)
- [Quickstart](#quickstart)
- [Project structure](#project-structure)
- [How it works](#how-it-works)
- [Configuration](#configuration)
- [Environment variables](#environment-variables)
- [Third-party integrations](#third-party-integrations)
- [Scripts](#scripts)
- [Deployment](#deployment)
- [Documentation](#documentation)
- [Roadmap / known issues](#roadmap--known-issues)
- [Contributing](#contributing)
- [License](#license)

---

## Demo

The home page (`/`) renders the full experience on a dark (`#101010`)
stage:

1. **Mesh-gradient body** — a live GPU shader (pastel pink → deep blue)
   clipped to a ghost/arch silhouette via SVG `<clipPath>`.
2. **Cursor-tracking eyes** — two ellipses follow your mouse with soft
   spring physics (±8 px clamp).
3. **Blink loop** — eyes squash vertically every 3 seconds.
4. **Float loop** — the whole figure hovers with squash-and-stretch
   (`y: 0 → -8 → 0`, 2.8 s, infinite).

## Features

- 🎨 **Real-time WebGL shader** — `@paper-design/shaders-react`
  `MeshGradient`, GPU-rendered, zero image assets for the artwork itself
- 👀 **Interactive eyes** — global `mousemove` tracking, spring-smoothed
  with Framer Motion
- 👻 **SVG-clipped rendering** — shader canvas inside `<foreignObject>`,
  cut to a custom path via `<clipPath>`
- 🌓 **Dark-mode active** — `next-themes` mounted (default `dark`, system-aware), Tailwind `darkMode:
  'class'`, full `.dark` token set (toggle UI not yet added)
- 🧩 **shadcn/ui foundation** — `components.json` manifest, full Radix
  primitive suite, `cn()` utility; ready for `shadcn add`
- 📊 **Vercel Analytics** — one-line `<Analytics />`, zero config
- 🔤 **Geist fonts** — self-hosted, no external font requests
- ⚙️ **Zero-config startup** — no `.env`, no API keys, no database

## Tech stack

| Layer        | Technology |
|--------------|------------|
| Framework    | Next.js 15 (App Router, React 19) |
| Language     | TypeScript (strict) |
| Styling      | Tailwind CSS v3 + shadcn/ui design tokens |
| Shader       | `@paper-design/shaders-react` (WebGL) |
| Animation    | Framer Motion |
| Theming      | `next-themes` (mounted, default dark) |
| Fonts        | Geist Sans + Geist Mono (self-hosted) |
| Analytics    | `@vercel/analytics` |
| Icons        | `lucide-react` |
| Package mgr  | pnpm (`pnpm-lock.yaml` committed) |
| Hosting      | Vercel (auto-deploy from GitHub) |
| AI builder   | v0.app (repo auto-syncs with the v0 project) |

## Quickstart

**Prerequisites:** Node.js 18.18+ (20+ recommended), pnpm 8+.

```bash
# 1. Clone
git clone https://github.com/girishlade111/shader-svg.git
cd shader-svg

# 2. Install (respects pnpm-lock.yaml — use pnpm, not npm)
pnpm install

# 3. Run
pnpm dev
```

Open http://localhost:3000 — the ghost appears immediately. No
environment variables, no setup wizard, no database.

## Project structure

```
shader-svg/
├── app/
│   ├── layout.tsx               # root layout: Geist fonts, metadata, <Analytics/>
│   ├── page.tsx                 # "/" — centers <MeshGradientSVG/> on a dark stage
│   └── globals.css              # Tailwind + shadcn tokens (the ACTIVE stylesheet)
├── components/
│   ├── mesh-gradient-svg.tsx    # ★ the animated shader ghost (core feature)
│   └── theme-provider.tsx       # next-themes wrapper (mounted in app/layout.tsx)
├── lib/
│   └── utils.ts                 # cn() — clsx + tailwind-merge class helper
├── public/                      # static assets (v0 placeholder images)
├── docs/
│   ├── ENVIRONMENT.md           # .env conventions + every config file explained
│   ├── INTEGRATIONS.md          # all third-party services, status & costs
│   └── DEVELOPER_GUIDE.md       # setup, component deep-dive, tasks, troubleshooting
├── components.json              # shadcn/ui CLI manifest
├── next.config.mjs              # strict build (lint/TS errors fail), images unoptimized
├── tailwind.config.ts           # Tailwind v3 + shadcn semantic tokens
├── postcss.config.mjs           # Tailwind PostCSS plugin
└── tsconfig.json                # strict TS, "@/*" path alias
```

## How it works

The entire visual is one component: `components/mesh-gradient-svg.tsx`.

```
┌─ <motion.div>            float loop (y, scaleY, 2.8s ∞)
│  ┌─ <svg viewBox="0 0 231 289">
│  │  ├─ <clipPath id="shapeClip">   ghost/arch silhouette path
│  │  ├─ <foreignObject clip-path>   HTML-in-SVG…
│  │  │   └─ <MeshGradient>          …WebGL canvas, clipped to silhouette
│  │  ├─ <motion.ellipse> ×2         eyes, spring-track the cursor
│  │  └─ <style jsx>                 blink keyframes (ry: 30 → 3)
```

- **Silhouette:** an SVG path defines the shape; the shader canvas is
  clipped to it, so the gradient *is* the ghost's body.
- **Eyes:** cursor position → offset from SVG center × 0.08, clamped to
  ±8 px → Framer Motion spring (`stiffness: 150, damping: 15`).
- **Palette:** `["#FFB3D9", "#87CEEB", "#4A90E2", "#2C3E50", "#1A1A2E"]`
  at `speed={1}` — edit the array to re-theme.

Full annotated walkthrough in `docs/DEVELOPER_GUIDE.md` §4.

## Configuration

Every config file is documented in `docs/ENVIRONMENT.md` §3. The short
version:

| File | Purpose |
|------|---------|
| `next.config.mjs` | strict build — ESLint/TS errors fail the build, `next/image` unoptimized |
| `tailwind.config.ts` | Tailwind v3, `darkMode: 'class'`, shadcn color tokens, accordion keyframes |
| `postcss.config.mjs` | Tailwind PostCSS plugin |
| `tsconfig.json` | strict TS, `@/*` → repo root alias |
| `components.json` | shadcn/ui manifest (style `default`, neutral base, CSS variables, lucide icons) |
| `.gitignore` | ignores `node_modules`, `.next/`, `.env*`, `.vercel` |

## Environment variables

**None required.** The app runs, builds, and deploys with zero env vars.

When the first secret is ever needed, the conventions are:

- Browser-exposed vars must be prefixed `NEXT_PUBLIC_` (they're inlined
  at **build** time — changing one needs a redeploy).
- Never commit secrets: `.gitignore` covers `.env*`; production secrets
  go in Vercel → Project Settings → Environment Variables.
- Local dev: copy `.env.example` → `.env.local` (git-ignored, highest
  precedence).

Details + per-file reference: `docs/ENVIRONMENT.md`.

## Third-party integrations

| Integration | Status | Notes |
|-------------|:------:|-------|
| `@paper-design/shaders-react` | ✅ used | WebGL mesh gradient; pinned via lockfile (`latest` in package.json — pin it) |
| Framer Motion | ✅ used | float loop + spring eye tracking |
| `@vercel/analytics` | ✅ used | zero-config page-view analytics on Vercel |
| `next-themes` | ✅ active | mounted in `app/layout.tsx`, `defaultTheme="dark"`, `enableSystem` |
| Geist fonts | ✅ used | self-hosted Sans + Mono |
| shadcn/ui + Radix + Tailwind | ⚠️ foundation | tokens wired; no `components/ui/*` installed yet |
| `lucide-react` | ✅ installed | unused — import on demand |
| Vercel + v0.app sync | ✅ active | v0 pushes auto-sync to this repo; treat v0 as source of truth for UI |

Full breakdown (props, costs, versioning risks): `docs/INTEGRATIONS.md`.

## Scripts

| Command | Does |
|---------|------|
| `pnpm dev` | dev server + hot reload → http://localhost:3000 |
| `pnpm build` | production build → `.next/` |
| `pnpm start` | serve the production build (after `pnpm build`) |
| `pnpm lint` | ESLint |

## Deployment

**Vercel (current, automatic).** This repo is connected to Vercel and
synced with a v0.app project: pushing to `main` (or deploying from the
v0 interface, which pushes here first) triggers a production deploy.
No environment variables to set. Analytics starts reporting on the
first deploy.

**Elsewhere.** Standard Next.js 15 app — works on Netlify, Cloudflare
Pages/Workers, or any Node host. For static export add `output: 'export'`
to `next.config.mjs` (`images.unoptimized` is already set, as static
export requires).

> ⚠️ **v0 sync caveat:** as long as the v0.app ↔ GitHub sync is on,
> v0 can overwrite local edits to `app/` and `components/`. See
> `docs/DEVELOPER_GUIDE.md` §7 before doing sustained local development.

## Documentation

| Document | Covers |
|----------|--------|
| `docs/ENVIRONMENT.md` | `.env` conventions, `.env.example` template, every config file explained |
| `docs/INTEGRATIONS.md` | all third-party services: usage, props, costs, health table |
| `docs/DEVELOPER_GUIDE.md` | setup, project tour, component deep-dive, common tasks, build/deploy, troubleshooting |

## Roadmap / known issues

- [x] Pin `@paper-design/shaders-react` (`0.0.57`) and `framer-motion`
      (`12.23.24`) to exact versions
- [x] Mount `ThemeProvider` in `app/layout.tsx` (default dark, system-aware)
- [x] Remove `ignoreDuringBuilds` / `ignoreBuildErrors` for real CI safety
- [x] Delete dead `styles/globals.css` duplicate
- [x] Scope SVG `id`/`querySelector` in `mesh-gradient-svg.tsx` with
      sanitized `useId()` + `svgRef` for multi-instance safety
- [x] Update `app/layout.tsx` metadata (was "v0 App")
- [x] Security: `next` 15.2.4 → **15.2.8** (CVE-2025-55182 React2Shell RCE,
      CVE-2025-66478, CVE-2025-55184/67779; minimal safe bump on 15.2 line)
- [x] QA fixes (2026-09-27, Playwright/Chromium, desktop + mobile):
      eyes centered until first pointer input (were pinned left);
      blink keyframes moved to `app/globals.css` (styled-jsx scoping never
      reached framer-motion's `className`); `initial={{cx,cy}}` on ellipses
      (framer-motion sampled missing attrs as `undefined` → console errors);
      `pointermove` + `pointerdown` instead of `mousemove` so touch taps/drags
      move the eyes; `app/icon.svg` favicon added
- [ ] Prune unused deps (forms/charts/carousel/OTP) if the roadmap
      doesn't need them
- [ ] Add a visible theme toggle (provider is mounted; UI control missing)

## Contributing

1. Fork / branch off `main`.
2. `pnpm install` and `pnpm dev`.
3. Keep components client-only where browser APIs are used; keep the
   `@/` alias convention.
4. Note the v0 sync caveat above — coordinate UI changes through v0
   while the sync is active.

## License

No license file ships with this repo yet. Until one is added, all rights
remain with the author — contact before reuse.

---

Built with ❤ by **Girish Lade** · [ladestack.in](https://ladestack.in)
