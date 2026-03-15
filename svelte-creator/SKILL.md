---
name: svelte-creator
description: Create a new SvelteKit + TailwindCSS frontend project or add new modules/pages to an existing one. Use this skill whenever the user wants to create a frontend, start a new web app, scaffold a SvelteKit project, or says things like "crear un frontend", "crea una app con svelte", "nuevo frontend", "quiero un frontend", "new svelte project", "start a web app", "crear una web", "nueva web". Also use this skill when the user wants to add a new page, module, or section to an existing SvelteKit project, such as "crea una página de usuarios", "agrega un módulo de products", "nueva sección", "add a new page", "create a dashboard page", or any variation involving adding pages or features to a SvelteKit app. Always use this skill even if the user doesn't specifically mention "SvelteKit" or "Svelte" — if they want a new frontend or a new page inside one, this is the way.
---

# svelte-creator

Scaffold a production-ready SvelteKit + TailwindCSS 4 frontend in seconds. The generated project comes preconfigured with TypeScript, adapter-node for deployment on any VPS (DigitalOcean, AWS EC2, GCP), api-proxy architecture, request interceptor for token/IP injection, and a clean separation between public and protected routes.

## Tech Stack

| Technology | Version | Purpose |
|---|---|---|
| Svelte | 5 (Runes) | UI framework |
| SvelteKit | 2.x | Full-stack framework |
| TailwindCSS | 4.x | CSS-first utility framework |
| TypeScript | 5.x | Type safety |
| adapter-node | latest | Node.js deployment adapter |
| dotenv + dotenv-cli | latest | Environment variables (Node 18 compat) |

---

## Part 1: Create a New Project

### Step 1: Pre-flight Checks

Before creating anything, verify the current workspace state:

**1a. Check if a SvelteKit project already exists**

Look for a `svelte.config.js` or `svelte.config.ts` in the current workspace root. If it exists, inform the user and ask if they want to proceed in a subdirectory.

```bash
ls svelte.config.* 2>/dev/null
```

If there's already a project, say: "It looks like there's already a SvelteKit project here. Want me to create the new frontend in a subfolder?"

**1b. Check Node.js version**

```bash
node -v
```

This project supports Node 18+ thanks to dotenv-cli.

### Step 2: Create the Project

Ask the user for a project name if they haven't provided one, then run:

```bash
npx -y sv create <project-name>
```

When the CLI prompts you, select:
- **Template**: SvelteKit minimal
- **TypeScript**: Yes (TypeScript)
- **Add-ons**: None (we'll install them manually)

Then enter the project directory:

```bash
cd <project-name>
```

### Step 3: Install Dependencies

Install the core dev dependencies:

```bash
npm install -D @tailwindcss/vite tailwindcss @sveltejs/adapter-node
```

Install runtime dependencies:

```bash
npm install dotenv dotenv-cli
```

### Step 4: Configure the Project

After installing dependencies, modify the following files:

**4a. `svelte.config.js` → Replace adapter-auto with adapter-node:**

```typescript
import adapter from '@sveltejs/adapter-node';
import { vitePreprocess } from '@sveltejs/vite-plugin-svelte';

/** @type {import('@sveltejs/kit').Config} */
const config = {
    kit: {
        adapter: adapter(),
        alias: {
            '$lib': 'src/lib'
        }
    }
};

export default config;
```

**4b. `vite.config.ts` → Add TailwindCSS plugin and dotenv:**

```typescript
import tailwindcss from '@tailwindcss/vite';
import { sveltekit } from '@sveltejs/kit/vite';
import { defineConfig } from 'vite';
import { config } from 'dotenv';

export default defineConfig({
    plugins: [tailwindcss(), sveltekit()],
    server: {
        allowedHosts: true
    }
});

config();
```

**4c. Update `package.json` scripts:**

Add the `start` script for production deployment:

```json
{
    "scripts": {
        "dev": "vite dev --port 3050",
        "build": "vite build",
        "preview": "vite preview --port 3020",
        "start": "dotenv -e .env -- node build"
    }
}
```

> The `start` script uses `dotenv-cli` to load `.env` and then runs the built Node.js server. This ensures compatibility with Node 18 servers where `--env-file` is not available.

**4d. Create `.env` and `.env.example`:**

```bash
# .env.example (and .env with same content)
API_BASE_URL=http://localhost:3001
NODE_ENV=development
```

**4e. Create `.gitignore`** — add these entries if not already present:

```
.env
.env.*
!.env.example
!.env.test
node_modules/
build/
.svelte-kit/
.DS_Store
```

### Step 5: Create the Project Structure

Create the full directory structure. Read `references/project-structure.md` for the complete tree.

The key folders to create are:

```bash
# Route groups
mkdir -p src/routes/\(site\)
mkdir -p src/routes/\(admin\)
mkdir -p src/routes/api

# Lib structure
mkdir -p src/lib/components/site
mkdir -p src/lib/components/admin
mkdir -p src/lib/components/utils
mkdir -p src/lib/services
mkdir -p src/lib/utils
mkdir -p src/lib/stores

# Static assets
mkdir -p static
```

### Step 6: Create Core Files

**6a. CSS files** — Read `references/css-architecture.md` for the complete patterns. Create two CSS files:

- `src/site.css` — for the `(site)` route group (public pages)
- `src/admin.css` — for the `(admin)` route group (protected pages)

Both files must:
1. Import Google Fonts at the top (ask the user which fonts they want)
2. Import TailwindCSS: `@import 'tailwindcss';`
3. Define color palette using `@theme { --color-<name>-<shade>: <hex>; }`
4. Define font families in `@theme` using `--font-<name>: "<Font Name>", sans-serif;`
5. Define custom utility classes using `@layer utilities { }`

**6b. Layouts:**

`src/routes/(site)/+layout.svelte`:
```svelte
<script lang="ts">
    import "../../site.css";

    interface Props {
        children?: import("svelte").Snippet;
    }

    let { children }: Props = $props();
</script>

<main>
    <div class="min-h-screen">
        {@render children?.()}
    </div>
</main>
```

`src/routes/(admin)/+layout.svelte`:
```svelte
<script lang="ts">
    import "../../admin.css";

    interface Props {
        children?: import("svelte").Snippet;
    }

    let { children }: Props = $props();
</script>

<main>
    <div class="min-h-screen">
        {@render children?.()}
    </div>
</main>
```

**6c. `src/hooks.server.ts`** — The request interceptor. Read `references/api-proxy-patterns.md` for the full template. This file:
- Extracts the client IP from `x-forwarded-for` or `getClientAddress()`
- Saves it to `event.locals.clientIp`
- Overrides `event.fetch` to inject `Authorization` (Bearer token), `X-Client-IP`, `X-Forwarded-For`, and `X-Real-IP` headers on every outgoing request
- Handles 401 responses by clearing auth cookies
- Optionally protects routes by checking authentication state

**6d. Default pages:**

Create `src/routes/(site)/+page.svelte` with a minimal welcome page.

### Step 7: Post-Setup Guidance

After the project is created, tell the user:

1. Configure `.env` with at minimum:
   - `API_BASE_URL` — the backend API URL
   - `NODE_ENV` — development or production

2. Start the dev server:
   ```bash
   npm run dev
   ```

3. Build for production:
   ```bash
   npm run build
   npm start
   ```

---

## Part 2: Add New Modules / Pages

When the user wants to add a new page or module to an existing SvelteKit project, follow these steps.

### Step 1: Determine the Route Group

Ask which route group the new page belongs to:
- `(site)` — public pages (marketing, blog, landing)
- `(admin)` — protected pages (dashboard, settings, users)

### Step 2: Create the Route Files

For a page at e.g. `src/routes/(admin)/users/`:

**`+page.ts`** — data loading, always calls the api-proxy (never the backend directly):

```typescript
import type { PageLoad } from "./$types";
import { redirect } from '@sveltejs/kit';

export const load: PageLoad = async ({ fetch, url }) => {
    const page = url.searchParams.get("page") ?? "1";

    const res = await fetch('/api/users/list', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ page })
    });

    if (res.status === 401) {
        throw redirect(302, '/login');
    }

    const json = await res.json();
    return { users: json.data };
};
```

**`+page.svelte`** — clean presentation, no business logic:

```svelte
<script lang="ts">
    import type { PageData } from "./$types";
    let { data }: { data: PageData } = $props();
</script>

<h1>Users</h1>
<!-- Use components from $lib/components/admin/users/ -->
```

### Step 3: Create the API Proxy

For every backend call, create an api-proxy endpoint in `src/routes/api/<module>/<action>/+server.ts`. Read `references/api-proxy-patterns.md` for the full template.

```typescript
import type { RequestHandler } from './$types';

export const POST: RequestHandler = async ({ fetch, request }) => {
    const body = await request.json();

    const response = await fetch(`${process.env.API_BASE_URL}/v1/users/list`, {
        method: 'GET',
        headers: { 'Content-Type': 'application/json' }
    });

    const data = await response.text();
    return new Response(data, {
        status: response.status,
        headers: { 'Content-Type': 'application/json' }
    });
};
```

> Because `hooks.server.ts` overrides `event.fetch`, every call made from `+server.ts` using `fetch()` automatically includes the Bearer token, client IP headers, and handles 401 responses. You don't need to manually add these headers.

### Step 4: Create Components

If the page needs reusable UI pieces, create components in the appropriate directory:

- `src/lib/components/site/<module>/` — for public pages
- `src/lib/components/admin/<module>/` — for protected pages
- `src/lib/components/utils/` — shared across both

### Step 5: Create Services (if needed)

If the `+page.ts` or `+server.ts` logic becomes complex, extract it into service files:

- `src/lib/services/<name>.ts` — API-related logic
- `src/lib/utils/<name>.ts` — utility functions

---

## Design Principles

Every page and component should feel **intentional and memorable**, not generic. Before writing any markup, commit to a clear aesthetic direction and execute it with precision.

### Design Thinking (Before Coding)

Before creating a page or component, answer these questions:

1. **Purpose**: What problem does this interface solve? Who uses it?
2. **Tone**: Pick a direction — brutally minimal, luxury/refined, retro-futuristic, editorial/magazine, organic/natural, playful, art deco/geometric, soft/pastel, industrial/utilitarian, brutalist/raw, maximalist. Commit fully to it.
3. **Differentiation**: What makes this page memorable? What's the one visual element someone will remember?
4. **Constraints**: Performance needs, accessibility, mobile-first requirements.

**If generating designs with Stitch MCP**: use the tone and differentiation answers to craft a specific, detailed prompt (e.g., "dashboard with brutalist aesthetic, bold sans-serif type, monochrome palette with a single orange accent, raw grid layout" instead of just "create a dashboard").

### Typography

Choose fonts with **character and personality**. Register them in `@theme`.

**Mandatory rules:**
- Import from Google Fonts at the top of the CSS file
- Register font families in `@theme` using `--font-<name>`
- Pair a **distinctive display font** with a **refined body font**

**Forbidden defaults** — never use these as the primary font choice:
- ❌ Arial, Helvetica, system-ui, sans-serif (browser defaults)
- ❌ Inter, Roboto (overused, generic AI aesthetic)
- ❌ Space Grotesk (becoming the new "AI cliché")

**Recommended approach** — explore characterful choices like:
- Display: Playfair Display, Syne, Clash Display, Cabinet Grotesk, Unbounded, DM Serif Display, Fraunces, Instrument Serif
- Body: DM Sans, Plus Jakarta Sans, Outfit, Satoshi, Manrope, Work Sans, Source Sans 3, Nunito Sans
- Monospace: JetBrains Mono, Fira Code, IBM Plex Mono

> **Key**: Vary fonts between projects. Never converge on the same font combination across different apps.

### Color & Palette

Build a **cohesive, intentional palette** via `@theme`. Avoid timid, evenly-distributed colors.

**Strategy:**
- Choose a **dominant color** (primary) and one or two **sharp accents**
- Generate the full shade scale (50–950) for each using [UIColors.app](https://uicolors.app)
- Register everything in `@theme` — never use raw hex in Tailwind classes

**Forbidden palettes:**
- ❌ Purple gradient on white background (classic AI cliché)
- ❌ Generic blue (#3B82F6) as the only color
- ❌ Flat gray without warmth or personality

**Recommended approach:**
- Warm neutrals (slate with brown tint, stone, warm gray)
- Unexpected accent colors (amber, teal, rose, emerald, cyan)
- Dark modes with depth (not just `bg-gray-900`, but layered dark surfaces using `bg-<name>-950`, `bg-<name>-900`, `bg-<name>-800`)

### Motion & Micro-interactions

Use animations to create delight and guide attention. All animations must be implemented through **CSS `@keyframes` in the CSS file** and **custom utility classes in `@layer utilities`**.

**Implementation approach:**
- Define `@keyframes` in the CSS file (outside `@layer`)
- Create utility classes that apply the animation via `@apply`
- Use Tailwind's built-in `transition`, `duration-*`, `ease-*` classes for simple hover/focus states

**High-impact moments to prioritize:**
1. **Page load reveals** — staggered fade-in using `animation-delay` on child elements
2. **Hover states that surprise** — scale, color shifts, shadow changes
3. **Scroll-triggered effects** — use Intersection Observer in `$effect()` to toggle classes
4. **Micro-feedback** — button press states, input focus glow, loading skeletons

> See `references/css-architecture.md` for concrete `@keyframes` and animation utility examples.

### Spatial Composition

Break free from predictable layouts while staying within Tailwind's grid and flex system:

- **Asymmetry**: unequal column widths (`grid-cols-5` with `col-span-2` + `col-span-3`)
- **Overlap**: `relative`/`absolute` positioning, negative margins (`-mt-8`, `-ml-4`)
- **Generous whitespace**: use `py-16`, `py-24`, `gap-8`, `gap-12` — let content breathe
- **Grid-breaking elements**: full-bleed sections using custom utility classes
- **Diagonal flow**: `rotate-*` on decorative background elements

### Backgrounds & Atmosphere

Create depth and atmosphere instead of flat solid colors. Implement everything through `@layer utilities` custom classes:

- **Gradient meshes**: multi-stop gradients using `bg-gradient-to-*` or custom gradient utility classes
- **Noise/grain overlays**: pseudo-elements (`::before`, `::after`) with SVG noise filter — defined as custom utility classes in the CSS file
- **Layered transparencies**: overlapping elements with `opacity-*` and `backdrop-blur-*`
- **Dramatic shadows**: `shadow-xl`, `shadow-2xl`, custom shadow utilities in `@theme`
- **Geometric patterns**: CSS grid patterns or SVG backgrounds applied via custom utilities

> See `references/css-architecture.md` for concrete background/atmosphere examples.

### Anti-Patterns (NEVER Do These)

| ❌ Don't | ✅ Do Instead |
|---|---|
| Use Inter/Roboto/Arial as primary font | Choose a distinctive, characterful font pair |
| Purple gradient on white background | Commit to a unique, intentional palette |
| Same visual style for every project | Vary aesthetics: dark vs light, serif vs sans, minimal vs rich |
| Plain `bg-white` for every section | Create atmosphere with gradients, textures, or tinted backgrounds |
| Uniform, predictable grid layouts | Introduce asymmetry, overlap, or generous whitespace |
| No animations at all | Add at least page-load reveals and hover micro-interactions |
| Generic placeholder content | Design for real-world content and proportions |

---

## Architecture Rules (CRITICAL)

These rules must be followed at all times when writing code for this project:

### Language Rules
1. **All code must be written in English** — variable names, function names, file names, folder names, class names, comments, JSDoc, type definitions, and any other code artifact must always be in English, regardless of the language used in the user's prompt.
2. **UI content can be in any language** — text displayed to end users (buttons, labels, messages, headings, placeholders, tooltips, error messages) can be in the language the user specifies (e.g., Spanish). The distinction is: *code is English, content is the user's language*.

### CSS Rules
1. **Never use `<style>` tags** in `.svelte` files — all CSS goes in `site.css` or `admin.css`
2. **Never use inline CSS** — no `style="..."` attributes
3. **Never use arbitrary Tailwind values** — no `text-[11px]`, `w-[360px]`, `bg-[#ff0000]`, etc.
4. **Only use official TailwindCSS 4 classes** or custom classes built with `@apply`
5. **Colors**: only use colors defined in `@theme` + `white` and `black`
6. **Custom classes**: create them with `@apply` inside `@layer utilities { }` in the CSS file

### Code Organization Rules
7. **`+page.svelte`**: presentation only — no fetch calls, no business logic
8. **`+page.ts`**: data loading — calls go to the api-proxy, never to the backend directly
9. **`+server.ts`**: api-proxy — calls the backend using `fetch()` (which includes the interceptor)
10. **Complex logic** → extract to `src/lib/services/` or `src/lib/utils/`
11. **Components** → keep `+page.svelte` files small; extract UI into `src/lib/components/`

### Route Rules
12. **`(site)`**: public routes with their own layout and `site.css`
13. **`(admin)`**: protected routes with their own layout and `admin.css`
14. **API proxy**: all backend calls go through `src/routes/api/<module>/<action>/+server.ts`

---

## References

For more details, read:
- `references/project-structure.md` — Complete project directory tree
- `references/css-architecture.md` — CSS patterns, @theme, @apply, color palettes, fonts
- `references/api-proxy-patterns.md` — hooks.server.ts, +server.ts, +page.ts patterns
