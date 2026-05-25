# Figma UX Flow Builder

Build **user flow diagrams**, **detailed journey maps**, **sitemaps**, and **screen flow diagrams** in Figma Design and FigJam from any source — project docs, video walkthroughs, live app URLs, screenshots, or verbal descriptions.

Options 1–3 use **design system** tokens in Figma Design — zero raw hex values. Option 4 uses Mermaid in FigJam for quick conceptual flows.

## One Command

**`/user-journey-mapping`**

## Guided Flow

```
Step 1: What type of diagram?
        → User Flow = step-by-step task diagram (Figma Design)
        → Detailed Journey Map = experience grid with emotions (Figma Design)
        → Sitemap / IA = hierarchical page tree (Figma Design)
        → Screen Flow = quick conceptual flow (FigJam, via Mermaid)

Step 2: Include screenshots? (skipped for Screen Flow)
        → Yes, I have a live URL → Playwright opens a browser, you navigate,
          screenshots auto-captured every screen change
        → Yes, I have a video → Provide transcript or timestamps for key frames
        → Yes, I have screenshot files → Share image paths
        → No → Diagram only, no screenshots

Step 3: What's the source?
        → Project docs, PRD, or spec
        → Video walkthrough or transcript
        → Live app URL
        → Screenshots or Figma/FigJam boards
        → Codebase (reads routes/components)
        → Verbal description

Step 4: Extract steps
        → Parses source into numbered steps
        → Shows the extraction for confirmation

Step 5: Build in Figma Design or FigJam
        → Options 1–3: Creates Figma Design file with design system tokens
        → Option 4: Creates FigJam file via Mermaid generate_diagram
        → Renders nodes, connectors, labels
        → All colors bound to design system variables (Options 1–3)

Step 6: Validate
        → Audits design system binding: text styles, text colors, fills, strokes

Step 7: Iterate
        → Add/remove steps, adjust layout, expand sub-flows
```

## Four Diagram Types

### 1. User Flow
Step-by-step diagram for a specific task. For devs, PMs, designers. *Figma Design.*

```
[Start] → [Screen] → <Decision?> → Yes → [Screen] → [End]
                         ↓
                        No → [Error] → [Recovery]
```

### 2. Detailed Journey Map
Full experience across phases with data-rich rows. For UX audits, stakeholders. *Figma Design.*

```
┌──────────┬──────────┬──────────┬──────────┐
│ Phase 1  │ Phase 2  │ Phase 3  │ Phase 4  │
├──────────┼──────────┼──────────┼──────────┤
│ Actions  │ Actions  │ Actions  │ Actions  │
│ Thoughts │ Thoughts │ Thoughts │ Thoughts │
│ Emotions │ Emotions │ Emotions │ Emotions │ ← sentiment curve
│ Pain pts │ Pain pts │ Pain pts │ Pain pts │ ← severity tags
│ Effort   │ Effort   │ Effort   │ Effort   │ ← Low/Med/High pills
│ Opps     │ Opps     │ Opps     │ Opps     │
└──────────┴──────────┴──────────┴──────────┘
```

### 3. Sitemap / IA
Hierarchical page tree. For IA audits, navigation redesigns. *Figma Design.*

```
                    [Home]
        ┌──────────┬──┴──────────┐
   [Section A]  [Section B]  [Section C]
    ├─ Page 1    ├─ Page 4    ├─ Page 7
    ├─ Page 2    └─ Page 5    └─ Page 8
    └─ Page 3
```

### 4. Screen Flow (FigJam)
Wireflow with actual screenshots connected by arrows. For visual walkthroughs, demos. *FigJam.*

```
  ┌──────────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐
  │Screenshot│───>│Screenshot│───>│Screenshot│───>│Screenshot│
  │ Landing  │    │  Login   │    │ Dashboard│    │ Settings │
  └──────────┘    └──────────┘    └──────────┘    └──────────┘
  [Sticky:        [Sticky:        [Sticky:        [Sticky:
   "Entry"]        "Auth"]         "Home"]         "Config"]
```

## Automated Capture (Zero Manual Input)

The plugin auto-navigates and captures entire apps without manual input. It uses a **persistent browser profile** so you only log in once — subsequent runs are fully automated.

### Setup (one-time)

```bash
# Install Playwright + Chrome
npm install playwright
npx playwright install chromium
```

### How It Works

1. **First run**: Plugin opens Chrome with a persistent profile (`~/.ux-flow-browser-profile/`). If the app requires auth, you log in once manually. The session is saved.
2. **Every subsequent run**: Browser opens already logged in. The plugin then:
   - Auto-navigates all sidebar sections, dropdowns, tabs
   - Expands all collapsible menus and chevrons
   - Captures screenshots at each distinct screen
   - Discovers all pages for sitemaps
   - Captures hover states, kebab menus, modals
   - Presents a complete capture summary
   - Builds the diagram in Figma — no user input needed
3. **Session expires?** Plugin detects the login page and asks you to re-login once.

### Persistent Profile

The plugin uses Playwright's `launchPersistentContext` to store cookies, localStorage, and session data:

```javascript
const { chromium } = require('playwright');
const browser = await chromium.launchPersistentContext(
  '~/.ux-flow-browser-profile',
  { headless: false, channel: 'chrome' }
);
```

This works with any auth method — SSO, MFA, OAuth, user accounts. No cookie extraction or injection needed.

## Screenshot Capture

| Method | How it works |
|--------|-------------|
| **Live URL (auto)** | Persistent browser profile. Auto-navigates all pages, captures screenshots on URL change. Zero manual input after first login. |
| **Video / transcript** | Provide a Loom/YouTube transcript or timestamps. Key frames extracted at transition points. |
| **Screenshot files** | Share image paths directly. |
| **FigJam / Figma** | Reads existing boards or design files for screen references. |

Screenshots are placed as **180x112px thumbnails** inline next to their corresponding flow nodes, connected by a 1px line.

## Supported Sources

| Source | How it's processed |
|--------|-------------------|
| **Text docs / PRD** | Extracts steps, decisions, error paths from written specs |
| **Video / transcript** | Identifies screen transitions and user actions |
| **Live app URL** | Playwright captures screenshots as you navigate |
| **Screenshots** | Analyzes visual sequence to reconstruct the flow |
| **FigJam board** | Reads existing sticky notes and connectors |
| **Codebase** | Reads router config, page components, navigation logic |
| **Verbal description** | Parses natural language into structured steps |

## Design System Tokens

All output uses design system tokens. The plugin works with any design system library connected to your Figma file:

| Element | Design Token |
|---------|-----------|
| Background | `background/default` variable |
| Step nodes | `background/elevated/low` variable |
| Decision nodes | `background/elevated/high` variable |
| Start/End nodes | `foreground/accent` variable |
| Connectors | `border/default` variable, 2px |
| Text | Design system library text styles (headingLG, paragraphSM, uiSMBold, etc.) |
| Status tags | Tag Static component instances (Green/Red/Orange/Yellow) |
| Dividers | Divider component instances |

## Recommended: Claude Opus

This plugin works best with **Claude Opus** for accurate flow extraction and design decisions.

```
/model opus
```

## Figma MCP Setup

Requires the Figma MCP server:

```bash
claude mcp add figma -- npx -y @anthropic/claude-figma-mcp --figma-access-token=YOUR_TOKEN
```

Get your token: Figma → Settings → Personal access tokens → Generate.

Also requires the **figma-use** skill loaded before every `use_figma` call (from the official Figma plugin).

## Install

```
/plugin add [repo-url]
```
