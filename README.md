# User Journey Mapping Plugin

## Overview
This Claude Code plugin automates the creation of UX flow diagrams directly in Figma. It supports four diagram types — user flows, customer journey maps, sitemaps, and screen flow wireframes — built from live URLs, codebases, PRDs, video walkthroughs, or verbal descriptions. All output uses the UUI Loveship light theme with self-contained color tokens and no external library dependencies.

## Diagram Types
The `/user-journey-mapping` skill produces four distinct deliverables:
- **User Flow** — step-by-step task diagrams with decisions, error paths, and branch logic. Rendered in Figma Design with normalized arrow connectors, skip-path bypasses, and L-shaped feedback loops.
- **Detailed Journey Map** — experience grid across phases with actions, touchpoints, thoughts, emotions, severity-rated pain points, user effort indicators, and actionable opportunities. Includes a dedicated screenshots row when live URL capture is enabled.
- **Sitemap / Information Architecture** — hierarchical page tree with 3-column vertical layout, page type annotations, and sub-section grouping. Supports sites with 150+ pages built section-by-section across multiple API calls.
- **Screen Flow (FigJam)** — wireflow with actual screenshots captured via Playwright, connected by labeled arrows, and annotated with sticky notes. Renders in FigJam with branching support and follow-up sections.

## Source Processing
The plugin extracts flow data from multiple input types:
- **Live URLs** — Playwright captures screenshots automatically while the user navigates. For sitemaps, `browser_evaluate` programmatically extracts navigation links for 10x faster discovery.
- **Codebases** — reads router configs, page components, and navigation calls to reconstruct the page inventory.
- **PRDs and specs** — parses step-by-step behavior descriptions, acceptance criteria, and state transitions.
- **Video recordings** — works from transcripts (Loom, YouTube captions, Otter.ai) to identify key screens and transitions.
- **Verbal descriptions** — guided interview format: "Walk me through the experience from the user's perspective."

## Authentication Handling
When screenshot capture hits a login wall, the plugin stops and asks the user to sign in rather than silently producing placeholder images. It opens a headed browser with a persistent session, brings the window to foreground, waits for user confirmation, then navigates back to the intended page and resumes capture. This ensures authenticated screens (checkout pages, account settings, admin panels) are included in the deliverable.

## Rendering Approach
All diagrams render through the Figma Plugin API via `use_figma` calls. Colors, typography, and spacing follow the Loveship palette applied as raw hex values — no Figma library binding required. The plugin uses proven batch patterns: user flows complete in 2 API calls, journey maps in 2 calls, and sitemaps in 3-4 calls depending on size. Each call includes the full helper function set (palette, fill/stroke setters, text creation, node builders) since execution contexts don't persist between calls.

## Battle-Tested Rules
The plugin includes 70 rules derived from real-world testing across Rozetka and Amazon, covering:
- Connector direction normalization for arrows that point left, right, or loop backward
- Branch stacking instead of chaining for decision nodes with multiple alternatives
- Vertical arrow gap prevention (the `cy+VGAP` vs `cy+VGAP-H` fix)
- Section-by-section rendering with error recovery for large sitemaps
- Mandatory screenshot question before any diagram build begins
- Post-login redirect handling when sites bounce to their homepage after authentication

## Important Caveat
The plugin generates static flat diagrams — no interactive prototypes, no hover states, no component variants. Output is designed for UX audits, stakeholder presentations, developer handoff, and design documentation. The extraction phase relies on visible navigation surfaces and may miss dynamically loaded content behind complex JavaScript interactions.
