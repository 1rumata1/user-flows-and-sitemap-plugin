---
name: user-journey-mapping
description: Build user flow diagrams, customer journey maps, sitemaps, and screen flow diagrams in Figma Design and FigJam using the Figma MCP server and design system tokens. Trigger on "user flow", "journey map", "flow diagram", "task flow", "UX flow", "wireflow", "user path", "customer journey", "onboarding flow", "checkout flow", "signup flow", "sitemap", "information architecture", "page hierarchy", "site structure", "navigation map", "screen flow", "FigJam flow", "quick flow", or when the user wants to reverse-engineer flows from PRDs, specs, video walkthroughs, live app URLs, screenshots, FigJam boards, codebases, or verbal descriptions. Even "map out the flow", "diagram how users navigate this", or "show me the page structure" should trigger it. Options 1–3 render in Figma Design with design system tokens. Option 4 renders in FigJam via Mermaid.
---

# Figma UX Flow Builder

User flow diagrams, journey maps, sitemaps, and screen flow diagrams in Figma Design and FigJam. Options 1–3 are built with design system library components, text styles, and color variables in Figma Design. Option 4 uses Mermaid flowcharts in FigJam for quick conceptual flows.

Output is **flat static diagrams** — no interactive prototypes, no hover states. But every color, font, and UI element uses real design system library bindings so the output respects theme changes and design system updates.

## On Activation

Read silently on activation (initial load):
- `references/design-tokens.md` — design system color variables, text styles, spacing, and binding patterns (THE source of truth for all styling)
- `references/source-processing.md` — Extracting flows from each source type
- `references/open-source-ecosystem.md` — Plugins and alternative MCP servers

**IMPORTANT: Read the diagram-specific reference file AGAIN right before Step 5 (Render).** Do NOT rely on the activation read — by render time, the details (exact node sizes, colors, layout rules) will be forgotten. The mandatory re-read is specified in Step 5 below and in Rule #49. The diagram-specific references are:
- User Flow (Option 1) → `references/flow-patterns.md`
- Journey Map (Option 2) → `references/journey-map-structure.md`
- Sitemap (Option 3) → `references/sitemap-structure.md`
- Screen Flow (Option 4) → `references/screen-flow-figjam.md`

---

## Prerequisites

Requires:
1. **Figma MCP server** (remote) — `use_figma`, `generate_diagram`, `create_new_file`, `search_design_system`
2. **Design System Components library** connected in Figma (for Tag, Divider, Button instances)
3. **figma-use** skill loaded before every `use_figma` call

Setup: `claude plugin install figma@claude-plugins-official`, then authenticate.

---

## Workflow

### 1. Ask: What type of diagram?

**Always ask this first.**

> "What type of diagram do you need?
>
> 1. **User Flow** — step-by-step diagram of a specific task with screens, decisions, and error paths. Shows the exact sequence a user follows. Best for devs, PMs, and designers. *Renders in Figma Design.*
>
> 2. **Detailed Journey Map** — experience across phases with actions, touchpoints, thoughts, emotions (sentiment curve), pain points (severity-rated), user effort indicators, and opportunities. Includes key screen thumbnails per phase. Best for UX audits, stakeholder presentations, and redesign planning. *Renders in Figma Design.*
>
> 3. **Sitemap / Information Architecture** — hierarchical tree of all pages and sections in a product. Shows navigation structure, page groupings, and depth levels. Best for IA audits, redesigns, and developer handoff. *Renders in Figma Design.*
>
> 4. **Screen Flow (FigJam)** — wireflow with actual screenshots placed on a FigJam canvas, connected by arrows, and annotated with sticky notes. Screenshots are captured via Playwright or provided as files. Best for visual walkthroughs, stakeholder demos, and team alignment. *Renders in FigJam.*"

**User Flow** → screens, steps, decisions, errors. Linear or branching paths. For devs/PMs/designers.
**Detailed Journey Map** → phases, expanded numbered actions, touchpoints, thoughts, emotions with sentiment curve, severity-rated pain points (CRITICAL/MAJOR/MINOR), user effort indicators (Low/Medium/Very High), key screen thumbnails per phase, actionable opportunities, summary stats bar. For stakeholders/CX/UX audits.
**Sitemap / IA** → hierarchical tree showing all pages, sections, and navigation groups. Top-down layout. Color-coded by section. Annotated with page types (landing, form, dashboard, settings). For IA audits, navigation redesigns, and developer handoff.
**Screen Flow (FigJam)** → wireflow with actual screenshots connected by arrows in FigJam. Sticky notes for annotations. For visual walkthroughs, stakeholder demos, and team alignment.

### 2. Ask: Include screenshots? (MANDATORY — NEVER SKIP — Rule #64)

**This step is MANDATORY for ALL diagram types. NEVER skip it. NEVER assume "no screenshots".**

Even if the user provided the diagram type upfront (e.g., "build a journey map for Amazon"), you MUST still ask this question before proceeding. Skipping it results in journey maps without screenshots, which is the #1 quality issue.

**If the user chose Option 4 (Screen Flow), screenshots are REQUIRED — not optional.** Screen Flow wireflows are built from actual screenshots. Ask for a live URL or screenshot files. If the user has neither, suggest Option 1 (User Flow) for an abstract diagram instead.

**For all options, always ask this immediately after Step 1.**

> "Should I include screenshots alongside the flow?
>
> - **Yes — I have a live URL** → I'll open a browser, you navigate through the flow, and I auto-capture each screen
> - **Yes — I have a video/recording** → Provide a Loom/YouTube link or transcript, and I'll extract key frames
> - **Yes — I have screenshot files** → Share the image files or paths
> - **No** — Build the diagram without screenshots"

#### Screenshot capture methods:

**Live URL (Playwright):**
1. Install Playwright if needed: `npm install playwright` + `npx playwright install chromium`
2. Launch a **headed** Chrome browser navigating to the user's URL: `chromium.launch({ headless: false, channel: 'chrome' })`
3. Use `osascript -e 'tell application "Google Chrome" to activate'` to bring the window to the foreground (macOS)
4. Auto-capture screenshots every time the URL changes or every 12–15 seconds
5. Save to `/tmp/app-screenshots/` with sequential numbering
6. Tell the user the browser is open and to navigate through the flow at their own pace
7. When the user says "done", stop capture and review all screenshots
8. Deduplicate — keep only screenshots where the page visually changed (different file sizes = different content)
9. Upload unique screenshots to Figma via `upload_assets` → `curl POST` with raw bytes

**Video/Recording:**
1. Ask user for a transcript (Loom auto-generates, YouTube has captions, Otter.ai)
2. If no transcript, walk through step by step: "What's the first screen? What happens next?"
3. If the user provides screenshot timestamps, extract frames at those points
4. If the user shares the video file, take screenshots at key transition points

**Screenshot files:**
1. Read image files from provided paths
2. Upload to Figma via `upload_assets`

**Important Playwright notes:**
- If the browser window isn't visible, use `osascript` to bring it to the foreground (macOS)
- If the process dies, relaunch — use a persistent profile (`launchPersistentContext`) so the user stays logged in
- Clean up duplicate periodic captures — keep only distinct screens (compare file sizes)

**Auth-required screens — NEVER silently skip (Rule #65):**
When you encounter a login/auth wall during screenshot capture (e.g., a sign-in page redirect, a login modal, a 401/403 response), you MUST:
1. **STOP capturing immediately.** Do NOT silently create "Auth required" placeholders.
2. **Tell the user** explicitly: *"I hit a login wall at [URL]. I need you to log in so I can capture the authenticated screens."*
3. **Open a headed browser** with `launchPersistentContext('/tmp/amazon-profile', { channel: 'chrome' })` so the session persists.
4. **Bring the browser to the foreground**: `osascript -e 'tell application "Google Chrome" to activate'`
5. **Wait for the user to confirm** they've logged in: *"Let me know when you're signed in and I'll resume capturing."*
6. **Resume capturing** from where you left off — navigate to the auth-gated URL again and continue.
7. **Only use placeholders as a LAST RESORT** — if the user explicitly says they can't or won't log in, THEN use "Auth required" placeholders AND list them in the capture summary as NOT captured with the reason.

This applies to ALL diagram types that use screenshots: Journey Maps (Option 2), Screen Flows (Option 4), and User Flows with screenshots enabled.

### 3. Gather Source Material

Supported inputs: text docs, video transcripts, live URLs, screenshots, FigJam boards, Figma files, verbal descriptions, codebases, analytics data. See `references/source-processing.md`.

### 4. Extract

**User flows**: entry point, screens, actions, decisions (with branches), errors, end point.
**Detailed journey maps**: persona, goal, phases, expanded actions per phase, touchpoints, thoughts, emotions (with sentiment level), pain points (with severity), user effort, opportunities.
**Sitemaps / IA**: root page, top-level sections, sub-pages per section, page types (landing, form, dashboard, list, detail, settings), navigation depth levels, external links.

Present as numbered list or indented tree. Confirm with user before rendering.

### 5. Render

**MANDATORY: Re-read BOTH reference files right now, before writing any `use_figma` code:**
1. **`references/design-tokens.md`** — for color keys, text styles, and binding patterns
2. **The diagram-specific reference** — for exact node sizes, layout rules, and code patterns:
   - Option 1 (User Flow) → `references/flow-patterns.md`
   - Option 2 (Journey Map) → `references/journey-map-structure.md`
   - Option 3 (Sitemap) → `references/sitemap-structure.md`
   - Option 4 (Screen Flow) → `references/screen-flow-figjam.md`

**Do NOT skip this re-read. Do NOT rely on memory from the activation read.** The reference files contain exact pixel sizes, color assignments, and layout formulas that MUST be followed precisely. Failure to re-read produces diagrams with wrong node sizes, missing visual elements, and inconsistent styling.

**THIS IS THE MOST CRITICAL STEP. The agent MUST use the exact code patterns from the reference files. No shortcuts, no skipping.**

#### Step 5a: Mandatory preamble — paste into EVERY `use_figma` call

Every `use_figma` call MUST start with the Loveship palette and helpers. Copy from `design-tokens.md`:

```javascript
// === MANDATORY PREAMBLE ===
function h(hex) {
  return { r: parseInt(hex.slice(1,3),16)/255, g: parseInt(hex.slice(3,5),16)/255, b: parseInt(hex.slice(5,7),16)/255 };
}

var C = {
  WHITE: "#FFFFFF", NIGHT50: "#FAFAFC", NIGHT100: "#F5F6FA", NIGHT200: "#EBEDF5",
  NIGHT300: "#E1E3EB", NIGHT400: "#CED0DB", NIGHT500: "#ACAFBF", NIGHT600: "#6C6F80",
  NIGHT700: "#474A59", NIGHT800: "#303240", NIGHT900: "#1D1E26",
  SKY_5: "#F5FDFF", SKY_10: "#E1F4FA", SKY_50: "#009ECC", SKY_60: "#0086AD", SKY_70: "#006B8A",
  GRASS_50: "#67A300", GRASS_70: "#396F1F",
  FIRE_5: "#FEF6F6", FIRE_50: "#FF4242", FIRE_70: "#AD0000",
  SUN_50: "#FCAA00", SUN_70: "#BD5800",
};

function setFill(n, hex) { n.fills = [{ type: "SOLID", color: h(hex) }]; }
function setStroke(n, hex, w) { n.strokes = [{ type: "SOLID", color: h(hex) }]; n.strokeWeight = w || 1; }

// Font loading — Source Sans Pro preferred, Inter fallback
try {
  await figma.loadFontAsync({ family: "Source Sans Pro", style: "Regular" });
  await figma.loadFontAsync({ family: "Source Sans Pro", style: "Semibold" });
  await figma.loadFontAsync({ family: "Source Sans Pro", style: "Bold" });
} catch(e) {}
await figma.loadFontAsync({ family: "Inter", style: "Regular" });
await figma.loadFontAsync({ family: "Inter", style: "Semi Bold" });
await figma.loadFontAsync({ family: "Inter", style: "Bold" });
await figma.loadFontAsync({ family: "Inter", style: "Medium" });

async function makeText(parent, content, fontSize, fontWeight, colorHex) {
  var t = figma.createText();
  var style = fontWeight || "Regular";
  try {
    await figma.loadFontAsync({ family: "Source Sans Pro", style: style });
    t.fontName = { family: "Source Sans Pro", style: style };
  } catch(e) {
    var interStyle = style === "Semibold" ? "Semi Bold" : style;
    await figma.loadFontAsync({ family: "Inter", style: interStyle });
    t.fontName = { family: "Inter", style: interStyle };
  }
  t.characters = content;
  t.fontSize = fontSize;
  t.fills = [{ type: "SOLID", color: h(colorHex || C.NIGHT800) }];
  parent.appendChild(t);
  return t;
}
```

**DO NOT use `bindColor()`, `importVariableByKeyAsync()`, `importStyleByKeyAsync()`, `search_design_system()`, or `get_libraries()`.** All colors and fonts are self-contained in the preamble above. No Figma library dependency.

#### Step 5b: Build order — diagram-specific

Follow the build order for the chosen diagram type. Each type has its own reference file with exact code patterns.

#### User Flow — build order

1. Create file: `create_new_file` → name: "{Flow Name} — User Flow"
2. **Call 1 — Main spine** (~10-12 nodes): Create root frame (`setFill(root, C.WHITE)`, `clipsContent=false`), title, ALL main-path nodes top-to-bottom using `buildNode()`, vertical `drawArrow()` between each pair, branch labels on spine arrows. Return root ID + decision Y positions.
3. **Call 2 — All branches + resize**: Reference root via `getNodeByIdAsync`. Add all LEFT branches, RIGHT branches, stacked alternatives, skip-path bypasses, feedback loops, and error nodes. Resize root frame to fit content. Take screenshot to verify.
4. If screenshots were captured, add in a **Call 3**: upload via `upload_assets` + `curl POST`, place in a dedicated right column (200×125px thumbnails).

**This 2-call pattern is PROVEN** across Rozetka (16 nodes) and Amazon (17 nodes). It replaces the old "max 10 nodes per call" guidance (Rule #60).

**Connector rules (see `flow-patterns.md` for code):**
- Use the NORMALIZED `drawArrow(root, x1, y1, x2, y2)` with `Math.min` for position (Rule #57) — handles all directions
- Use `createVector()` with `ARROW_EQUILATERAL` stroke cap on the end vertex
- All vertical connectors at `node.x + W/2` (centered)
- All horizontal connectors at `node.y + H/2` (centered)
- LEFT branches: `drawArrow(root, COL, midY, leftX+W, midY)` — arrowhead on left node (Rule #58)
- Multiple horizontal branches: stack vertically, each with own arrow from decision (Rule #58)
- Feedback loops: use `drawFeedbackLoop()` L-shaped routing (Rule #59)

**Node types (all 200px wide, 48px tall — Loveship LIGHT theme):**
- **Start/End**: Pill (cornerRadius=9999), `setFill(node, C.SKY_50)`, white text
- **Step/Screen**: Rectangle (cornerRadius=6), `setFill(node, C.WHITE)`, `setStroke(node, C.NIGHT400, 1)`
- **Decision**: Rectangle (cornerRadius=8), `setFill(node, C.SKY_5)`, `setStroke(node, C.SKY_50, 1.5)`
- **Error**: Rectangle (cornerRadius=6), `setFill(node, C.FIRE_5)`, `setStroke(node, C.FIRE_50, 1.5)`

**Screenshot placement:**
- Dedicated column: `x = rightmost_branch_column + 200 + 60`
- Each thumbnail: 200×125px, 4px radius, 1px subtle border
- Dashed 1px connector from flow node right edge to screenshot left edge
- Vertically centered with corresponding flow node

#### Journey Map — flat grid with Loveship light theme

Build as table-like grid. Every cell uses `setFill()` for fills and `makeText()` for text — direct Loveship hex values, no library binding. See `references/journey-map-structure.md` for the complete grid layout, row structure, label construction, and sizing rules.

Rows: Screenshots, Actions, Touchpoints, Thoughts, Emotions, Pain Points, User Effort, Opportunities.
Phase headers use 14px Semibold `C.NIGHT800`. Emotion row uses color-coded text (grass/fire/night). Pain points use severity prefixes (CRITICAL/MAJOR/MINOR) with fire/sun colors.

#### Sitemap / IA — vertical section-based layout with Loveship light theme

Build as a 3-column vertical section layout. See `references/sitemap-structure.md` for complete code patterns.

**CRITICAL: Sitemaps are ALWAYS detailed by default (Rule #52).** The extraction phase must drill at least 3 levels deep. Never produce a shallow top-level-only sitemap unless the user explicitly requests it.

**CRITICAL: Build section-by-section (Rule #53).** Split into multiple `use_figma` calls. If a call fails, retry or skip and continue — never abort silently.

**CRITICAL: Optimal batch size is 3-6 sections per call (Rule #54).** NOT 1-2. Proven sweet spot: 30-55 nodes per call.

1. Create file: `create_new_file` → name: "{Product Name} — Sitemap"
2. **Call 1** — Root frame + title + first 3 sections (~35 nodes):
   - Root frame: `figma.createFrame()`, 1800×6000, white fill (`#FFFFFF`), `clipsContent=false`
   - Title: 30px Bold + 14px subtitle with metadata (domain, page count, depth)
   - Divider: 1680×1 rectangle, NIGHT300
   - First 3 section groups using `secHdr()` + `ch()` helpers from `sitemap-structure.md`
   - **Save the root frame ID** — all subsequent calls reference it
   - 3-column grid: `COL1=60`, `COL2=540`, `COL3=1020`
3. **Calls 2-3** — Append 6-7 sections per call (~44-55 nodes each):
   - Reference root frame ID: `await figma.getNodeByIdAsync("ROOT_ID")`
   - Paste full helper preamble (palette, setFill, setStroke, makeText, secHdr, ch, subSec)
   - Position sections with absolute X/Y within root frame
   - **After each call**: report progress ("Rendered N/M sections, X/Y nodes")
   - **On failure**: reduce batch to 3 sections, retry. If still fails, skip and continue.
4. **Final call** — Last sections + resize-to-fit:
   - Render remaining sections
   - Resize root frame to fit all content (Rule #50 resize pattern)
5. **Audit**: Take screenshot + run node count audit (Rule #22). Compare section names to confirmed tree.

**Node types (Loveship light theme — NO dark backgrounds, NO library imports):**
- **Section Header**: SKY_5 fill + SKY_50 border + emoji + 15px Semi Bold SKY_70 text
- **Sub-section**: NIGHT100 fill + NIGHT300 border + 13px Semi Bold NIGHT700 text
- **Page**: transparent + NIGHT400 border + 13px Medium NIGHT800 text + inline page type suffix
- **External**: transparent + NIGHT400 border + 13px Medium NIGHT500 text + " ↗" suffix

**Layout:**
- 3-column vertical layout (COL1=60, COL2=540, COL3=1020)
- Sections positioned absolutely within root frame
- Auto-layout VERTICAL for each section group (header + children stacked)
- Spacer-based indentation: 16px (L1), 32px (L2), 48px (L3)
- Page type annotations inline as suffix text (not below the node)
- No connectors needed — indentation and visual hierarchy convey structure
- Center parent node above its children
- Max 6–8 nodes per row before wrapping

#### Screen Flow (FigJam) — screenshot wireflow

This type uses `use_figma` on a **FigJam file** (not Figma Design). No design system tokens. Screenshots are the core content.

See `references/screen-flow-figjam.md` for the complete build patterns.

1. **Capture screenshots** via Playwright (reuse existing Step 2 workflow) or accept user-provided screenshot files.
2. **Create FigJam file**: `create_new_file({ fileName: "{Flow Name} — Screen Flow", planKey: PLAN_KEY, editorType: "figjam" })`
3. **Upload screenshots**: `upload_assets({ fileKey: FILE_KEY, count: N })` → POST each screenshot to the returned `submitUrl` via `curl`.
4. **Load skills**: Load `figma-use` AND `figma-use-figjam` skills before calling `use_figma`.
5. **Position and connect** via `use_figma`:
   - Find uploaded frames on the page
   - Position left-to-right in flow order (100px gap)
   - Create `figma.createConnector()` between adjacent screenshots with text labels
   - Add `figma.createSticky()` below each screenshot with screen name
   - For branching: stack alternatives vertically, add decision stickies
   - Optionally group into `figma.createSection()` by phase
6. Share the FigJam file URL with the user.

**Do NOT use `bindColor()`, `makeText()`, design system variable keys, or the design system validation audit for Screen Flow.**

### 6. Validate (MANDATORY — do not skip)

**If the diagram is a Screen Flow (Option 4), skip the design system audit.** Instead:
1. Take a screenshot of the FigJam board via `get_screenshot`
2. Visually confirm all uploaded screenshots are positioned and visible
3. Confirm all connectors link the correct screenshots with labels
4. Confirm sticky notes are placed below each screenshot with screen names
5. If any screenshots are missing or mispositioned, fix via `use_figma`

**For all other types (Options 1–3), run the structural + visual audit:**

After building, run this audit via `use_figma` to verify completeness and take a screenshot for visual check.

```javascript
var root = await figma.getNodeByIdAsync(ROOT_ID);

// Count all node types
var allFrames = root.findAll(n => n.type === "FRAME");
var allText = root.findAll(n => n.type === "TEXT");
var allVectors = root.findAll(n => n.type === "VECTOR");

// For sitemaps: count sections
var sections = root.children.filter(n => n.type === "FRAME" && n.layoutMode === "VERTICAL" && n.name !== "Title");

// Visual check
await root.screenshot();

return {
  frames: allFrames.length,
  textNodes: allText.length,
  connectors: allVectors.length,
  sections: sections.length,
  sectionNames: sections.map(s => s.name)
};
```

**Validation checks:**
- **Node count**: Compare total frames/text to expected count from the extraction phase. Missing nodes = incomplete diagram.
- **Section count (sitemaps)**: Compare section names to the confirmed tree. Every section must be present.
- **Visual check**: Take a screenshot and verify: text is readable (dark on light), no overlapping nodes, connectors connect to correct nodes, no orphaned elements.
- **Connector direction**: Verify all horizontal arrows point in the correct direction — especially left branches and feedback loops (Rules #57-59).
- **Loveship theme check**: Root frame should be white, text should be dark (NIGHT800), accents should be SKY_50. No dark backgrounds.

### 7. Iterate

Share Figma link. Common changes: add/remove steps, adjust layout, add swim lanes, expand sub-flows.

---

## Critical Rules

1. **Ask diagram type first (flow / journey map / sitemap / screen flow), then ask about screenshots (skip for screen flow).** Always. Two questions before any work begins (one question for screen flow).
2. **Use `setFill()` / `setStroke()` with Loveship palette hex values for all colors.** Do NOT use `bindColor()`, `importVariableByKeyAsync()`, or any Figma library variable binding. All colors come from the UUI Loveship palette as raw hex values (see `design-tokens.md`). Use `makeText()` with direct `fontSize` and `fontWeight` parameters — do NOT use `importStyleByKeyAsync()`. Font family is Source Sans Pro (fallback: Inter).
3. **Use the Loveship `C = { ... }` palette object in every `use_figma` call.** Paste the full palette constants from `design-tokens.md` into every script. No `search_design_system()` needed — all colors are predefined hex values.
4. **`setFill()`, `setStroke()`, and `makeText()` are the ONLY way to set colors and create text.** Copy the exact function code from `design-tokens.md` into every `use_figma` call. No Figma library imports, no variable binding.
5. **Draw all UI elements directly.** Tags, dividers, badges, and pills are drawn as simple shapes with Loveship colors. Do NOT use `importComponentSetByKeyAsync` — there is no library dependency.
6. **Font pre-loading is mandatory.** Every `use_figma` call that touches text starts with loading Source Sans Pro Regular, Semibold, Bold. Fall back to Inter if Source Sans Pro is unavailable.
7. **Light theme by default.** Root frames use white (`#FFFFFF`) backgrounds with dark text (`#303240`). This is a light theme — do NOT use dark backgrounds unless explicitly requested.
8. **All nodes 200×48px, LOCKED with `primaryAxisSizingMode = "FIXED"`.** Start/End pills, steps, decisions, errors — ALL 200px wide, 48px tall. After setting `layoutMode = "HORIZONTAL"` for centered text, you MUST set `node.primaryAxisSizingMode = "FIXED"` and `node.counterAxisSizingMode = "FIXED"`. Without this, auto-layout HUGs text content, making nodes different widths and causing arrows to misalign. Decision nodes are rectangles with blue border (NOT rotated diamonds).
9. **Use built-in line arrow caps.** Set `ARROW_EQUILATERAL` on the end vertex via `vectorNetwork` — do NOT create separate polygon arrowhead shapes. This produces cleaner arrows and fewer nodes.
10. **One line = one arrow.** Each connector is a single `createLine()` with an arrow cap on the end vertex. No separate shapes to position or align.
11. **All connectors centered on nodes.** Vertical at `node.x + W/2`. Horizontal at `node.y + H/2`.
12. **Screenshots in dedicated right column.** Use 200×125px. Place ALL screenshots in a column to the right of the entire flow (not inline). Draw 1px dashed connector from node right edge to screenshot left edge.
20. **Clear default fills on auto-layout frames.** `figma.createAutoLayout()` creates frames with a WHITE fill by default. Always set `frame.fills = []` immediately after creation for transparent containers (legend items, row wrappers, etc.).
21. **Render branch-by-branch, not level-by-level.** For sitemaps and large trees, render each branch completely (parent + all descendants) before moving to the next branch. Never render all L1, then all L2, then all L3 — this causes deep branches to be skipped. After each batch, count rendered nodes and compare to the extracted tree. Don't present until ALL branches are complete.
22. **Audit node count after rendering.** Before presenting a sitemap, run `root.findAll(n => n.type === "FRAME").length` and compare to the total nodes in the extracted tree. If the count doesn't match, identify missing branches and render them.
23. **Dynamic parent spacing in trees.** When a parent node has children, the horizontal space reserved for that parent MUST be at least as wide as its children's total width. Calculate `groupWidth = children.length * childWidth + (children.length - 1) * childGap` BEFORE placing parent nodes. Never use fixed spacing between parents — a parent with 6 children needs 6x the space of a parent with 1 child. See `sitemap-structure.md` for the formula.
24. **Exhaustive navigation discovery for sitemaps.** When extracting a sitemap from a live URL, you MUST explore EVERY navigable surface — not just the visible sidebar. This means:
    - **Every dropdown/chevron** in the top nav AND sidebar — hover AND click to reveal sub-items.
    - **Avatar / account menu** — click the user avatar to discover Account, Settings, Sign out, etc.
    - **Organization/workspace selector** — the entry-point picker (e.g. "Choose your organization") is a real page in the IA.
    - **Expandable sidebar sections** — click every chevron/arrow to reveal nested pages.
    - **Tab bars within pages** — navigate into each section and record tab names (e.g. Organization → Members, Billing, Settings...; Product Settings → API Keys, Environments, Clients...).
    - **Utility links on dashboard pages** — Player Search, Downloads, Changelog, etc. are real pages even if they appear as action buttons.
    - **Persistent footer items** — Notifications, Mods Publishing, etc.
    - **Shadow DOM / web components** — if dropdowns don't appear in the accessibility tree, query shadow roots directly.
    Present the COMPLETE tree to the user for confirmation. If the tree exceeds ~80 nodes, propose a scoping strategy (e.g. "Org-level only" vs "Full product drill-down") — do NOT silently skip nodes.
25. **Never silently skip nodes or sub-categories.** A sitemap with only top-level categories is NOT a sitemap — it's a navigation bar. You MUST drill at least 2 levels deep: navigate INTO each section page to discover sub-categories. If a sitemap has more nodes than fit in a single `use_figma` call, **build section-by-section** — render 1-2 sections per call, count after each batch, and continue until every node from the confirmed tree is placed. Never present an incomplete diagram as finished. See `sitemap-structure.md` § Mandatory Drill-Down.
26. **Vertical layout for large sitemaps (>50 nodes).** When the extracted tree has more than 50 nodes, switch from the default horizontal top-down layout to a **vertical list layout** organized by section. Each section becomes a vertical group: section header on top, children listed vertically below with indent. Sections are arranged in a multi-column grid (2–3 columns) to fit the frame. This prevents the horizontal explosion that makes wide sitemaps unreadable. See `sitemap-structure.md` § Vertical Layout for the pattern.
27. **Screen Flow uses `use_figma` on a FigJam file with `figma-use-figjam` skill.** Option 4 creates a FigJam file (`editorType: "figjam"`), uploads screenshots via `upload_assets`, then uses `use_figma` to position frames, create Connectors, and add Stickies. No design system tokens. Must load both `figma-use` and `figma-use-figjam` skills. See `screen-flow-figjam.md`.
28. **Screen Flow wireflow rules.** Screenshots are REQUIRED (not optional). Upload via `upload_assets`, position left-to-right with 100px gap, connect with `createConnector()`, annotate with `createSticky()`. Branch alternatives stack vertically. Max ~15 screenshots per flow — split larger flows into sub-flows.
29. **Preserve original screenshot aspect ratio.** `upload_assets` creates frames at a default size (e.g. 400x300) that does NOT match the actual image dimensions. You MUST check the real pixel dimensions of the screenshots BEFORE uploading (e.g. PNG header or `sips`), then resize the uploaded frames to match: `W = 520, H = round(520 * (imgH / imgW))`. Use `scaleMode: "FILL"` — it only crops when the aspect ratio is wrong. Always match the ratio exactly.
32. **Capture ALL branches and corner cases.** When a flow has decision points (dropdowns, modals, role-based routing), capture screenshots for EVERY branch — not just the happy path. For each decision, walk through each option and capture the resulting screen. Present the complete set to the user before building. If you skip a branch, the wireflow is incomplete.
33. **Connectors must not overlap screenshots or stickies.** FigJam connectors with `magnet: "AUTO"` can route through other nodes. To prevent this, position branch screenshots far enough apart that connectors route cleanly around them. Use `connectorLineType: "STRAIGHT"` or increase horizontal offset for branch targets so elbowed connectors have room. If a connector still overlaps, add horizontal padding (move branch screenshots further right) or use `magnet: "RIGHT"` on the source and `magnet: "LEFT"` on the target to force horizontal-first routing.
34. **Ask the user to specify the flow's starting point before any capture.** Never assume where a flow begins. Before capturing ANY screenshots, proactively suggest logical starting points based on the target URL and feature:
    - *"Where should this flow start? Here are common options:*
      - *Login / Landing page — captures the full journey from first visit*
      - *Dashboard — starts after authentication*
      - *Product/feature page — starts at the feature itself"*
    - Default suggestion should be the earliest meaningful entry point (e.g., Dashboard if auth is required, Landing page if public). Always capture the COMPLETE journey from the user-specified entry point through to the end goal — including navigation steps to reach the feature (selecting an org, picking a product, opening a sidebar section). This ensures the wireflow tells the full story, not just the feature in isolation.
35. **Connector labels MUST start with an action verb.** Every connector text label must describe the user's action with a verb. Examples: "Clicks Create achievement", "Selects UX organization", "Hovers over dropdown", "Submits form", "Navigates to Achievements". Never use bare nouns like "Create" or "Next" — always include the verb that describes HOW the user triggers the transition. For conditional paths use "If [condition]" format. For system-initiated transitions use passive voice: "Redirected to dashboard", "Modal appears".
36. **Flow title must be visually prominent.** The Screen Flow title sticky (or text node for Options 1–3) must be visually distinct from all other annotations. For FigJam Screen Flows: create the title as a `createShapeWithText()` with `shapeType: "ROUNDED_RECTANGLE"`, large font size, and position it above the entire flow with clear vertical separation (at least 80px gap). It should read as the banner/header of the entire board — not a regular sticky that blends in. For Figma Design flows (Options 1–3): use the `headingLG` text style at increased scale.
37. **Never truncate text on stickies or connector labels.** Reserve as much space as needed for text content. Stickies should contain full, readable descriptions — never cut off mid-sentence. If a sticky note's text is long, let it grow naturally. When positioning stickies, account for their actual rendered size (measure `sticky.width` and `sticky.height` AFTER setting text) rather than assuming a fixed size. The same applies to connector labels — use the full descriptive text, don't abbreviate.
38. **Sticky notes must include detailed action explanations.** Every screenshot's sticky note should describe not just the screen name but WHAT the user does on this screen and WHY it matters in the flow. Format: **"Screen Name — Description of what happens here and what the user's goal is."** Examples:
    - Bad: "Achievements List"
    - Good: "Achievements List — Displays all configured achievements with visibility status, icons, and descriptions. User clicks 'Create achievement' to start the creation flow."
    - Bad: "Select Stat"
    - Good: "Step 1: Select Stat — User picks up to 3 stats that will automatically unlock this achievement as players progress. Required step for stat-based achievements only."
    Decision stickies should list ALL options and explain the consequence of each choice.
39. **Exhaustive hover and dropdown exploration applies to ALL diagram types using a live URL — not just Screen Flow.** Whenever you explore a live URL via Playwright for ANY diagram type (User Flow, Journey Map, Sitemap, or Screen Flow), you MUST exhaustively explore every interactive element before presenting the extraction:
    - **Hover** over buttons, links, and menu items to reveal tooltips, dropdowns, or sub-menus.
    - **Click every dropdown/menu** and record all available options. For Screen Flow, screenshot the open state. For Sitemaps, record the discovered pages.
    - **Expand all collapsible sections** (chevrons, accordions, "more" buttons) — capture what's inside.
    - **Check contextual menus** ("..." buttons, right-click menus, kebab menus) — record the actions available.
    - **Navigate into every section** and record its sub-pages, tabs, and nested content. Don't just read the top-level sidebar — click into each section to discover what's inside.
    - **Check tab bars within pages** — every tab is a distinct page/view that belongs in the sitemap.
    - **Try the ecosystem/global dropdowns** (e.g., product switcher, org selector, locale menu) — these reveal navigation surfaces that aren't in the sidebar.
    - For Screen Flows: if a dropdown has 3 options, that's 3 branches to capture — not 1 with a text note about the others.
    - For Sitemaps: every discovered page goes into the tree. A section with undiscovered sub-pages is incomplete.
    This rule supersedes rule #24 (sitemap-specific exploration) by applying the same rigor to ALL diagram types.
40. **Present a capture summary before building — never silently fail.** Before generating the FigJam wireflow, present a structured summary to the user:
    - **Captured screens**: numbered list of all screenshots taken, with descriptions.
    - **Flow paths**: all connections between screens (main path + branches).
    - **Branches explored**: list of decision points and which options were captured.
    - **NOT captured / inaccessible**: explicitly list any screens, options, or paths that could NOT be captured and WHY (e.g., "Bulk Import option — requires a ZIP file upload, could not proceed without test data", "Admin panel — insufficient permissions", "Payment form — requires real payment method"). NEVER silently skip a screen. If Playwright could not access an element (shadow DOM, iframe, auth wall, file upload required), state it clearly.
    - **Suggested follow-ups**: propose additional captures the user might want (e.g., "error states if form is submitted empty", "what happens after Create succeeds").
    The user must confirm this summary before the FigJam build begins.
41. **MANDATORY: Every FigJam Screen Flow board MUST include "Not Captured" and "Follow-ups" sections.** These are not optional — they are required parts of every wireflow deliverable, just like the title banner and stickies. After placing all screenshots and connectors, ALWAYS add these two annotation sections at the bottom of the FigJam canvas:
    - **"NOT Captured / Inaccessible"** — a `createShapeWithText()` header with `ROUNDED_RECTANGLE` shape, followed by one sticky per item. Each sticky must explain WHAT was not captured and WHY (e.g., "Bulk Import completion — requires valid ZIP file with achievement data; could not proceed past upload without test data"). Use detailed explanations, not vague summaries. If nothing was inaccessible, add a single green sticky: "All identified flows were captured successfully."
    - **"Suggested Follow-ups"** — a `createShapeWithText()` header with `ROUNDED_RECTANGLE` shape, followed by one sticky per recommendation. Each sticky describes a concrete next capture that would extend the wireflow (e.g., "Capture success state by creating a test achievement with dummy data and icons"). Always suggest at least 2-3 follow-ups even if the flow seems complete — there are always edge cases, error states, or alternative paths worth exploring.
    Position these sections below the last row of screenshots with at least 500px vertical separation from the last sticky. These sections serve as the flow's documentation trail — they tell anyone viewing the board what was covered and what remains.
42. **Capture every flow you can — do not skip what is safely explorable.** Before listing anything as "NOT captured", ask yourself: can this be triggered without permanent side effects? If yes, you MUST capture it. Specifically:
    - **Validation errors** — always submit empty/invalid forms to capture error states. This never creates data.
    - **Delete confirmation dialogs** — always click Delete to reveal the confirmation dialog, then click Cancel. The dialog itself is a critical screen in the flow.
    - **Cancel/Back paths** — always capture what happens when the user cancels mid-flow (does a confirmation dialog appear? does the form reset?).
    - **Hover states and tooltips** — always hover interactive elements to capture revealed content.
    - **Empty states** — if you can navigate to a page that would be empty for a new user, capture it.
    - Only mark something as "NOT captured" when it truly requires external input you don't have (file uploads with specific data, payment methods, third-party auth), would cause irreversible changes (actually deleting production data, submitting real transactions), or is blocked by permissions/access. The goal is to minimize the "NOT captured" list to only genuinely impossible items — not items you were cautious about. Be aggressive about exploring safely.
43. **Horizontal-first layout — only branch vertically at decision points.** The flow should read left-to-right as the primary direction. Place screenshots in a continuous horizontal line along the main (happy) path. Only introduce a new vertical row when the flow branches at a decision point (e.g., a dropdown with multiple options). Each branch continues horizontally on its own row. Side paths (e.g., edit/delete from a list) drop down as a vertical fork from the relevant screen. Never stack screens vertically just because there are many of them — horizontal scrolling is preferable to a tall, narrow diagram.
44. **Connector labels and sticky notes must NEVER overlap screenshots.** Text elements (connector labels, sticky notes, shape-with-text annotations) must be fully clear of all screenshot frames — no partial overlap, no text rendering on top of images. To prevent this:
    - **Connector labels**: Use explicit `magnet` values (`"RIGHT"` → `"LEFT"` for horizontal, `"BOTTOM"` → `"TOP"` for vertical) so connectors route around screenshots, not through them. If a connector label still overlaps a screenshot, increase the horizontal or vertical gap between the connected screenshots.
    - **Sticky notes**: Position stickies with enough vertical clearance below their screenshot (`screenshot.y + screenshot.height + 20`). Before placing the NEXT row of screenshots, calculate the actual bottom edge of the tallest sticky in the current row and add at least 60px padding. Measure sticky height AFTER setting text — don't assume a fixed size.
    - **Branch connector labels**: When connectors fan out from a decision point to multiple branches, the labels can pile up and overlap nearby screenshots. Increase the vertical gap between branch rows to at least `screenshot_height + sticky_height + 100px` to give connector labels room to render.
    - **After placing all elements**, take a screenshot and visually verify no text overlaps any image. If overlap is found, increase spacing and reposition.
45. **Branch rows start to the RIGHT of the decision screenshot, not below it.** When a decision point (dropdown, modal, menu) fans out to multiple branches, each branch row must begin at `decision.x + decision.width + GAP` — to the right of the decision screenshot. The decision screenshot itself stays on the main row. Branch rows are stacked vertically but their first screenshot is horizontally offset to the right of the decision, so connectors go RIGHT then DOWN, not straight DOWN through the decision screenshot. This prevents branch screenshots from being placed directly below the decision and overlapping with its sticky note or connector labels.
46. **No node may overlap any screenshot frame.** Before placing any element (screenshot, sticky, connector label, shape-with-text, section header), verify it does not overlap any existing screenshot frame's bounding box. Use this check: two rectangles overlap if `(A.x < B.x + B.width) && (A.x + A.width > B.x) && (A.y < B.y + B.height) && (A.y + A.height > B.y)`. If overlap is detected, shift the new element until it clears all screenshots. When calculating row positions, reserve enough vertical space per row: `ROW_HEIGHT = screenshot_height + max_sticky_height + connector_label_height + padding` (at minimum `screenshot_height + 350px`). After all elements are placed, run a final overlap check via `use_figma` — checking screenshots vs ALL other nodes (stickies, shapes, other screenshots) — and fix any violations before presenting.
47. **Separate branch zones horizontally to prevent connector path collisions.** FigJam elbowed connectors that drop vertically from one row to another will pass THROUGH any screenshots in intermediate rows that occupy the same horizontal zone. This is the #1 cause of visual overlap in multi-row flows. To prevent it:
    - **Map out horizontal zones** before placing anything. Each branch from a different decision point gets its own non-overlapping horizontal zone (X range).
    - **Side branches go LEFT**, main branches go RIGHT. If the main path has a decision at column 5, side branches from earlier screens (column 4) should be placed in columns 0-3 (left zone). Branches from the decision should be placed in columns 6+ (right zone). This ensures vertical connector paths from the decision never cross through side branch screenshots.
    - **Verify connector corridors are clear.** For every BOTTOM→TOP connector, check that the vertical X corridor between source and target contains NO screenshots from other rows. If it does, move the target screenshot to a different X zone.
    - **Place ALL branch targets on the SAME Row 2** — never create a Row 3 that a connector must skip over. If a decision has 3 branches, spread them horizontally across Row 2 (left zone, middle zone, right zone) rather than stacking vertically on separate rows. A connector that skips a row will ALWAYS have its elbowed path pass through the intermediate row's content.
    - **Calculate Row 2 Y so the elbowed connector midpoint falls in empty space.** FigJam elbowed BOTTOM→TOP connectors have a horizontal turn at the midpoint Y = `(sourceBottom + targetTop) / 2`. This horizontal segment must be BELOW all Row 1 stickies and ABOVE all Row 2 screenshots. Formula: `Y2 > 2 * maxStickyBottom - sourceBottom`. For typical values (sourceBottom=394, maxStickyBottom=766): `Y2 > 2*766 - 394 = 1138`, so **Y2 = 1200** ensures the midpoint (797) clears Row 1's tallest sticky (766) with margin.
    - **Never connect across more than 1 row gap with elbowed connectors.** If you need Row 3, connect Row 1→Row 2 and Row 2→Row 3 separately — never Row 1→Row 3 directly. An elbowed connector spanning 2+ rows will route through all intermediate content.
50. **Resize the root frame to fit content after all elements are placed.** Never use a fixed frame size (e.g., `root.resize(2400, 3200)`) as the final size. After all nodes, connectors, and annotations are placed, run a final `use_figma` call to measure the bounding box of all children and resize the root frame to fit with padding:
```javascript
var root = await figma.getNodeByIdAsync(ROOT_ID);
var minX = Infinity, minY = Infinity, maxX = -Infinity, maxY = -Infinity;
for (var child of root.children) {
  minX = Math.min(minX, child.x);
  minY = Math.min(minY, child.y);
  maxX = Math.max(maxX, child.x + child.width);
  maxY = Math.max(maxY, child.y + child.height);
}
var PAD = 60;
// Shift all children so content starts at (PAD, PAD)
var shiftX = PAD - minX;
var shiftY = PAD - minY;
for (var child of root.children) { child.x += shiftX; child.y += shiftY; }
root.resize(maxX - minX + PAD * 2, maxY - minY + PAD * 2);
```
    This ensures the frame tightly wraps the diagram content with consistent padding — no oversized empty space below or to the right.
49. **Re-read the relevant reference file IMMEDIATELY before rendering — never work from memory.** Before writing ANY `use_figma` code for a diagram, re-read the specific reference file for that diagram type:
    - User Flow → `references/flow-patterns.md` + `references/design-tokens.md`
    - Journey Map → `references/journey-map-structure.md` + `references/design-tokens.md`
    - Sitemap → `references/sitemap-structure.md` + `references/design-tokens.md`
    - Screen Flow → `references/screen-flow-figjam.md`
    The reference files contain exact node sizes, colors, layout rules, and code patterns. Do NOT rely on memory or earlier reads — re-read the file fresh right before the render step. Deviating from the reference (e.g., using auto-layout chips instead of the specified 180×36px nav items, or using full-width rows instead of 2-3 column grid) produces diagrams that look "weird" and inconsistent with the plugin's design language.
48. **Horizontal GAP must be wide enough to display the longest connector label without clipping.** FigJam renders connector labels along the connector path. If the path between two screenshots is shorter than the label text, the text is clipped mid-word. To prevent this:
    - **Use a minimum horizontal GAP of 300px** between adjacent screenshots (not 140px). This gives connector labels enough horizontal space to render fully.
    - **Before finalizing the layout**, check the longest connector label in the flow. Estimate its pixel width at ~8px per character (FigJam default font). If `longestLabel.length * 8 > GAP`, increase GAP to `longestLabel.length * 8 + 40` (40px padding).
    - **For vertical connectors** (BOTTOM→TOP between rows), the label renders along the vertical segment. Ensure ROW_GAP is tall enough to fit the label text vertically — same formula applies.
    - If a label is too long for any reasonable gap, shorten it while keeping the action verb (e.g., "Selects UX org, Clicks Continue" → "Selects org, Continues").
13. **One goal per flow.** Break broad requests into sub-flows.
14. **Show error paths.** Every action that can fail needs a failure branch.
15. **One direction.** Top-to-bottom or left-to-right, not both.
16. **Label every branch.** Yes/No, Success/Failure on decision arrows.
17. **Validate before presenting.** Take a screenshot after building to verify all colors match the Loveship palette, text is readable (dark on light), and layout is clean. Check that no fills use dark-mode colors.
18. **Static only.** No components with variants, no prototyping, no hover states. All styling uses raw Loveship palette hex values.
19. **No Figma library dependency.** Do NOT call `search_design_system`, `get_libraries`, `importVariableByKeyAsync`, or `importStyleByKeyAsync`. All tokens are self-contained in the `C = { ... }` palette and `makeText()` function from `design-tokens.md`.
52. **Sitemaps MUST be detailed by default — never produce a shallow top-level-only sitemap.** When the user requests a sitemap, the output must include AT MINIMUM 3 levels of depth (Root → Sections → Sub-pages → Notable children). A sitemap that only lists top-level nav items is a navigation bar, not a sitemap. During extraction:
    - Navigate INTO every section to discover its sub-pages, tabs, and nested content.
    - For e-commerce sites: drill into category pages to list sub-categories (e.g., "Electronics → Laptops, Phones, TVs, Accessories").
    - For SaaS apps: explore every sidebar section, every tab bar, every settings page.
    - For content sites: map content categories, article types, and user-generated sections.
    - Present the COMPLETE detailed tree to the user BEFORE rendering. If the tree has >80 nodes, propose scoping — but never silently produce a shallow sitemap.
    - **Default is ALWAYS detailed.** The user must explicitly ask for "top-level only" or "shallow" to get a reduced sitemap. Never reduce detail level on your own.
53. **Build sitemaps section-by-section with error recovery — NEVER fail silently.** Large sitemaps MUST be built incrementally across multiple `use_figma` calls. If any single call fails, the agent MUST retry or skip that section and continue with the next — never abort the entire sitemap because one section failed. Pattern:
    - **Call 1**: Create root frame + title + first 3 sections (~35 nodes). Save root frame ID.
    - **Call 2-3**: Append 6-7 sections per call (~44-55 nodes each). Reference root frame ID.
    - **Final call**: Remaining sections + resize-to-fit + node count audit.
    - **After each call**: Report progress: "Rendered N/M sections (X/Y nodes). Remaining: [list]."
    - **On failure**: If a `use_figma` call fails (timeout, API error, too many nodes), (a) reduce the batch size to 3 sections per call, (b) retry the failed batch, (c) if it still fails, skip that batch and continue with the next. After all sections are attempted, report any skipped sections to the user with the error reason.
    - **Never present an incomplete sitemap as finished.** After all calls, run the node count audit (Rule #22). If sections are missing, identify which were skipped and either retry them or tell the user "Section X could not be rendered because [reason]".
    - **If Figma rate limit is hit**: Check if the user has multiple Figma plans (Starter vs Enterprise). Enterprise plans have higher limits. Offer to recreate the file on the higher-tier plan. Do NOT silently stop — always explain and offer alternatives.
54. **Optimal batch size: 3-6 sections per `use_figma` call (30-55 nodes).** This is the PROVEN sweet spot from real-world testing. Smaller batches (1-2 sections) waste API calls and are slow. Larger batches (8+ sections) risk timeout. The proven pattern for a ~160-node sitemap with 24 sections is 4 calls total:
    - Call 1: 3 sections (35 nodes) — includes root frame, title, helpers
    - Call 2: 6 sections (44 nodes)
    - Call 3: 7 sections (55 nodes)
    - Call 4: 8 sections + resize (30 nodes)
55. **Use `browser_evaluate` for programmatic extraction from live URLs.** When extracting a sitemap from a live URL, do NOT navigate to each page manually. Instead, use Playwright's `browser_evaluate` to run JavaScript that programmatically extracts navigation links:
    ```javascript
    // Extract category links from page
    () => {
      const cats = [];
      document.querySelectorAll('a').forEach(a => {
        const href = a.href;
        const text = a.textContent.trim();
        if (text && href && href.match(/\/c\d+\//) && text.length > 2 && text.length < 80) {
          cats.push({ text, href });
        }
      });
      const seen = new Set();
      return cats.filter(c => { if (seen.has(c.href)) return false; seen.add(c.href); return true; });
    }
    ```
    Adapt the URL pattern regex for each site. Navigate to 3-5 key category pages and extract sub-categories from each. This is 10x faster than hovering/clicking through menus manually. Also extract footer links, utility links (cart, wishlist, tracking), and account pages separately.
56. **Paste the FULL helper function set into every `use_figma` call.** Every call must include: `h()`, `C = {}` palette, `setFill()`, `setStroke()`, font loading, `makeText()`, `secHdr()`, `ch()`, `subSec()`. These are defined in `sitemap-structure.md` § Layout Rules. Do NOT assume helpers persist between calls — each `use_figma` call is a fresh execution context. The helpers are compact (~30 lines total) and the cost of pasting them is negligible compared to the cost of a failed call.
57. **Use the NORMALIZED `drawArrow()` for ALL flow connectors — handles all directions.** The arrow helper MUST use `Math.min(x1,x2)` and `Math.min(y1,y2)` to calculate the vector position, with local coordinates relative to that origin. The old pattern (`vec.x = x1`) BREAKS for leftward or upward arrows because Figma vectors need non-negative local coordinates. See `flow-patterns.md` § Generic Arrow for the correct code. This fixes disconnected horizontal arrows in user flows.
58. **Never chain horizontal branch nodes — each branch gets its own arrow from the decision.** When a decision node has multiple horizontal alternatives (e.g., Auth Method → Google, Apple), each alternative MUST get its own arrow directly from the decision node. Do NOT chain them (Auth → Google → Apple) — this implies a sequential relationship that doesn't exist. Stack alternatives vertically at the same X position: first at decisionY, second at decisionY+H+30. See `flow-patterns.md` § Multiple horizontal branches.
59. **Feedback loop connectors use L-shaped routing — never draw backward through other nodes.** When a node (e.g., "Show Error Messages") loops back to an earlier node (e.g., "Enter Recipient Info"), use a 3-segment L-shaped path that routes AROUND the flow: (1) right from source, (2) up to target Y, (3) left back to target right edge. Use `drawFeedbackLoop()` from `flow-patterns.md`. Never draw a straight horizontal line backward — it will cross through intermediate nodes.
60. **Build user flows in exactly 2 `use_figma` calls — not 10+ incremental calls.** The proven pattern: **Call 1** builds the entire main spine (all nodes top-to-bottom + vertical connectors + spine labels). **Call 2** adds all branches (left, right, stacked, skip-paths, feedback loops) + resizes the root frame. This 2-call pattern was proven across Rozetka (16 nodes, 4 decisions) and Amazon (17 nodes, 5 decisions). The old "max 10 nodes per call" guidance was too conservative and produced unnecessary API calls. See `flow-patterns.md` § Proven Build Order.
61. **Skip-path bypass uses a 3-segment right corridor.** When a decision's "Yes" path should skip multiple spine nodes (e.g., "Signed In? Yes" skips auth to reach Review Form), route through a right-side bypass corridor: (1) horizontal right from decision, (2) vertical down to target Y, (3) horizontal left back into target. This creates a visible bypass lane on the right side of the diagram. See `flow-patterns.md` § Skip-path bypass.
62. **Start pill = blue (SKY_50), End pill = green (GRASS_50).** Differentiate start and end nodes visually. The old pattern used SKY_50 for both, making them indistinguishable. Green for "success/complete" follows the Loveship semantic color system (grass = success).
63. **Vertical arrows must span the FULL gap — never subtract H.** The correct pattern after placing a node: `cy += H; drawArrow(root, x, cy, x, cy + VGAP); cy += VGAP;`. The arrow goes from node bottom (`cy`) to next node top (`cy + VGAP`), spanning the full VGAP distance. The WRONG pattern `cy + VGAP - H` creates arrows that are 48px too short, leaving a visible gap between the arrowhead and the next node. This was the #1 visual bug in early user flow renders.
51. **Always include screenshots in journey maps when available.** If screenshots were captured (via live URL, video, or provided files) during Step 2, they MUST be placed as a dedicated "Screenshots" row in the journey map — directly below the phase headers and above the Actions row. Each phase column gets a thumbnail of the corresponding screen. Screenshot cells use `setFill(cell, C.NIGHT100)` with 8px padding, and each image is resized to fill the cell width with a 4px corner radius. A caption label below each image identifies the screen name using `makeText(cell, name, 11, "Semibold", C.NIGHT600)`. Do NOT skip this row or relegate screenshots to a separate frame — they are integral to the journey map and provide essential visual context for each phase. The only exception is when the user explicitly chose "No screenshots" in Step 2.
64. **NEVER skip Step 2 (Ask about screenshots) — even when the user provides the diagram type upfront.** Step 2 is MANDATORY for ALL diagram types except when the user explicitly says "no screenshots" in the same message. If the user says "build a journey map for Amazon", you MUST still ask: "Should I include screenshots? (Live URL / Video / Files / No)". This applies even when the diagram type was implied from a previous conversation. The agent skipping Step 2 was the #1 workflow violation observed in testing — it resulted in journey maps without a Screenshots row, which significantly reduces the deliverable's value. The two-question sequence (Step 1: type, Step 2: screenshots) must ALWAYS happen before any extraction or rendering begins.
65. **When you hit an auth wall during screenshot capture, STOP and ask the user to log in — NEVER silently skip.** If Playwright navigates to a URL and gets redirected to a login/sign-in page (detected by URL containing `/signin`, `/login`, `/auth`, or page title containing "Sign In"), the agent MUST: (1) Stop capturing immediately. (2) Open a headed browser with `launchPersistentContext` so the session persists. (3) Bring the browser to foreground with `osascript`. (4) Tell the user: "I hit a login wall at [URL]. Please sign in, then let me know when you're ready." (5) Wait for confirmation before resuming. (6) Only use "Auth required" placeholders if the user explicitly declines to log in. Silently producing placeholders instead of asking the user to log in was the #2 quality issue in testing — it resulted in incomplete journey maps and screen flows with missing authenticated screens.
66. **After the user logs in, navigate BACK to the intended URL — never assume the redirect worked.** Most sites redirect to their homepage after login, NOT to the page you came from (even with `return_to` params). After the user confirms they're signed in: (1) Check the current URL — if it's the homepage or an unexpected page, navigate back explicitly (e.g., go to cart, then click checkout again). (2) Verify you're on the intended authenticated page before resuming capture. (3) If the site loses the cart/state after login, re-add the item and re-navigate. This was observed with Amazon: after sign-in, the browser landed on the homepage instead of checkout — had to manually navigate cart → checkout to resume.
67. **Upload ALL captured screenshots — including auth wall screens.** The sign-in/login page IS a real screen in the user's journey and belongs in the Auth phase of the journey map. Do not skip it just because it's "the auth wall." If you captured 6 screenshots, upload 6 — not 5. Every captured screen maps to a phase.
68. **Build journey maps in exactly 2 `use_figma` calls.** The proven pattern: **Call 1** creates the root frame, title, phase headers, screenshots row (with uploaded images), actions row, and touchpoints row. **Call 2** adds thoughts, emotions, pain points, user effort, and opportunities rows. This was proven across both Amazon Product Review (6 phases, 8 rows) and Amazon Checkout (6 phases, 9 rows with screenshots). The old guidance of building row-by-row was too conservative.
69. **Screenshot thumbnail sizing in journey maps: `height = Math.round(cellWidth * (imgHeight / imgWidth))`.** For standard 1728×911 screenshots in a 216px-wide cell: height = `Math.round(216 * (911/1728))` = 114px. Always calculate from actual image dimensions, never use a hardcoded height. Set `cornerRadius = 4` on the image frame. Set `layoutSizingHorizontal = "FILL"` so the image scales with the column width.
70. **If a screenshot upload fails, request a new upload URL and retry — do not skip.** Upload URLs are single-use and expire after 10 minutes. If a POST fails (network error, typo, timeout), call `upload_assets` again to get a fresh URL for that one image, then retry. Never leave a gap in the screenshots row because of a failed upload.
