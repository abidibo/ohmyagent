# Visual Style Guide

This file defines the visual style for all HTML artifacts produced during the spec workflow (diagrams, flowcharts, data models, tables, etc.).

This should be a dark theme modern elegant cool frontend application.

---

## Format

Always produce a **single, self-contained static HTML file**. All CSS and JS must be loaded from CDN — no build steps, no separate files, no inline `<style>` blobs for layout (use Tailwind utility classes instead).

---

## CDN libraries

Always include these in `<head>` / end of `<body>` as needed:

```html
<!-- Tailwind CSS -->
<script src="https://cdn.tailwindcss.com"></script>

<!-- Mermaid (diagrams, flowcharts) -->
<script src="https://cdn.jsdelivr.net/npm/mermaid@11/dist/mermaid.min.js"></script>

<!-- Alpine.js (lightweight interactivity, collapsible sections, tabs) — add only when needed -->
<script defer src="https://cdn.jsdelivr.net/npm/alpinejs@3/dist/cdn.min.js"></script>
```

Add other libraries only when genuinely required by the content (e.g. a chart library for data visualizations). Never include libraries that are not used.

---

## Tailwind configuration

Extend the default config to register the design tokens used throughout:

```html
<script>
  tailwind.config = {
    darkMode: 'class',
    theme: {
      extend: {
        colors: {
          surface: {
            DEFAULT: '#0f1117',
            raised: '#161b27',
            overlay: '#1e2537',
            border: '#2a3348',
          },
          accent: {
            DEFAULT: '#6366f1',   // indigo-500
            hover:   '#818cf8',   // indigo-400
            muted:   '#312e81',   // indigo-900
          },
          text: {
            primary:   '#e2e8f0',
            secondary: '#94a3b8',
            muted:     '#475569',
          },
        },
        fontFamily: {
          sans: ['Inter', 'ui-sans-serif', 'system-ui', 'sans-serif'],
          mono: ['JetBrains Mono', 'ui-monospace', 'monospace'],
        },
      },
    },
  }
</script>

<!-- Inter + JetBrains Mono from Google Fonts -->
<link rel="preconnect" href="https://fonts.googleapis.com">
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
<link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&family=JetBrains+Mono:wght@400;500&display=swap" rel="stylesheet">
```

---

## Theme

**Always dark.** Apply `class="dark"` on `<html>`. Background is `surface` (`#0f1117`). Never render on a white or light background.

| Token | Hex | Usage |
|---|---|---|
| `surface` | `#0f1117` | Page background |
| `surface-raised` | `#161b27` | Cards, panels |
| `surface-overlay` | `#1e2537` | Popovers, code blocks, nested containers |
| `surface-border` | `#2a3348` | Borders, dividers |
| `accent` | `#6366f1` | Primary highlights, active states, links |
| `accent-hover` | `#818cf8` | Hover states on accent elements |
| `accent-muted` | `#312e81` | Subtle accent backgrounds (badges, tags) |
| `text-primary` | `#e2e8f0` | Body text, headings |
| `text-secondary` | `#94a3b8` | Labels, captions, secondary info |
| `text-muted` | `#475569` | Placeholders, disabled, decorative |

---

## Typography

- **Font:** Inter for all prose; JetBrains Mono for code, identifiers, file paths.
- **Base size:** `text-sm` (14px) for body; `text-xs` for captions and metadata.
- **Line height:** `leading-relaxed` for body paragraphs; `leading-snug` for headings.
- **Headings:** semibold (`font-semibold`), `text-text-primary`. Use `text-lg` → `text-base` → `text-sm` hierarchy. Do not use `text-xl` or larger inside content sections.
- **Code spans:** `font-mono text-accent bg-surface-overlay px-1 rounded`.

---

## Layout

```html
<body class="dark bg-surface text-text-primary font-sans antialiased min-h-screen">
  <div class="max-w-4xl mx-auto px-6 py-10 space-y-10">
    <!-- page title -->
    <header class="border-b border-surface-border pb-6">
      <h1 class="text-lg font-semibold text-text-primary tracking-tight">Feature Name</h1>
      <p class="mt-1 text-sm text-text-secondary">Short subtitle or description</p>
    </header>

    <!-- content sections -->
    <section class="space-y-4">
      <h2 class="text-sm font-semibold uppercase tracking-widest text-accent">Section Title</h2>
      <!-- content -->
    </section>
  </div>
</body>
```

- Max width: `max-w-4xl`, centered.
- Generous vertical rhythm: `space-y-10` between top-level sections.
- Section headings: small-caps style — `text-sm font-semibold uppercase tracking-widest text-accent`.

---

## Cards and panels

```html
<div class="bg-surface-raised border border-surface-border rounded-xl p-5 space-y-3">
  <h3 class="text-sm font-semibold text-text-primary">Card Title</h3>
  <p class="text-sm text-text-secondary leading-relaxed">Content goes here.</p>
</div>
```

- Rounded corners: `rounded-xl`.
- No drop shadows. Separation is achieved through background contrast and a subtle border.

---

## Tables

```html
<div class="overflow-x-auto rounded-xl border border-surface-border">
  <table class="w-full text-sm">
    <thead class="bg-surface-overlay text-text-secondary uppercase text-xs tracking-wider">
      <tr>
        <th class="px-4 py-3 text-left font-medium">Column</th>
      </tr>
    </thead>
    <tbody class="divide-y divide-surface-border">
      <tr class="hover:bg-surface-overlay transition-colors">
        <td class="px-4 py-3 text-text-primary">Value</td>
      </tr>
    </tbody>
  </table>
</div>
```

---

## Diagrams (Mermaid)

Wrap Mermaid diagrams in a centered panel. Configure Mermaid to match the dark theme:

```html
<script>
  mermaid.initialize({
    startOnLoad: true,
    theme: 'base',
    themeVariables: {
      darkMode: true,
      background:      '#161b27',
      primaryColor:    '#312e81',
      primaryBorderColor: '#6366f1',
      primaryTextColor:   '#e2e8f0',
      lineColor:       '#6366f1',
      secondaryColor:  '#1e2537',
      tertiaryColor:   '#1e2537',
      edgeLabelBackground: '#161b27',
      fontFamily: 'Inter, ui-sans-serif, sans-serif',
      fontSize: '13px',
    },
  });
</script>

<div class="bg-surface-raised border border-surface-border rounded-xl p-6 flex justify-center">
  <div class="mermaid text-sm">
    flowchart TD
      A[Start] --> B[Step]
  </div>
</div>
```

---

## Badges and tags

```html
<!-- Neutral -->
<span class="inline-flex items-center px-2 py-0.5 rounded-full text-xs font-medium bg-surface-overlay text-text-secondary border border-surface-border">
  Label
</span>

<!-- Accent -->
<span class="inline-flex items-center px-2 py-0.5 rounded-full text-xs font-medium bg-accent-muted text-accent border border-accent/30">
  Active
</span>
```

---

## Code blocks

```html
<pre class="bg-surface-overlay border border-surface-border rounded-lg p-4 overflow-x-auto text-xs font-mono text-text-primary leading-relaxed"><code>your code here</code></pre>
```

---

## Minimal full-page template

Use this as the starting skeleton for every generated HTML file:

```html
<!DOCTYPE html>
<html lang="en" class="dark">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>{{Title}}</title>
  <link rel="preconnect" href="https://fonts.googleapis.com">
  <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
  <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&family=JetBrains+Mono:wght@400;500&display=swap" rel="stylesheet">
  <script src="https://cdn.tailwindcss.com"></script>
  <script>
    tailwind.config = {
      darkMode: 'class',
      theme: {
        extend: {
          colors: {
            surface: { DEFAULT: '#0f1117', raised: '#161b27', overlay: '#1e2537', border: '#2a3348' },
            accent:  { DEFAULT: '#6366f1', hover: '#818cf8', muted: '#312e81' },
            text:    { primary: '#e2e8f0', secondary: '#94a3b8', muted: '#475569' },
          },
          fontFamily: {
            sans: ['Inter', 'ui-sans-serif', 'system-ui', 'sans-serif'],
            mono: ['JetBrains Mono', 'ui-monospace', 'monospace'],
          },
        },
      },
    }
  </script>
  <!-- Add Mermaid only when the page contains diagrams -->
  <!-- <script src="https://cdn.jsdelivr.net/npm/mermaid@11/dist/mermaid.min.js"></script> -->
  <!-- Add Alpine only when the page needs interactivity -->
  <!-- <script defer src="https://cdn.jsdelivr.net/npm/alpinejs@3/dist/cdn.min.js"></script> -->
</head>
<body class="dark bg-surface text-text-primary font-sans antialiased min-h-screen">
  <div class="max-w-4xl mx-auto px-6 py-10 space-y-10">

    <header class="border-b border-surface-border pb-6">
      <h1 class="text-lg font-semibold text-text-primary tracking-tight">{{Title}}</h1>
      <p class="mt-1 text-sm text-text-secondary">{{Subtitle}}</p>
    </header>

    <!-- Sections go here -->

  </div>
</body>
</html>
```

---

## Tone and aesthetics

- **Elegant and restrained** — no gradients, no glows, no animations unless they carry meaning.
- **High contrast text** — `text-primary` on `surface` always passes WCAG AA.
- **Whitespace is content** — generous padding, never cramped.
- **Accent sparingly** — indigo (`accent`) is used to draw the eye, not to decorate. Reserve it for headings, active states, and key highlights.
- **No decorative icons** unless they aid comprehension (e.g. a checkmark on a completed step).
