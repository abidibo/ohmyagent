---
name: draft
description: Create, read, update, and delete interactive draft documents. Generates static HTML/JS/CSS output with dark theme, Highcharts, and Mermaid diagrams. Use when the user invokes /draft or asks to create/manage a draft.
---

# Draft Skill

## Overview

The Draft skill guides users through creating rich, structured draft documents for features, projects, or implementations. Each draft is stored in `.brain/{draft-name}/` at the project root and produces a polished static HTML output with navigation, charts, and diagrams.

## Commands

| Command | Description |
|---------|-------------|
| `/draft create {name}` | Start creating a new draft |
| `/draft read {name}` | Open the draft HTML in the browser |
| `/draft update {name}` | Resume incomplete work or edit/add steps |
| `/draft delete {name}` | Delete a draft and all its files (with confirmation) |

## File Reference

See the **STYLE.md — Output styling rules** section at the bottom of this file (dark theme, Tailwind, Highcharts, Mermaid)

## Brain Directory Structure

All draft data lives in `.brain/{draft-name}/` relative to the project root:

```
.brain/{draft-name}/
├── TOPIC.md          # Structured summary of the draft's purpose and goals
├── TOC.md            # Table of contents — ordered list of steps
├── progress.json     # Tracks which steps are defined, in-progress, or pending
├── steps/            # Individual step definition files
│   ├── 01-{slug}.md
│   ├── 02-{slug}.md
│   └── ...
└── output/
    └── index.html    # Generated HTML output
```

## Workflow Phases

### Phase 1: Topic Discovery

**Goal:** Understand what the user is trying to accomplish and produce a structured summary.

**Process:**

1. Ask the user to describe the feature/project/implementation they want to draft
2. Ask clarifying questions to understand:
   - What is the problem being solved?
   - What is the desired end state?
   - What files/areas of the current project are involved? (read them if needed)
   - What constraints or requirements exist?
   - Who is the audience for this draft?
3. Continue asking until you have a thorough understanding
4. Produce a structured summary and present it to the user for approval
5. Save the approved summary to `TOPIC.md`

**TOPIC.md format:**

```markdown
# {Draft Name}

## Problem Statement
{What problem is being solved}

## Goals
- {Goal 1}
- {Goal 2}

## Scope
{What is in scope and out of scope}

## Context
{Relevant files, current state, dependencies}

## Constraints
{Technical constraints, deadlines, requirements}

## Audience
{Who will read/use this draft}
```

### Phase 2: Step Structure (Table of Contents)

**Goal:** Create an ordered list of steps needed to fully describe the draft.

**Process:**

1. Based on the topic, propose an initial list of steps (title + one-line description each)
2. Discuss with the user — add, remove, reorder steps as needed
3. Each step should be a logical, self-contained unit of the overall plan
4. Steps should flow naturally from one to the next
5. Once the user approves the structure, save it to `TOC.md`

**TOC.md format:**

```markdown
# Table of Contents

## Steps

1. **{Step Title}** — {One-line description}
2. **{Step Title}** — {One-line description}
3. **{Step Title}** — {One-line description}
```

**Initialize progress.json** after TOC is approved:

```json
{
  "draft_name": "{name}",
  "created_at": "{ISO date}",
  "updated_at": "{ISO date}",
  "topic_complete": true,
  "toc_complete": true,
  "total_steps": 3,
  "current_step": 1,
  "steps": [
    {
      "number": 1,
      "title": "{Step Title}",
      "slug": "{step-slug}",
      "status": "pending",
      "file": "steps/01-{step-slug}.md"
    },
    {
      "number": 2,
      "title": "{Step Title}",
      "slug": "{step-slug}",
      "status": "pending",
      "file": "steps/02-{step-slug}.md"
    }
  ],
  "html_generated": false
}
```

**Step status values:** `"pending"` | `"in_progress"` | `"complete"`

### Phase 3: Step-by-Step Definition

**Goal:** Define each step in full detail through dialogue with the user.

**Process — repeat for each step:**

1. Announce which step you are working on (e.g., "Step 2 of 5: Database Schema Design")
2. Update `progress.json`: set the step status to `"in_progress"` and `current_step` to this step number
3. Ask the user to describe what this step involves
4. Ask targeted clarifying questions to understand the step deeply
5. **Propose at least 3 approaches/solutions** for the main decisions in this step, clearly marking one as **Recommended** with reasoning
6. Discuss trade-offs with the user until they choose an approach
7. After each major point is settled, ask: **"Do you want to add anything else to this step, or shall we move on?"**
8. Only move to the next step when the user explicitly says to proceed
9. Save the complete step definition to `steps/{NN}-{slug}.md`
10. Update `progress.json`: set step status to `"complete"`, increment `current_step`
11. **Regenerate the HTML output** so the user can see progress in real time

**Step file format** (`steps/{NN}-{slug}.md`):

```markdown
# Step {N}: {Title}

## Overview
{What this step covers and why it matters}

## Details

### {Subsection 1}
{Detailed description}

### {Subsection 2}
{Detailed description}

## Decisions Made
| Decision | Chosen Approach | Alternatives Considered |
|----------|----------------|------------------------|
| {What} | {Chosen} | {Alt 1}, {Alt 2} |

## Diagrams
{Mermaid diagram definitions if applicable — flowcharts, sequences, ER diagrams}

## Data / Metrics
{Highcharts data if applicable — comparisons, projections, benchmarks}

## Notes
{Additional notes, caveats, open questions}
```

### Phase 4: HTML Generation

**Goal:** Produce the final HTML output incorporating all steps.

**When to generate:**
- After each step is completed (incremental updates)
- After all steps are complete (final version)
- When `/draft update` modifies steps

**How to generate:**

1. Read `TOPIC.md` for the header and overview
2. Read `TOC.md` for navigation structure
3. Read each completed step file from `steps/`
4. Generate a single `output/index.html` following the rules in the STYLE.md section at the bottom of this file
5. The HTML must include:
   - Left sidebar navigation with all steps (completed steps marked with checkmark, pending steps dimmed)
   - Header section with draft title and topic summary
   - Each completed step rendered as a full section with proper formatting
   - Mermaid diagrams rendered from any mermaid code blocks in step files
   - Highcharts visualizations rendered from any chart data in step files
   - Responsive layout that works well on desktop
6. Update `progress.json`: set `html_generated` to `true` and `updated_at`

## Command Implementations

### `/draft create {name}`

1. Validate the name (lowercase, alphanumeric, hyphens only)
2. Check if `.brain/{name}/` already exists — if so, warn the user and ask if they want to overwrite or use `/draft update` instead
3. Create the directory structure: `mkdir -p .brain/{name}/steps .brain/{name}/output`
4. Execute **Phase 1** (Topic Discovery)
5. Execute **Phase 2** (Step Structure)
6. Execute **Phase 3** (Step-by-Step Definition) — process ALL steps
7. Final HTML is ready at `.brain/{name}/output/index.html`

### `/draft read {name}`

1. Check if `.brain/{name}/output/index.html` exists
2. If yes: run `xdg-open .brain/{name}/output/index.html`
3. If no: inform the user the draft has no HTML output yet and suggest `/draft create` or `/draft update`

### `/draft update {name}`

1. Check if `.brain/{name}/` exists — if not, suggest `/draft create`
2. Read `progress.json` to determine current state
3. **If steps are incomplete** (any step with status `"pending"` or `"in_progress"`):
   - Inform the user of progress: "You have completed X of Y steps. Resuming from Step Z."
   - Resume **Phase 3** from the first non-complete step
4. **If all steps are complete**, ask the user what they want to do:
   - Edit an existing step (show the list, let them pick)
   - Add new steps (append to TOC, define them)
   - Reorder steps
   - Regenerate HTML
5. After any changes, regenerate the HTML output
6. Update `progress.json` accordingly

### `/draft delete {name}`

1. Check if `.brain/{name}/` exists — if not, inform the user
2. Show the user what will be deleted (list files)
3. **Ask for explicit confirmation**: "Are you sure you want to delete the draft '{name}' and all its files? This cannot be undone."
4. Only proceed if the user confirms
5. Run `rm -rf .brain/{name}/`
6. Confirm deletion to the user

## HTML Generation Rules

### Content Transformation

When converting step markdown to HTML:

- **Mermaid blocks**: Look for fenced code blocks with language `mermaid` in step files. Render them as `<pre class="mermaid">` elements
- **Chart data**: Look for fenced code blocks with language `highcharts-json` in step files. Parse the JSON and render a `<div>` with a Highcharts initialization script
- **Code blocks**: Render as styled `<pre><code>` elements with the appropriate styling from STYLE.md
- **Tables**: Render as styled HTML tables following STYLE.md patterns
- **Info/Warning/Tip blocks**: Look for blockquotes prefixed with `> [!INFO]`, `> [!WARNING]`, `> [!TIP]`, `> [!RECOMMENDED]` and render them as styled callout boxes
- **Decisions tables**: Render with highlight on the chosen approach row
- **Headers**: Map to appropriate HTML heading levels with STYLE.md typography
- **Lists**: Render as styled unordered/ordered lists

### Navigation Generation

The sidebar must:
- List all steps with their number and title
- Show status indicators: checkmark icon for complete, circle for pending
- Highlight the currently viewed step
- Include smooth scroll anchors
- Show the draft title at the top
- Show a progress indicator (e.g., "3 of 5 steps complete")

### Incremental Generation

When generating HTML after a single step completion:
- The HTML must include ALL completed steps, not just the latest one
- Pending steps should appear in the nav as dimmed/disabled items
- The content area should only show completed steps

## Critical Rules

1. **Always ask before proceeding** — Never assume what the user wants. Ask questions until you understand.
2. **Three solutions minimum** — For every significant decision point, propose at least 3 approaches with a recommended one.
3. **Explicit stop signal** — Keep asking "anything else for this step?" until the user explicitly says to move on.
4. **Incremental HTML** — Regenerate HTML after EVERY step completion, not just at the end.
5. **Progress persistence** — Always keep `progress.json` current so work can be resumed across sessions.
6. **Read before writing** — When resuming, read all existing files before asking the user anything.
7. **Project context** — When the user mentions project files, read them to provide informed suggestions.
8. **No hallucination** — If you don't know something about the user's project, ask. Don't guess.
9. **Step isolation** — Each step file must be self-contained and understandable on its own.
10. **Validate names** — Draft names must be lowercase with hyphens only (e.g., `intelx-rework`, `auth-redesign`).


---

## STYLE.md — Output styling rules

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
