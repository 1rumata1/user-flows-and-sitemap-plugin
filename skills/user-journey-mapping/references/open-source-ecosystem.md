# Open Source Ecosystem

Tools, plugins, and resources that complement this skill. Since we build **static flat shapes** (not components or variants), these tools play different roles: some help with input processing, some improve output after generation, and some provide alternative MCP servers for teams with access constraints.

---

## Official Figma MCP Tools

### figma-use skill (MANDATORY)
Load this before every `use_figma` call. It prevents common Plugin API bugs:
- Colors are 0–1 range, not 0–255
- Fills/strokes are read-only arrays — clone, modify, reassign
- Font must be loaded before text: `await figma.loadFontAsync({family, style})`
- Always return created node IDs

### generate_diagram tool
Creates FigJam diagrams from Mermaid syntax. Fastest path to a flow diagram. Supports flowcharts, sequence diagrams, state diagrams. No styling control — output uses FigJam defaults.

---

## Pre-Generation: Input Processing Helpers

These help extract or structure source material *before* we render.

### Information Architecture Generator (community skill, MIT)
- Tools: `get_metadata`, `get_design_context`, `get_screenshot`, `use_figma`
- Generates sitemap + per-screen content hierarchy from an existing Figma file
- **Use case**: Run on a Figma file with app screens → get a structured page inventory → use that as input for flow extraction

### Figma-to-User-Stories-Generator (open source, GitHub)
- URL: `github.com/aledema90/Figma-to-User-Stories-Generator`
- Extracts screens, flows, and components from Figma → generates user stories
- **Use case**: If the user has existing Figma screens but no documentation, this can extract the structure we then render as a flow

### figma-mcp-go (open source, GitHub)
- URL: `github.com/vkhanhqui/figma-mcp-go`
- Unique feature: maps prototype reactions into interaction flow diagrams
- **Use case**: Reverse-engineer flows from existing Figma prototypes — the prototype connections define the actual flow

### DESIGN.md Generator (Figma plugin)
- URL: `figma.com/community/plugin/1612814320994608244`
- Extracts local Figma styles → generates DESIGN.md with design system specs
- **Use case**: Run on the team's Figma file to discover their actual color/typography tokens, then use those instead of our fallback values

### Emotional Quality Auditor (community skill, MIT)
- Audits designs for emotional quality: joy, satisfaction, resonance
- **Use case**: When building journey maps, run this on existing app screens to inform the emotion row with data rather than guessing

---

## Post-Generation: Enhancement Plugins

After our skill creates the static shapes, these plugins add things we can't build through the Plugin API.

### Autoflow (Figma plugin, free with limits)
- URL: `figma.com/community/plugin/733902567457592893`
- Draws flow arrows between selected frames with intelligent obstacle detection
- Supports hand-drawn or minimalist line styles, text annotations on paths
- **Why recommend this**: Our static connector lines don't move when nodes are rearranged. Autoflow creates smart connectors that re-route. Suggest this after generation: "I've created the flow nodes — you can run the Autoflow plugin to add connectors that stay attached when you rearrange things."
- Free up to 50 flows per file

### Simpleflow (Figma plugin)
- URL: `figma.com/community/plugin/751821593330638172`
- Adds native FigJam-style connectors inside Figma Design files
- Connectors automatically update when frames move
- **Why recommend this**: Same benefit as Autoflow — persistent connectors that survive layout changes

### UX Flow Logic (Figma plugin, open source)
- URL: `figma.com/community/plugin/1548695544263550149`
- Auto-numbers flow steps with one click
- Documents triggers, assumptions, and conditions for developer handoff
- Provides a sticker sheet of standard flowchart symbols
- **Why recommend this**: Our skill creates the structure and content. UX Flow Logic adds the handoff layer — numbering, annotations, developer context. Complementary, not overlapping.

### User Flows Magic (Figma plugin)
- URL: `figma.com/community/plugin/1466972599456956555`
- Creates user flows in FigJam from text input
- **Why recommend this**: Alternative to `generate_diagram` if Mermaid syntax produces unexpected results

---

## Alternative MCP Servers

For users who can't use the official Figma MCP (free accounts, rate limits, no paid seat).

### figma-console-mcp (Southleft, open source)
- URL: `github.com/southleft/figma-console-mcp`
- Setup: `npx figma-console-mcp@latest` + Desktop plugin
- Full read/write via plugin bridge — no REST API, no rate limits
- Works with free Figma accounts
- Explicitly documents "Build a user flow diagram for the checkout process" as a use case
- Cloud mode for web AI clients (Claude.ai, v0, Replit) via pairing code

### claude-talk-to-figma-mcp (open source)
- URL: `github.com/arinspunk/claude-talk-to-figma-mcp`
- Works with any Figma account including free
- Supports parallel agent execution

### figma-mcp-go (open source)
- URL: `github.com/vkhanhqui/figma-mcp-go`
- No REST API at all — bypasses all rate limits
- Also does prototype-to-flow reverse engineering (see above)

### Framelink MCP (open source)
- URL: `github.com/GLips/Figma-Context-MCP`
- Provides simplified, AI-optimized layout data from Figma
- Good for reading existing designs to extract flows — cleaner context than raw API

---

## Visual References (Not Programmatically Used)

These community Figma files are useful as **design references** — examples of what well-structured flows and maps look like. We don't instantiate their components (we build flat shapes), but they're worth suggesting if the user wants to see best-in-class examples before we generate.

- **UX Flow Kit 3.0** (1M+ downloads): Most comprehensive flow chart kit. Good reference for node sizing, connector styles, and layout spacing.
- **Omnichart**: Modular UX flow chart with separated parts. Good reference for annotation and feedback marker placement.
- **User Flow Kit**: Arrow styles and screen preview cards. Good reference for wireflow layouts.
- **Sitemap Generator**: Themeable sitemap components. Good reference for IA-layer visual hierarchy.

Suggest these when the user asks "what should a good flow look like?" or "can you show me examples?" — link them to the Figma Community files so they can see the visual standard we're aiming for.

---

## When to Recommend What

| User says | Suggest |
|---|---|
| "The connectors don't move when I rearrange" | Autoflow or Simpleflow plugin |
| "I need step numbers and dev handoff notes" | UX Flow Logic plugin |
| "I'm on a free Figma account" | figma-console-mcp or claude-talk-to-figma-mcp |
| "I'm hitting rate limits" | figma-mcp-go |
| "I have screens in Figma but no flow documentation" | IA Generator skill or Figma-to-User-Stories-Generator |
| "What should a good flow look like?" | Link to UX Flow Kit 3.0 or Omnichart on Figma Community |
| "I want to use our team's actual colors" | DESIGN.md Generator plugin to extract their tokens |
| "I have a prototype, can you map its flow?" | figma-mcp-go (prototype reaction mapping) |
| "What emotions do users feel at each step?" | Emotional Quality Auditor on existing screens |
