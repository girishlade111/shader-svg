# Environment & Configuration

This document covers every environment variable convention and configuration
file in the **shader-svg** project. It answers two questions: *what do I put
in a `.env` file?* and *what does each config file do?*

## 1. Do you need a `.env` file?

**Short answer: no.** This project currently requires **zero environment
variables** to run, build, or deploy. There is no API key, database URL, or
secret anywhere in the codebase. All rendering is client-side, and the only
network integrations (@vercel/analytics) are zero-config.

This document still defines the `.env` convention so the project is ready the
moment a secret is needed (e.g. a CMS token, a form backend, an API key).

## 2. Next.js environment variable conventions

Next.js loads environment files in this precedence order (later overrides
earlier):

| File                    | Loaded in            | Committed to git? |
|-------------------------|----------------------|-------------------|
| `.env`                  | all                  | no (ignored)      |
| `.env.local`            | all (highest priority) | no (ignored)    |
| `.env.development`      | `next dev`           | yes (if needed)   |
| `.env.production`       | `next build`/`start` | yes (if needed)   |
| `.env.test`             | `next test`          | yes (if needed)   |

Rules that matter:

- **Client vs server exposure.** Only variables prefixed with `NEXT_PUBLIC_`
  are inlined into the browser bundle. Everything else stays server-side.
  This component library is 100% client components, so any variable a
  component reads must use the `NEXT_PUBLIC_` prefix.
- **Build-time inlining.** `NEXT_PUBLIC_*` values are baked in at **build
  time**, not runtime. Changing one requires a rebuild/redeploy.
- **Never commit secrets.** `.gitignore` ignores `.env*` (see §4). Real
  secrets live in the deployment platform's dashboard (Vercel → Project
  Settings → Environment Variables), never in the repo.

### 2.1 Recommended `.env.example` (to create when needed)

When the first secret arrives, create this template and commit it:

```bash
# ── Public (safe to expose in the browser bundle) ──────────────────
# NEXT_PUBLIC_SITE_URL=https://shader-svg.vercel.app

# ── Private (server-side only, never add NEXT_PUBLIC_ prefix) ──────
# EXAMPLE_API_KEY=replace-me

# ── Local overrides ─────────────────────────────────────────────────
# Copy this file to `.env.local` and fill in real values.
# `.env.local` is git-ignored and takes precedence over everything.
```

Then duplicate it as `.env.local` with real values for local development.

### 2.2 Where variables would be read

| Location                  | Example usage                                    |
|---------------------------|--------------------------------------------------|
| `app/layout.tsx` metadata | `process.env.NEXT_PUBLIC_SITE_URL` for canonical URLs / OG images |
| Any client component      | `process.env.NEXT_PUBLIC_*` only                 |
| `next.config.mjs`         | `process.env.*` (server-side, build time)        |

## 3. Configuration files reference

### 3.1 `next.config.mjs`

```js
// Strict build: lint and type errors fail `next build`
// (ignoreDuringBuilds / ignoreBuildErrors were removed 2026-09-27).
const nextConfig = {
  images: {
    unoptimized: true,          // next/image skips the Image Optimization API
  },
}
```

What this means in practice:

- **Builds never fail on lint/type errors.** Convenient for a v0-generated
  prototype, dangerous for production: type errors ship silently. The
  developer guide recommends removing both `ignore*` flags once the project
  matures.
- **`images.unoptimized: true`** disables Next.js image optimization. Every
  `<Image>` renders as a plain `<img>`. Correct for a static export or for
  hosts without an image optimizer; on Vercel you can drop it to get
  automatic optimization.

### 3.2 `tailwind.config.ts`

Tailwind CSS **v3** configuration (note: not v4 — there is no `@import
"tailwindcss"` and no `@tailwindcss/postcss` here).

- `darkMode: ['class']` — dark theme activates via a `.dark` class on
  `<html>`, which is exactly what `next-themes` toggles. Ready to wire up;
  see `docs/INTEGRATIONS.md`.
- `content` globs cover `./pages`, `./components`, `./app`, and root-level
  files — Tailwind scans all of them for class names.
- `theme.extend.colors` maps semantic tokens (`bg-background`,
  `text-foreground`, `border-border`, …) to the CSS variables defined in
  `app/globals.css`. This is the standard shadcn/ui token system.
- `borderRadius` derives `lg`/`md`/`sm` from `--radius` (0.5rem).
- `keyframes`/`animation` add `accordion-down` / `accordion-up` for Radix
  Accordion (shadcn component, not yet installed in `components/ui`).
- `plugins: [require('tailwindcss-animate')]` — the `tailwindcss-animate`
  plugin (v1.0.7).

### 3.3 `postcss.config.mjs`

```js
const config = {
  plugins: {
    tailwindcss: {},   // Tailwind v3 PostCSS plugin
    // autoprefixer is installed but NOT wired here —
    // add `autoprefixer: {}` if you need vendor prefixes beyond
    // what Tailwind emits by default.
  },
}
```

### 3.4 `tsconfig.json`

- `target: ES6`, `module: esnext`, `moduleResolution: bundler`,
  `jsx: preserve` (Next.js handles JSX transform), `strict: true`.
- `paths: { "@/*": ["./*"] }` — the `@/` import alias used everywhere
  (`@/components/mesh-gradient-svg`, `@/lib/utils`).
- `noEmit: true` — type-check only; Next.js emits the build.
- `include` covers all `.ts`/`.tsx` plus Next.js generated types.

### 3.5 `components.json` (shadcn/ui)

The shadcn/ui CLI manifest. Key settings:

| Key | Value | Meaning |
|-----|-------|---------|
| `style` | `default` | shadcn component style variant |
| `rsc` | `true` | React Server Components supported |
| `tailwind.css` | `app/globals.css` | where the CLI injects CSS variables |
| `tailwind.baseColor` | `neutral` | the gray palette |
| `tailwind.cssVariables` | `true` | theme via CSS vars, not hardcoded colors |
| `aliases.components` | `@/components` | install target for new components |
| `aliases.ui` | `@/components/ui` | install target for shadcn primitives |
| `iconLibrary` | `lucide` | icons come from `lucide-react` |

No `components/ui/*` files exist yet — the CLI has simply never been run
against this project. See `docs/DEVELOPER_GUIDE.md` for the add-component
workflow.

### 3.6 `app/globals.css` (+ `styles/globals.css` duplicate)

- `app/globals.css` is the **active** stylesheet, imported by
  `app/layout.tsx`. It defines the full shadcn token set (`--background`,
  `--foreground`, `--primary`, `--chart-1…5`, `--sidebar-*`, `--radius`) for
  both `:root` (light) and `.dark`, plus base rules (`border-border` on
  everything, `bg-background text-foreground` on body) and a
  `.text-balance` utility.
- `styles/globals.css` is a **byte-identical duplicate** that nothing
  imports (leftover from the v0 scaffold). Safe to delete; keeping it is
  harmless but confusing.

### 3.7 `.gitignore`

Ignores `node_modules`, `.next/`, `/out/`, `/build`, debug logs,
**`.env*`** (all env files), `.vercel`, and TypeScript build info
(`*.tsbuildinfo`, `next-env.d.ts`).

### 3.8 `pnpm-lock.yaml`

The project is pinned to **pnpm**. `package.json` has no
`packageManager` field, so contributors must know to run `pnpm install`,
not `npm install` — npm would resolve `latest` tags (e.g.
`@paper-design/shaders-react`, `framer-motion`) to different versions
than the lockfile.

## 4. Environment checklist for a new machine

```bash
git clone https://github.com/girishlade111/shader-svg.git
cd shader-svg
pnpm install        # respects pnpm-lock.yaml
pnpm dev            # http://localhost:3000 — no .env needed
```

If a `.env.local` ever becomes necessary, create it from
`.env.example` (see §2.1) and never commit it.
