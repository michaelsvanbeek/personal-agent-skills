---
name: presentations
description: >-
  Presentation design, outlining, visual best practices, and brand compliance. Use
  when: building a presentation outline, choosing visuals for a slide deck, structuring
  a talk for a specific audience, iterating on presentation content in markdown before
  graduating to slides, reviewing an existing deck for flow and clarity, selecting
  the right chart or diagram type for a given message, or ensuring slides comply
  with an organization's brand guide.
tags:
  - manager
  - general
  - writer
  - communication
---

# Presentation Design

## When to Use

- Building a new presentation from scratch
- Structuring a talk for a specific audience (executives, engineers, mixed)
- Choosing the right visual (chart, diagram, table, screenshot) for each slide
- Reviewing an existing deck for narrative flow and clarity
- Iterating on presentation content in markdown before moving to slides
- Preparing speaker notes and delivery aids
- Ensuring a presentation follows the org's brand guide (colors, fonts, terminology, templates)

---

## Core Workflow: Markdown First, Slides Second

Presentations start as markdown outlines and graduate to external slide tools only after the content is solid.

### Phase 1 — Outline in Markdown

1. Define the audience, goal, and one-sentence takeaway.
2. Write a flat list of section headings (the "spine" of the talk).
3. Under each heading, write 2–5 bullet points — these are the ideas, not the slide text.
4. Review for flow: does each section lead logically to the next?
5. Identify which visuals each section needs (chart, diagram, table, screenshot, none).

### Phase 2 — Iterate and Tighten

1. Cut anything that doesn't serve the one-sentence takeaway.
2. Ensure every section answers "so what?" for the audience.
3. Mark slides that need data or assets you don't have yet.
4. Get feedback on the markdown outline before touching slides.

### Phase 3 — Graduate to Slides

1. Move content to the slide tool (Google Slides, PowerPoint, Keynote).
2. Record the external slide ID in the markdown frontmatter for traceability.
3. Keep the markdown outline as the source of truth for content — the slide deck is the visual layer.

### Markdown Presentation Document Format

Name files with date and topic: `YYYY-MM-DD-topic-slug.md`. Store in your preferred notes system. Presentation outlines are working documents; this skill holds only process guidance and reusable templates.

Frontmatter:

```yaml
---
title: "Presentation Title"
date: YYYY-MM-DD
audience: "Who is this for"
goal: "One sentence: what should the audience walk away with"
status: draft | review | final
external_slide_id: ""        # Google Slides ID, PowerPoint URL, etc.
external_slide_url: ""       # Direct link to the slide deck
---
```

---

## Audience Analysis

Before writing a single slide, answer these questions:

| Question | Why It Matters |
|----------|---------------|
| **Who is in the room?** | Determines depth, jargon tolerance, and framing |
| **What do they already know?** | Avoids over-explaining or under-explaining |
| **What do they care about?** | Drives which points to emphasize |
| **What do you want them to do after?** | Shapes the call to action |
| **How much time do you have?** | Constrains scope — cut ruthlessly |

### Audience Profiles

| Audience | Optimize For | Avoid |
|----------|-------------|-------|
| **Executives** | Impact, decisions, business outcomes | Implementation details, jargon |
| **Engineers** | Architecture, trade-offs, concrete examples | Hand-waving, marketing language |
| **Mixed / cross-functional** | Concepts first, depth on demand, analogies | Assuming shared vocabulary |
| **External / conference** | Narrative arc, memorable takeaways, polish | Inside references, acronyms |

---

## Presentation Flow

### The Narrative Spine

Every presentation needs a clear narrative arc. The audience should feel momentum, not a list of topics.

```
1. Hook        — Why should I care? (problem, question, surprising fact)
2. Context     — What do I need to know to follow along?
3. Core        — The main content (2–5 sections, each building on the last)
4. Synthesis   — Bring it together — what does this all mean?
5. Call to Action — What should the audience do next?
```

### Flow Rules

- **One idea per slide** — If you need two bullets, you might need two slides.
- **No orphan slides** — Every slide must connect to the one before and after it.
- **Signpost transitions** — "Now that we've covered X, let's look at Y" (in speaker notes, not on the slide).
- **Vary the rhythm** — Alternate between concept slides, example slides, and visual slides.
- **End sections with a summary** — Especially for talks > 15 minutes.

### Slide Count Guidelines

| Duration | Slides | Pace |
|----------|--------|------|
| 5 min (lightning) | 5–8 | ~1 slide/min |
| 15 min | 12–18 | ~1 slide/min |
| 30 min | 20–30 | ~1 slide/min |
| 45–60 min | 30–45 | ~1 slide/min, with pauses |

---

## Choosing the Right Visual

### Decision Matrix

| What You're Showing | Best Visual | Avoid |
|---------------------|------------|-------|
| **Trend over time** | Line chart | Pie chart, table |
| **Comparison across categories** | Bar chart (horizontal for many items) | Pie chart with > 5 slices |
| **Part of a whole** | Stacked bar or pie (≤ 5 segments) | Table of percentages |
| **Relationship / correlation** | Scatter plot | Bar chart |
| **Process or workflow** | Flowchart or swim lane diagram | Bullets describing steps |
| **Architecture / system** | Block diagram | Prose paragraph |
| **Before / after** | Side-by-side screenshots or metrics | Bullet list of changes |
| **Hierarchy / taxonomy** | Tree diagram or nested boxes | Indented bullet list |
| **Timeline / sequence** | Horizontal timeline | Table of dates |
| **Key metric** | Big number with context ("42% ↑ from Q1") | Chart with one data point |
| **Code or config** | Syntax-highlighted code block | Screenshot of an IDE |
| **Comparison of options** | Table with checkmarks / ✗ | Prose paragraphs |

### Visual Design Principles

- **Label everything** — Axes, legends, data points. Never make the audience guess.
- **Remove chartjunk** — No 3D effects, no gradient fills, no decorative gridlines.
- **Use color intentionally** — Highlight the point, dim the context. Max 3–4 colors.
- **Size for the room** — Minimum 24pt font for projected slides. If they squint, it's too small.
- **One takeaway per visual** — Title the chart with the insight, not the topic: "Latency dropped 60% after migration" not "Latency Over Time".

---

## Slide Content Rules

### Text

- **Max 6 words per bullet** — If it's a sentence, it's a paragraph, and paragraphs don't belong on slides.
- **Max 4 bullets per slide** — More than that and the audience reads instead of listens.
- **No full sentences on slides** — Sentences go in speaker notes.
- **Use the slide title as the takeaway** — "Agent costs dropped 40%" not "Cost Analysis".

### Speaker Notes

- Write full sentences in speaker notes as your talk track.
- Include transitions: how you get from this slide to the next.
- Note timing cues for longer talks: "[2 min on this slide]".
- Include backup data or answers to anticipated questions.

---

## Brand Guide

Every organization has visual and verbal standards. A **brand guide document** captures these rules so that every presentation is on-brand without re-checking the source material each time.

### What a Brand Guide Document Contains

| Section | What It Covers |
|---------|---------------|
| **Slide template references** | Links/IDs to the official slide template(s) — Google Slides, PowerPoint, Keynote |
| **Color palette** | Primary and secondary colors with hex, RGB, and usage guidance |
| **Typography** | Primary and secondary fonts, weights, and where each is used |
| **Logo usage** | Approved logo variants, clear space rules, co-branding guidance |
| **Gradients and imagery** | Approved gradient styles, photography guidelines, illustration rules |
| **Terminology and voice** | Preferred terms, product names, capitalization, and tone guidance |
| **Slide layout rules** | Required elements (confidentiality footer, product disclaimer), layout patterns |
| **Misuse rules** | What not to do — color misuse, logo distortion, font substitution |

### Brand Guide Document Format

Store brand guides in `templates/presentations/brand/`. One file per organization:

```
templates/presentations/brand/<org-slug>-brand-guide.md
```

Frontmatter:

```yaml
---
org: "Organization Name"
last_updated: YYYY-MM-DD
source_urls:
  - "URL to official brand guidelines"
  - "URL to official brand guidelines (additional)"
slide_template_ids:
  - id: "Google Slides / PowerPoint ID"
    name: "Template name"
    url: "Direct link"
---
```

### How to Create a Brand Guide

1. **Gather source material** — Official brand site, slide templates, style guides, design system docs.
2. **Extract rules** — Colors, fonts, logo rules, terminology, required slide elements.
3. **Document in markdown** — Use the format above. Include hex values, font names, and concrete rules.
4. **Reference slide templates** — Link to the official template(s) by ID and URL.
5. **Note misuse rules** — What the brand explicitly prohibits.
6. **Keep it current** — Update `last_updated` when the brand guidelines change.

### Using the Brand Guide in Presentations

When graduating a markdown outline to slides:

1. **Start from the official template** — Never start from a blank deck.
2. **Check colors** — Use only palette colors. Reference the brand guide for hex values.
3. **Check fonts** — Use the approved typefaces. No substitutions.
4. **Check required elements** — Confidentiality footer, disclaimers, logo placement.
5. **Check terminology** — Use the org's preferred terms and capitalization.
6. **Reference the brand guide** in the markdown frontmatter so reviewers know which guide applies.

### Markdown Presentation Frontmatter (Extended)

When a brand guide exists, reference it in presentation outlines:

```yaml
---
title: "Presentation Title"
date: YYYY-MM-DD
audience: "Who is this for"
goal: "One-sentence takeaway"
status: draft | review | final
brand_guide: "<org-slug>"           # references templates/presentations/brand/<org-slug>-brand-guide.md
external_slide_id: ""
external_slide_url: ""
---
```

---

## Specifying Charts and Visuals in Markdown

Every slide that needs a chart or other visual must include a precise **visual spec block** in the markdown source. The goal is to give enough information that a human, a slide tool, or a code generation model can produce the correct visual — on-brand — without ambiguity.

### The `Visual:` Line

Every slide starts its visual specification with a one-line `Visual:` descriptor:

```markdown
Visual: **[type]** — [key dimensions and data story]. [Layout hint if needed.]
```

Examples:
```markdown
Visual: **Scatter plot** — X: input cost ($/1M tokens, log scale), Y: intelligence index. Color by provider. Label each point.
Visual: **Horizontal bar chart** — top 5 cost reduction techniques, ordered by % savings. Single color fill.
Visual: **Layered block diagram** — stack from bottom to top: Foundation Model → Gateway → Tools → Skills → Agent.
Visual: **Side-by-side table** — 2 columns comparing Local MCP vs. Hosted MCP across 8 rows.
```

### The `<!-- visual-spec -->` Block

For charts that require data or will be generated programmatically, follow the `Visual:` line with a fenced spec block:

````markdown
Visual: **Scatter plot** — description.

```visual-spec
type: scatter
title: "Model Intelligence vs. Price · April 2026"
x_axis:
  label: "Input Cost ($/1M tokens)"
  scale: log
  range: [0.10, 20]
y_axis:
  label: "AA Intelligence Index"
  range: [30, 62]
color_by: provider
label_points: true
brand_colors:
  OpenAI: "#10a37f"
  Anthropic: "#d97c4f"
  Google: "#4285f4"
  DeepSeek: "#e74c3c"
background: "#F1F3F6"     # your org's neutral background color
annotation: "← Efficiency frontier (high intel, low cost)"
source: "artificialanalysis.ai/leaderboards/models · provider pricing pages"
data:
  - name: "GPT-5.4"
    provider: OpenAI
    x: 2.50
    y: 57
  - name: "Gemini 3.1 Pro Preview"
    provider: Google
    x: 2.00
    y: 57
```
````

### Visual Spec Field Reference

| Field | Required | Description |
|-------|----------|-------------|
| `type` | Yes | `scatter`, `bar`, `line`, `stacked_bar`, `pie`, `table`, `block_diagram`, `flowchart`, `timeline`, `heatmap` |
| `title` | Yes | The insight title — not the topic. "Latency dropped 60%" not "Latency Over Time". |
| `x_axis` / `y_axis` | Charts | Label, scale (`linear`/`log`), range, unit |
| `color_by` | When applicable | Field name or explicit color map |
| `brand_colors` | When color_by used | Map each category to a hex. Include brand-equivalent in comment |
| `background` | Optional | Defaults to `#FFFFFF` (Paper) or `#F1F3F6` (Business Card) for charts |
| `annotation` | Optional | Callout text with approximate position |
| `source` | Yes for data charts | Citation for data. One line. |
| `data` | For exact charts | Inline data rows. Use this when values are known and verified. |
| `data_note` | When data is approximate | Caveat for any unverified or back-calculated values |
| `orientation` | Bar charts | `horizontal` (default for many categories) or `vertical` |
| `sort` | Bar charts | `descending`, `ascending`, or `categorical` |

### Mapping Chart Colors to Your Brand

Before graduating to slides, replace placeholder colors with your organization's brand palette. Maintain a color map in your brand guide document with hex values for each category. Do not exceed 4–5 colors in a single chart.

### Chart Generation Prompt Pattern

For charts that will be built in Python (prototype, handoff, presentation export):

1. Include a `### Chart Generation Prompt` section after the data table.
2. The code block must reference `matplotlib` with brand-compatible colors.
3. Include cited sources as comments in the data block.
4. Add a `fig.text()` citation line at the bottom of the figure.
5. Save to a named file: `<slug>_<YYYY-MM-DD>.png`.

### Diagrams and Block Diagrams

For non-chart visuals (architecture diagrams, flow charts, process diagrams), use ASCII art in a fenced code block as a placeholder:

````markdown
Visual: **Layered block diagram** — agent stack, bottom to top.

```diagram
┌──────────────────────────────┐
│        Agent / Orchestrator   │  ← Plans, routes, manages state
├──────────────────────────────┤
│         Skills / Subgraphs    │  ← Reusable multi-step workflows
├──────────────────────────────┤
│            Tools / MCP        │  ← Single callable actions
├──────────────────────────────┤
│      Model Provider / Gateway │  ← Inference routing, cost control
├──────────────────────────────┤
│        Foundation Model       │  ← LLM (GPT-5.4, Claude, Gemini)
└──────────────────────────────┘
```

Graduation note: Recreate as a vertical box stack in Google Slides using your organization's brand
colors: boxes in Ink (#0F2E66) with Paper (#FFFFFF) text; arrows in Water Cooler (#1C98E8).
````

### Tables as Visuals

When a table is the primary visual for a slide:

- Include the markdown table inline.
- Add a `Table notes:` line specifying any highlighting (e.g., "bold the winning column") and brand alignment.
- Tables must use Archivo font in slides; column headers should use Ink (`#0F2E66`) background with Paper text.

### Graduation Notes

Every visual spec block should end with a brief `Graduation note:` that tells the slide builder what brand-specific treatments to apply:

```markdown
Graduation note: Apply your organization's brand colors to axes, labels, and chart background. Specify the typeface and size for chart text according to your brand guide.
```

---

## Anti-Patterns

| Anti-Pattern | Problem | Fix |
|-------------|---------|-----|
| **Wall of text** | Audience reads slides, tunes out speaker | Cut to 4 bullets max; move text to notes |
| **No narrative** | Feels like a list of topics, not a story | Add a hook, connect sections, end with synthesis |
| **Wrong visual** | Pie chart with 12 slices; table where a chart works | Use the decision matrix above |
| **Missing "so what"** | Data without interpretation | Title every visual with the insight |
| **Demo without context** | Audience doesn't know what to watch for | Set up each demo with "what you're about to see is..." |
| **No call to action** | Talk ends with a shrug | Always close with what the audience should do next |
| **Too many slides** | Rushing through at the end | Cut scope or request more time |
| **Reading slides aloud** | Insults the audience; adds no value | Slides are visual aids; speak to the content |

---

## Review Checklist

Before finalizing a presentation:

- [ ] **Audience**: Can I name who's in the room and what they care about?
- [ ] **Takeaway**: Can I state the one-sentence takeaway?
- [ ] **Flow**: Does each section connect to the next? Is there a hook and a close?
- [ ] **Visuals**: Is every visual the right type for what it shows?
- [ ] **Text**: No slide has more than 4 bullets or full sentences?
- [ ] **Timing**: Slide count matches available time (~1/min)?
- [ ] **Speaker notes**: Every slide has talk track and transitions?
- [ ] **Call to action**: The audience knows what to do next?
- [ ] **Brand compliance**: Colors, fonts, logo, terminology, and required elements match the brand guide?
