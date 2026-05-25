# User Journey Mapping Plugin

## Overview
This Claude Code plugin automates the creation of UX flow diagrams directly in Figma. It supports four diagram types — user flows, customer journey maps, sitemaps, and screen flow wireframes — built from live URLs, codebases, PRDs, video walkthroughs, or verbal descriptions. All output uses the UUI Loveship light theme with self-contained color tokens and no external library dependencies.

## Diagram Types
The `/user-journey-mapping` skill produces four distinct deliverables:
- **User Flow** — step-by-step task diagrams with decisions, error paths, and branch logic. Rendered in Figma Design with normalized arrow connectors, skip-path bypasses, and L-shaped feedback loops.
- **Detailed Journey Map** — experience grid across phases with actions, touchpoints, thoughts, emotions, severity-rated pain points, user effort indicators, and actionable opportunities. Includes a dedicated screenshots row when live URL capture is enabled.
- **Sitemap** — hierarchical page tree with 3-column vertical layout, page type annotations, and sub-section grouping. Supports sites with 150+ pages built section-by-section across multiple API calls.
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
