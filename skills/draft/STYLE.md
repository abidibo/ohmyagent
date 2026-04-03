# Draft Output Style Guide

## Design Philosophy

Dark theme, modern, sophisticated. Clean typography with generous whitespace. Professional yet visually striking.

## Technology Stack

All loaded via CDN — no build step required.

```html
<!-- Tailwind CSS -->
<script src="https://cdn.tailwindcss.com"></script>

<!-- Highcharts (via jsDelivr) -->
<script src="https://cdn.jsdelivr.net/npm/highcharts@11/highcharts.js"></script>
<script src="https://cdn.jsdelivr.net/npm/highcharts@11/modules/accessibility.js"></script>

<!-- Mermaid.js for flowcharts, sequence diagrams, UML -->
<script src="https://cdn.jsdelivr.net/npm/mermaid@11/dist/mermaid.min.js"></script>
```

## Tailwind Configuration

Configure Tailwind inline for dark theme:

```html
<script>
  tailwind.config = {
    darkMode: 'class',
    theme: {
      extend: {
        colors: {
          surface: {
            900: '#0a0a0f',
            800: '#12121a',
            700: '#1a1a25',
            600: '#242430',
          },
          accent: {
            DEFAULT: '#6366f1',
            light: '#818cf8',
            dim: '#4f46e5',
          },
          text: {
            primary: '#e2e8f0',
            secondary: '#94a3b8',
            muted: '#64748b',
          }
        },
        fontFamily: {
          sans: ['Inter', 'system-ui', '-apple-system', 'sans-serif'],
          mono: ['JetBrains Mono', 'Fira Code', 'monospace'],
        }
      }
    }
  }
</script>
```

## Color Palette

| Role | Color | Hex |
|------|-------|-----|
| Background (deepest) | surface-900 | `#0a0a0f` |
| Card / Section bg | surface-800 | `#12121a` |
| Hover / Elevated | surface-700 | `#1a1a25` |
| Border / Divider | surface-600 | `#242430` |
| Primary accent | accent | `#6366f1` (indigo) |
| Accent light | accent-light | `#818cf8` |
| Text primary | text-primary | `#e2e8f0` |
| Text secondary | text-secondary | `#94a3b8` |
| Text muted | text-muted | `#64748b` |
| Success | green | `#22c55e` |
| Warning | amber | `#f59e0b` |
| Error | red | `#ef4444` |
| Info | cyan | `#06b6d4` |

## Layout Rules

- Body: `bg-surface-900 text-text-primary` with `class="dark"`
- Max content width: `max-w-6xl mx-auto`
- Sections: `bg-surface-800 rounded-2xl border border-surface-600 p-8`
- Cards: `bg-surface-700 rounded-xl border border-surface-600 p-6`
- Section spacing: `space-y-8` between major sections
- Inner spacing: `space-y-4` within sections

## Typography

- H1 (draft title): `text-4xl font-bold bg-gradient-to-r from-accent-light to-cyan-400 bg-clip-text text-transparent`
- H2 (step title): `text-2xl font-semibold text-text-primary`
- H3 (subsection): `text-lg font-medium text-accent-light`
- Body: `text-base text-text-secondary leading-relaxed`
- Code inline: `font-mono text-sm bg-surface-700 text-accent-light px-2 py-0.5 rounded`
- Code blocks: `bg-surface-900 border border-surface-600 rounded-xl p-4 font-mono text-sm text-text-secondary overflow-x-auto`

## Component Patterns

### Navigation Sidebar

Fixed left sidebar with step links. Active step highlighted with accent color and left border.

```html
<nav class="fixed left-0 top-0 h-full w-64 bg-surface-800 border-r border-surface-600 p-6 overflow-y-auto">
  <div class="text-xl font-bold text-accent-light mb-8">Draft Title</div>
  <ul class="space-y-2">
    <li>
      <a href="#step-1" class="block px-4 py-2 rounded-lg text-text-secondary hover:bg-surface-700 hover:text-text-primary transition-colors">
        Step 1: Title
      </a>
    </li>
    <!-- Active state -->
    <li>
      <a href="#step-2" class="block px-4 py-2 rounded-lg bg-accent/10 text-accent-light border-l-2 border-accent">
        Step 2: Title
      </a>
    </li>
  </ul>
</nav>
```

### Step Section

```html
<section id="step-1" class="bg-surface-800 rounded-2xl border border-surface-600 p-8">
  <div class="flex items-center gap-3 mb-6">
    <span class="flex items-center justify-center w-8 h-8 rounded-full bg-accent/20 text-accent-light text-sm font-bold">1</span>
    <h2 class="text-2xl font-semibold text-text-primary">Step Title</h2>
  </div>
  <div class="space-y-4 text-text-secondary leading-relaxed">
    <!-- Step content -->
  </div>
</section>
```

### Info / Warning / Tip Boxes

```html
<!-- Info -->
<div class="bg-cyan-500/10 border border-cyan-500/20 rounded-xl p-4 text-cyan-300">
  <strong>Info:</strong> Content here
</div>
<!-- Warning -->
<div class="bg-amber-500/10 border border-amber-500/20 rounded-xl p-4 text-amber-300">
  <strong>Warning:</strong> Content here
</div>
<!-- Tip / Recommendation -->
<div class="bg-accent/10 border border-accent/20 rounded-xl p-4 text-accent-light">
  <strong>Recommended:</strong> Content here
</div>
```

### Tables

```html
<div class="overflow-x-auto">
  <table class="w-full text-sm">
    <thead>
      <tr class="border-b border-surface-600">
        <th class="text-left py-3 px-4 text-text-muted font-medium uppercase tracking-wider text-xs">Header</th>
      </tr>
    </thead>
    <tbody class="divide-y divide-surface-600">
      <tr class="hover:bg-surface-700/50 transition-colors">
        <td class="py-3 px-4 text-text-secondary">Cell</td>
      </tr>
    </tbody>
  </table>
</div>
```

### Badges / Tags

```html
<span class="inline-flex items-center px-2.5 py-0.5 rounded-full text-xs font-medium bg-accent/20 text-accent-light">Tag</span>
<span class="inline-flex items-center px-2.5 py-0.5 rounded-full text-xs font-medium bg-green-500/20 text-green-400">Complete</span>
<span class="inline-flex items-center px-2.5 py-0.5 rounded-full text-xs font-medium bg-amber-500/20 text-amber-400">In Progress</span>
```

## Highcharts Theme

Apply dark theme to all Highcharts instances:

```javascript
Highcharts.setOptions({
  chart: {
    backgroundColor: '#12121a',
    style: { fontFamily: 'Inter, system-ui, sans-serif' }
  },
  title: { style: { color: '#e2e8f0' } },
  subtitle: { style: { color: '#94a3b8' } },
  xAxis: {
    gridLineColor: '#242430',
    lineColor: '#242430',
    tickColor: '#242430',
    labels: { style: { color: '#94a3b8' } },
    title: { style: { color: '#94a3b8' } }
  },
  yAxis: {
    gridLineColor: '#242430',
    lineColor: '#242430',
    tickColor: '#242430',
    labels: { style: { color: '#94a3b8' } },
    title: { style: { color: '#94a3b8' } }
  },
  legend: {
    itemStyle: { color: '#94a3b8' },
    itemHoverStyle: { color: '#e2e8f0' }
  },
  tooltip: {
    backgroundColor: '#1a1a25',
    borderColor: '#242430',
    style: { color: '#e2e8f0' }
  },
  colors: ['#6366f1', '#06b6d4', '#22c55e', '#f59e0b', '#ef4444', '#818cf8', '#a78bfa', '#34d399']
});
```

## Mermaid Theme

Initialize Mermaid with dark theme:

```javascript
mermaid.initialize({
  startOnLoad: true,
  theme: 'dark',
  themeVariables: {
    darkMode: true,
    primaryColor: '#6366f1',
    primaryTextColor: '#e2e8f0',
    primaryBorderColor: '#4f46e5',
    lineColor: '#94a3b8',
    secondaryColor: '#1a1a25',
    tertiaryColor: '#12121a',
    background: '#12121a',
    mainBkg: '#12121a',
    nodeBorder: '#4f46e5',
    clusterBkg: '#1a1a25',
    titleColor: '#e2e8f0',
    edgeLabelBackground: '#12121a'
  },
  flowchart: { curve: 'basis', padding: 20 },
  sequence: { actorMargin: 50, boxMargin: 10 }
});
```

## HTML Boilerplate

Every draft output must follow this structure:

```html
<!DOCTYPE html>
<html lang="en" class="dark">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Draft: {DRAFT_NAME}</title>
  <link rel="preconnect" href="https://fonts.googleapis.com">
  <link href="https://fonts.googleapis.com/css2?family=Inter:wght@300;400;500;600;700&family=JetBrains+Mono:wght@400;500&display=swap" rel="stylesheet">
  <script src="https://cdn.tailwindcss.com"></script>
  <script src="https://cdn.jsdelivr.net/npm/highcharts@11/highcharts.js"></script>
  <script src="https://cdn.jsdelivr.net/npm/highcharts@11/modules/accessibility.js"></script>
  <script src="https://cdn.jsdelivr.net/npm/mermaid@11/dist/mermaid.min.js"></script>
  <script>/* Tailwind config from above */</script>
  <style>
    /* Scrollbar styling */
    ::-webkit-scrollbar { width: 6px; }
    ::-webkit-scrollbar-track { background: #0a0a0f; }
    ::-webkit-scrollbar-thumb { background: #242430; border-radius: 3px; }
    ::-webkit-scrollbar-thumb:hover { background: #6366f1; }
    /* Smooth scrolling */
    html { scroll-behavior: smooth; }
  </style>
</head>
<body class="bg-surface-900 text-text-primary font-sans min-h-screen">
  <!-- Sidebar navigation -->
  <nav class="fixed left-0 top-0 h-full w-64 bg-surface-800 border-r border-surface-600 p-6 overflow-y-auto z-10">
    <!-- Nav content -->
  </nav>

  <!-- Main content -->
  <main class="ml-64 p-8 max-w-5xl">
    <!-- Header -->
    <header class="mb-12">
      <h1 class="text-4xl font-bold bg-gradient-to-r from-accent-light to-cyan-400 bg-clip-text text-transparent mb-4">
        {DRAFT_TITLE}
      </h1>
      <p class="text-text-secondary text-lg">{TOPIC_SUMMARY}</p>
      <div class="flex gap-3 mt-4">
        <span class="badge">tag1</span>
        <span class="badge">tag2</span>
      </div>
    </header>

    <!-- Steps -->
    <div class="space-y-8">
      <!-- Step sections go here -->
    </div>
  </main>

  <script>/* Highcharts theme + Mermaid init from above */</script>
</body>
</html>
```

## Content Rendering Guidelines

- Use Mermaid for: flowcharts, sequence diagrams, class diagrams, state diagrams, ER diagrams, Gantt charts
- Use Highcharts for: bar charts, line charts, pie charts, area charts, any data visualization
- Use code blocks for: API examples, configuration snippets, implementation details
- Use tables for: comparisons, field definitions, API endpoint lists
- Use info/warning boxes for: recommendations, caveats, important notes
- Use badges for: status indicators, technology tags, priority levels
