# Third-Party Integrations

Every external service and library this project depends on, what it does
here, how it is configured, and what it costs. Ordered by importance to
the app.

---

## 1. `@paper-design/shaders-react` — the mesh gradient shader

**What it is.** Paper Design's React wrapper around GPU fragment shaders.
Renders an animated WebGL mesh gradient on a `<canvas>`.

**How it is used** (`components/mesh-gradient-svg.tsx`):

```tsx
import { MeshGradient } from "@paper-design/shaders-react"

<MeshGradient colors={colors} className="w-full h-full" speed={1} />
```

| Prop        | Value used | Meaning |
|-------------|------------|---------|
| `colors`    | 5 hex stops (`#FFB3D9` → `#1A1A2E`) | pastel pink → sky blue → medium blue → dark blue-gray → near-black |
| `speed`     | `1`        | animation speed multiplier (1 = default) |
| `className` | `w-full h-full` | fills the `<div>` inside the SVG `<foreignObject>` |

The shader canvas is clipped to the ghost/arch silhouette by the SVG
`<clipPath id="shapeClip">` — the gradient itself knows nothing about the
shape; the clipping is pure SVG.

**Versioning risk.** `package.json` pins it as `"latest"` — every fresh
`pnpm install` without the lockfile can pull a new major. The lockfile
currently pins the resolved version; treat any `pnpm update` of this
package as a visual regression check (colors/speed API could change).

**Cost:** free, open-source (MIT).

---

## 2. `framer-motion` — animation engine

**What it is.** The standard React animation library (spring physics,
gestures, layout animations).

**How it is used** (`components/mesh-gradient-svg.tsx`):

1. **Floating ghost** — `<motion.div>` with an infinite keyframe loop:
   `y: [0, -8, 0]`, `scaleY: [1, 1.08, 1]`, `duration: 2.8`,
   `repeat: Infinity`, `ease: "easeInOut"`, `transformOrigin: "top center"`.
   The origin at the top makes the squash-and-stretch pivot like a hanging
   figure.
2. **Eye tracking** — two `<motion.ellipse>` elements animate `cx`/`cy`
   toward the cursor with a spring (`stiffness: 150, damping: 15`), giving
   the laggy, organic follow effect. Offset is clamped to ±8 px.

Also pinned as `"latest"` in `package.json` — same lockfile caveat as
above.

**Cost:** free, open-source (MIT).

---

## 3. `@vercel/analytics` — web analytics

**What it is.** Vercel's privacy-friendly analytics (page views, no
cookies, no fingerprinting).

**How it is used** (`app/layout.tsx`):

```tsx
import { Analytics } from '@vercel/analytics/next'
// …
<body>
  {children}
  <Analytics />
</body>
```

Zero configuration: the component injects the tracking script and
attributes hits to the Vercel project automatically. **No environment
variable required.** Data appears in the Vercel dashboard under the
Analytics tab once the project is deployed on Vercel.

**Cost:** free on Vercel Hobby (with event limits); paid tiers raise the
limits. Does nothing on non-Vercel hosts unless you wire a custom setup.

---

## 4. `next-themes` — dark/light theme management

**What it is.** Theme provider that toggles a `class` (`light`/`dark`) on
`<html>`, persists the choice in `localStorage`, and avoids hydration
flash.

**Status: installed and scaffolded, but NOT wired up.** The wrapper exists:

```tsx
// components/theme-provider.tsx
export function ThemeProvider({ children, ...props }: ThemeProviderProps) {
  return <NextThemesProvider {...props}>{children}</NextThemesProvider>
}
```

…but `app/layout.tsx` does not render it, and no theme toggle UI exists.
`tailwind.config.ts` (`darkMode: ['class']`) and `app/globals.css`
(`.dark` token set) are already prepared for it.

**To activate** (see also `docs/DEVELOPER_GUIDE.md`):

```tsx
// app/layout.tsx
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

`suppressHydrationWarning` on `<html>` is required because next-themes
mutates the class attribute after hydration.

**Cost:** free, open-source (MIT).

---

## 5. `geist` — font loading

**What it is.** Vercel's Geist typeface as self-hosted font packages
(`geist/font/sans`, `geist/font/mono`).

**How it is used** (`app/layout.tsx`):

```tsx
import { GeistSans } from 'geist/font/sans'
import { GeistMono } from 'geist/font/mono'
// …
html {
  font-family: ${GeistSans.style.fontFamily};
  --font-sans: ${GeistSans.variable};
  --font-mono: ${GeistMono.variable};
}
```

Fonts are bundled at build time — no Google Fonts network request, no
layout shift, works fully offline. The CSS variables also feed the
`--font-sans`/`--font-mono` tokens in `app/globals.css`.

**Cost:** free, open-source (OFL).

---

## 6. shadcn/ui + Radix UI + Tailwind CSS — UI foundation

**What it is.** shadcn/ui is not a package but a *pattern*: copy-pasteable
React components built on Radix UI primitives, styled with Tailwind.

**Status: foundation installed, no components added yet.**

- `components.json` — CLI manifest (style `default`, base color
  `neutral`, CSS variables, `@/components` aliases, `lucide` icons).
- `package.json` ships the full Radix suite (`accordion`, `dialog`,
  `dropdown-menu`, `toast`, `tooltip`, … 25 packages), `class-variance-authority`,
  `clsx` + `tailwind-merge` (via `lib/utils.ts` → `cn()`), and
  `tailwindcss-animate`.
- `components/ui/` **does not exist** — no shadcn component has been
  installed. Add them with:

```bash
pnpm dlx shadcn@latest add button
# installs components/ui/button.tsx using the components.json aliases
```

**Cost:** free, open-source (MIT for shadcn/ui and Radix).

---

## 7. `lucide-react` — icons

**What it is.** The Lucide icon set as tree-shakeable React components.

**Status: installed, currently unused.** Import on demand:

```tsx
import { Sparkles } from 'lucide-react'
<Sparkles className="h-4 w-4" />
```

**Cost:** free, open-source (ISC).

---

## 8. Supporting libraries (installed, unused in current UI)

These ship in `package.json` from the v0 scaffold but no component
imports them yet. Keep them only if the roadmap needs them; otherwise
they are dead weight in `node_modules` (they do **not** affect the
client bundle until imported).

| Package | What it is for |
|---------|----------------|
| `react-hook-form` + `@hookform/resolvers` + `zod` | forms + schema validation |
| `recharts` | charts |
| `react-day-picker` + `date-fns` | date picking / date utils |
| `embla-carousel-react` | carousels |
| `cmdk` | command palette (`⌘K`) |
| `sonner` | toast notifications |
| `vaul` | drawer component |
| `input-otp` | OTP inputs |
| `react-resizable-panels` | resizable layouts |
| `@emotion/is-prop-valid` | transitive-style helper (used by some shadcn patterns) |

---

## 9. Hosting & sync: Vercel + v0.app

| Integration | Role | Link |
|-------------|------|------|
| **Vercel** | hosting, CI/CD, analytics backend | deployment dashboard URL in `README.md` |
| **v0.app** | AI app builder; chats auto-push to this repo | project chat URL in `README.md` |

**How the sync works:** you iterate in the v0.app chat → deploy from the
v0 interface → v0 pushes the resulting code to this GitHub repository →
Vercel deploys the latest commit. Commits authored by v0 appear in git
history like any other commit. Because of this loop, **manual local edits
can be overwritten by the next v0 sync** — treat v0 as the source of
truth for UI changes, or disconnect the sync before doing sustained local
development (see `docs/DEVELOPER_GUIDE.md` → Troubleshooting).

**Cost:** Vercel Hobby free tier; v0.app has its own plan/credit model.

---

## 10. Integration health summary

| Integration | Configured | Used in code | Action needed |
|-------------|:----------:|:------------:|---------------|
| shaders-react | ✅ | ✅ | pin version instead of `latest` |
| framer-motion | ✅ | ✅ | pin version instead of `latest` |
| @vercel/analytics | ✅ | ✅ | none |
| next-themes | ✅ | ❌ | wire `ThemeProvider` into layout or remove |
| geist | ✅ | ✅ | none |
| shadcn/Radix/Tailwind | ✅ | ⚠️ (tokens only) | run `shadcn add` when UI grows |
| lucide-react | ✅ | ❌ | import when needed |
| Vercel + v0 sync | ✅ | n/a | decide source-of-truth workflow |
| form/chart/carousel libs | ✅ | ❌ | prune if roadmap doesn't need them |
