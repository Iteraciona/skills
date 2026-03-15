# CSS Architecture Reference

This document defines the CSS patterns, rules, and conventions used in all SvelteKit projects created by this skill.

## Fundamental Rules

1. **No `<style>` tags** in any `.svelte` file — all CSS goes in dedicated `.css` files
2. **No inline styles** — no `style="..."` on any HTML element
3. **No arbitrary Tailwind values** — `text-[11px]`, `w-[360px]`, `bg-[#ff0000]` are forbidden
4. **Only official TailwindCSS 4 classes** or custom classes created with `@apply`
5. **Only defined colors** — use palette colors from `@theme` plus `white` and `black`
6. **No inventing classes** — every class must exist in TailwindCSS 4 or be defined in the CSS file

## File Organization

Each route group has its own CSS file:

| Route Group | CSS File | Imported In |
|---|---|---|
| `(site)` | `src/site.css` | `src/routes/(site)/+layout.svelte` |
| `(admin)` | `src/admin.css` | `src/routes/(admin)/+layout.svelte` |

## CSS File Structure

Every CSS file follows this structure in order:

```css
/* 1. Google Fonts imports */
@import url("https://fonts.googleapis.com/css2?family=Poppins:wght@400;500;600;700&display=swap");
@import url('https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&display=swap');

/* 2. TailwindCSS import */
@import 'tailwindcss';

/* 3. Optional: TailwindCSS plugins */
@plugin '@tailwindcss/typography';

/* 4. Theme: colors and fonts (TailwindCSS 4 CSS-first config) */
@theme {
    /* Font families */
    --font-poppins: "Poppins", sans-serif;
    --font-inter: "Inter", sans-serif;

    /* Color palette — define ALL project colors here */
    --color-primary-50: #f0f9ff;
    --color-primary-100: #e0f2fe;
    --color-primary-200: #bae6fd;
    --color-primary-300: #7dd3fc;
    --color-primary-400: #38bdf8;
    --color-primary-500: #0ea5e9;
    --color-primary-600: #0284c7;
    --color-primary-700: #0369a1;
    --color-primary-800: #075985;
    --color-primary-900: #0c4a6e;
    --color-primary-950: #082f49;
}

/* 5. Custom utility classes */
@layer utilities {
    .custom-input {
        @apply border border-primary-200 rounded-md h-8 px-2 w-full bg-white shadow-sm focus:outline-none focus:ring-primary-400 focus:border-primary-400;
    }

    .table-th {
        @apply px-1 font-semibold text-xs lg:text-sm my-4;
    }

    .table-td {
        @apply px-1 font-normal text-xs lg:text-sm my-4;
    }
}

/* 6. Complex styles (outside @layer for specificity) */
.prose-custom h1 {
    @apply text-3xl font-bold mt-8 mb-3;
}
```

## Color Palette Pattern

When the user chooses a color palette, generate the full shade scale (50-950) and register it in `@theme`. Use tools like [UIColors.app](https://uicolors.app) to generate shades from a base color.

```css
@theme {
    --color-brand-50: #fef8ec;
    --color-brand-100: #fbebca;
    --color-brand-200: #f7d690;
    --color-brand-300: #f2b748;
    --color-brand-400: #f0a42f;
    --color-brand-500: #e98317;  /* ← base color */
    --color-brand-600: #ce6111;
    --color-brand-700: #ab4312;
    --color-brand-800: #8b3515;
    --color-brand-900: #732b14;
    --color-brand-950: #421406;
}
```

Once defined, these colors become available as Tailwind classes:
- `text-brand-500`, `bg-brand-100`, `border-brand-300`, etc.

**Rule**: Only use colors defined in `@theme` + `white` + `black`. Never use raw hex values in Tailwind classes.

## Font Pattern

Import fonts from Google Fonts, then register them in `@theme`:

```css
@import url("https://fonts.googleapis.com/css2?family=Poppins:wght@400;500;600;700&display=swap");

@theme {
    --font-poppins: "Poppins", sans-serif;
}
```

Usage in Tailwind: `font-poppins`

## Creating Custom Utility Classes

When a combination of Tailwind classes is reused frequently, create a custom class:

```css
@layer utilities {
    .btn-primary {
        @apply px-4 py-2 bg-primary-500 text-white rounded-lg font-medium hover:bg-primary-600 transition duration-200;
    }

    .card {
        @apply bg-white rounded-xl shadow-sm border border-primary-100 p-4;
    }

    .section-title {
        @apply text-2xl font-bold text-primary-900 mb-4;
    }
}
```

These classes are then used directly in `.svelte` files like any Tailwind class:

```svelte
<button class="btn-primary">Save</button>
<div class="card">
    <h2 class="section-title">Dashboard</h2>
</div>
```

## Complex Styles (outside @layer)

For styles that need higher specificity (e.g., WYSIWYG editor output, third-party component overrides), place them outside `@layer`:

```css
/* Outside @layer for specificity */
.editor-content h2 {
    @apply text-3xl font-black text-primary-900 italic tracking-tighter mt-7 mb-2;
}

.editor-content p {
    @apply mb-4 leading-relaxed;
}
```

## Font Pairing Pattern

Pair a **display font** (headings) with a **body font** (text). Import both from Google Fonts and register in `@theme`:

```css
/* Display + Body font pairing */
@import url("https://fonts.googleapis.com/css2?family=DM+Serif+Display:ital@0;1&display=swap");
@import url("https://fonts.googleapis.com/css2?family=DM+Sans:wght@400;500;600;700&display=swap");

@import 'tailwindcss';

@theme {
    --font-display: "DM Serif Display", serif;
    --font-body: "DM Sans", sans-serif;
}
```

Usage: `font-display` for headings, `font-body` for body text. Create utility classes for common heading styles:

```css
@layer utilities {
    .heading-hero {
        @apply font-display text-5xl lg:text-7xl font-normal tracking-tight;
    }

    .heading-section {
        @apply font-display text-3xl lg:text-4xl font-normal;
    }

    .body-text {
        @apply font-body text-base leading-relaxed;
    }
}
```

> **Vary between projects**: don't always use DM Serif + DM Sans. Explore Syne + Outfit, Fraunces + Manrope, Instrument Serif + Plus Jakarta Sans, etc.

## Animation Utility Pattern

Define `@keyframes` **outside `@layer`** (for browser compatibility), then create utility classes that reference them:

```css
/* @keyframes — defined outside @layer */
@keyframes fade-in-up {
    from {
        opacity: 0;
        transform: translateY(1rem);
    }
    to {
        opacity: 1;
        transform: translateY(0);
    }
}

@keyframes fade-in {
    from { opacity: 0; }
    to { opacity: 1; }
}

@keyframes scale-in {
    from {
        opacity: 0;
        transform: scale(0.95);
    }
    to {
        opacity: 1;
        transform: scale(1);
    }
}

/* Animation utility classes */
@layer utilities {
    .animate-fade-in-up {
        @apply opacity-0;
        animation: fade-in-up 0.6s ease-out forwards;
    }

    .animate-fade-in {
        @apply opacity-0;
        animation: fade-in 0.5s ease-out forwards;
    }

    .animate-scale-in {
        @apply opacity-0;
        animation: scale-in 0.4s ease-out forwards;
    }

    /* Stagger delays for page-load reveals */
    .delay-100 { animation-delay: 100ms; }
    .delay-200 { animation-delay: 200ms; }
    .delay-300 { animation-delay: 300ms; }
    .delay-400 { animation-delay: 400ms; }
    .delay-500 { animation-delay: 500ms; }
}
```

Usage in Svelte (staggered page-load reveal):

```svelte
<h1 class="heading-hero animate-fade-in-up">Welcome</h1>
<p class="body-text animate-fade-in-up delay-200">Your description here</p>
<button class="btn-primary animate-scale-in delay-400">Get Started</button>
```

For **scroll-triggered animations**, use Intersection Observer inside `$effect()`:

```svelte
<script lang="ts">
    let sectionRef = $state<HTMLElement>();
    let isVisible = $state(false);

    $effect(() => {
        if (!sectionRef) return;
        const observer = new IntersectionObserver(
            ([entry]) => { isVisible = entry.isIntersecting; },
            { threshold: 0.2 }
        );
        observer.observe(sectionRef);
        return () => observer.disconnect();
    });
</script>

<section bind:this={sectionRef} class={isVisible ? 'animate-fade-in-up' : 'opacity-0'}>
    <!-- content -->
</section>
```

## Background & Atmosphere Pattern

Create depth with gradients, overlays, and textures. All implemented via custom utility classes:

```css
@layer utilities {
    /* Gradient background with multiple stops */
    .bg-gradient-hero {
        @apply bg-gradient-to-br from-primary-950 via-primary-900 to-primary-800;
    }

    /* Glassmorphism card */
    .glass-card {
        @apply bg-white/10 backdrop-blur-xl border border-white/20 rounded-2xl shadow-2xl;
    }

    /* Subtle section tint (avoids flat white) */
    .bg-section-warm {
        @apply bg-gradient-to-b from-primary-50/50 to-white;
    }

    .bg-section-cool {
        @apply bg-gradient-to-b from-primary-50 to-primary-100/30;
    }
}
```

For **grain/noise overlays**, use a pseudo-element with an SVG filter (defined outside `@layer` for pseudo-element support):

```css
/* Grain overlay — defined outside @layer */
.grain-overlay {
    position: relative;
}

.grain-overlay::after {
    content: "";
    position: absolute;
    inset: 0;
    opacity: 0.04;
    pointer-events: none;
    background-image: url("data:image/svg+xml,%3Csvg viewBox='0 0 256 256' xmlns='http://www.w3.org/2000/svg'%3E%3Cfilter id='noise'%3E%3CfeTurbulence type='fractalNoise' baseFrequency='0.9' numOctaves='4' stitchTiles='stitch'/%3E%3C/filter%3E%3Crect width='100%25' height='100%25' filter='url(%23noise)'/%3E%3C/svg%3E");
    background-repeat: repeat;
    border-radius: inherit;
}
```

Usage:

```svelte
<section class="bg-gradient-hero grain-overlay py-24">
    <div class="relative z-10">
        <h1 class="heading-hero text-white">Premium feel</h1>
    </div>
</section>
```

## Anti-Generic Checklist

Before finishing any page, check against these common "AI slop" patterns:

| Check | ❌ Generic | ✅ Intentional |
|---|---|---|
| Font | Inter, Roboto, Arial | DM Serif, Syne, Fraunces, etc. |
| Colors | `bg-blue-500`, purple gradient | Custom palette via `@theme` |
| Backgrounds | Flat `bg-white` everywhere | Tinted gradients, grain, glass |
| Layout | Centered cards, uniform grid | Asymmetry, overlap, whitespace |
| Animations | None, or only `transition` | Page reveals, hover states, scroll triggers |
| Shadows | `shadow` or `shadow-md` only | Dramatic `shadow-xl`, `shadow-2xl` |
| Dark mode | `bg-gray-900` flat | Layered surfaces: 950/900/800 |

