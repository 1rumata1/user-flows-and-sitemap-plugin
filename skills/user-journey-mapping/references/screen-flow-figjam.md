# Screen Flow Wireflow (FigJam)

Screen Flow diagrams are **wireflows** — actual screenshots of each screen placed on a FigJam canvas, connected by arrows showing navigation paths, and annotated with sticky notes for decisions, errors, and context.

**Screenshots are REQUIRED.** This diagram type captures the real UI, not abstract boxes. The user must provide a live URL (for Playwright capture) or screenshot files.

**No design system tokens. No `bindColor()`. No `makeText()`.** FigJam uses its own styling. But it DOES use `create_new_file`, `upload_assets`, and `use_figma` (with the `figma-use-figjam` skill).

## When to Use Screen Flow vs User Flow

| Aspect | User Flow (Option 1) | Screen Flow (Option 4) |
|---|---|---|
| **Output** | Figma Design file | FigJam file |
| **Content** | Abstract nodes (rectangles, pills) | Actual screenshots of screens |
| **Tool** | `use_figma` + design system tokens | `use_figma` on FigJam + `upload_assets` |
| **Styling** | Design system token binding | FigJam defaults (no design system) |
| **Connectors** | Vector lines with arrow caps | FigJam native Connectors |
| **Annotations** | Text labels | FigJam Sticky notes |
| **Best for** | Production documentation, handoff | Visual walkthroughs, stakeholder demos, team alignment |
| **Editable by** | Figma Design users | Anyone with FigJam access |

**Rule of thumb:** If the user wants to show what the app actually looks like → Screen Flow. If they want an abstract diagram with design system styling → User Flow.

## Layout

```
 ┌─── Phase 1: Entry ──────────────┐  ┌─── Phase 2: Creation ─────────────────────────┐
 │                                  │  │                                                │
 │  ┌──────────┐    ┌──────────┐   │  │  ┌──────────┐    ┌──────────┐    ┌──────────┐  │
 │  │Screenshot│───>│Screenshot│───>│──│─>│Screenshot│───>│Screenshot│───>│Screenshot│  │
 │  │  Landing │    │   List   │   │  │  │  Step 1  │    │  Step 2  │    │  Success │  │
 │  └──────────┘    └──────────┘   │  │  └──────────┘    └──────────┘    └──────────┘  │
 │       │           [Sticky:      │  │       │                               │         │
 │  [Sticky:          "Clicks      │  │  [Sticky:                        [Sticky:      │
 │   "Entry"]         Create"]     │  │   "Fills form"]                   "Done!"]     │
 │                                  │  │                                                │
 └──────────────────────────────────┘  └────────────────────────────────────────────────┘
```

**Sizing:**
- Screenshots: **preserve original aspect ratio**. `upload_assets` creates frames at a default size (often 400x300) which does NOT match the screenshot dimensions. You MUST resize frames to match the actual image aspect ratio.
- **Before uploading**, check the actual pixel dimensions of the screenshots (e.g. via python `struct.unpack` on PNG header or `sips -g pixelWidth -g pixelHeight`). Typical browser screenshots are 1728x963 (ratio ~1.79).
- Resize frames to: `W = 520`, `H = round(520 * (imgHeight / imgWidth))`. This scales proportionally without cropping.
- `scaleMode: "FILL"` with correct aspect ratio = no cropping. If the ratio is wrong, FILL will crop. Always match the ratio exactly.
- Horizontal gap between screenshots: 140px
- Vertical gap between rows (for branching): **screenshot height + sticky gap + sticky height + breathing room**. FigJam stickies are **240px tall** by default. Formula: `BRANCH_GAP = H + 20 + 240 + 60` (~610px for 290px screenshots). This prevents stickies from overlapping the next row's screenshot. Always measure actual sticky height — don't assume.
- Stickies: placed 20px below their corresponding screenshot, centered horizontally (`sticky.x = node.x + W/2 - 75`)
- Sections: optional grouping frames for phases

**Branch coverage:**
- When a flow has decision points (dropdowns, modals, conditional routing), capture screenshots for **EVERY branch** — not just the happy path.
- For each decision, walk through each option and capture the resulting screen.
- Present the complete set to the user before building. If a branch is skipped, the wireflow is incomplete.

## FigJam Node Types

| Node Type | Figma API | Use |
|---|---|---|
| **Screenshot frame** | `upload_assets` → creates frame with image fill | The actual screen capture |
| **Connector** | `figma.createConnector()` | Arrow linking two screenshots |
| **Sticky** | `figma.createSticky()` | Annotation (screen label, decision note, error description) |
| **ShapeWithText** | `figma.createShapeWithText()` | Decision diamonds, action labels |
| **Section** | `figma.createSection()` | Phase grouping container |

## Connector Patterns

```javascript
// Connect two screenshot frames with an arrow
var connector = figma.createConnector();
connector.connectorStart = { endpointNodeId: screen1.id, magnet: "RIGHT" };
connector.connectorEnd = { endpointNodeId: screen2.id, magnet: "LEFT" };
connector.connectorLineType = "ELBOWED";
// Connector text label (navigation trigger)
await figma.loadFontAsync({ family: "Inter", style: "Medium" });
connector.text.characters = "Clicks Create";
```

**Connector label conventions (MUST start with an action verb):**
- Navigation trigger: "Clicks Login button", "Submits registration form", "Selects Items tab"
- Dropdown/menu: "Clicks Create item dropdown", "Selects 'With Metric' option"
- Hover: "Hovers over user avatar", "Hovers to reveal tooltip"
- Conditional: "If admin role", "If authenticated"
- Back/cancel: "Clicks Cancel", "Clicks Back button" (use different connector stroke style if possible)
- Skip/optional: "Clicks Skip"
- System-initiated: "Redirected to dashboard", "Modal appears", "Page auto-refreshes"
- **Never** use bare nouns like "Create", "Next", "Login" — always include the action verb.

**Preventing connector overlap with screenshots:**
- FigJam connectors with `magnet: "AUTO"` can route THROUGH other nodes. This is a common problem when branching vertically — the connector from a decision to a lower branch may cut through an upper branch's screenshot.
- **Fix 1**: Use explicit magnets — `magnet: "RIGHT"` on source, `magnet: "LEFT"` on target — to force connectors to route horizontally first, then vertically.
- **Fix 2**: Add horizontal offset — position branch screenshots further right from the decision point, giving connectors room to route around other nodes.
- **Fix 3**: Use `connectorLineType: "STRAIGHT"` for branches that would otherwise elbow through other nodes.
- **Always visually verify** after creating connectors that no lines cut through screenshots.

## Sticky Note Conventions

```javascript
// Screen label sticky (placed below screenshot) — MUST include detailed description
var sticky = figma.createSticky();
sticky.text.characters = "Items List — Displays all configured items with visibility status, icons, and descriptions. User clicks 'Create item' to start the creation flow.";
sticky.x = screenshot.x;
sticky.y = screenshot.y + screenshot.height + 16;
// Let sticky grow to fit text — do NOT assume fixed dimensions

// Decision annotation (placed near a branching point) — list ALL options with consequences
var decision = figma.createSticky();
decision.text.characters = "Which creation type?\n• With Metric — Opens 2-step wizard: select metric threshold first, then fill item details\n• Without Metric — Opens single-step form: item is unlocked by server API call\n• Bulk Import — Upload a ZIP file containing multiple items at once";
decision.authorVisible = false;
```

**Sticky text rules:**
- **Never truncate.** Let stickies grow to fit their full text content. After setting `sticky.text.characters`, measure the actual `sticky.width` and `sticky.height` before positioning adjacent elements.
- **Format**: "Screen Name — Detailed description of what happens here and what the user does."
- **Decision stickies**: List ALL options with a brief explanation of what each option leads to.

**Sticky colors (FigJam defaults):**
- Yellow: screen labels and general annotations
- Blue: decisions and branching points
- Green: success states
- Red: error states and failure paths
- Gray: context notes

## Build Order

### Step 0: Confirm starting point with the user

**Before capturing any screenshots**, proactively suggest a starting point:
- Analyze the target URL and feature to propose logical options (e.g., "Login page", "Dashboard", "Feature page directly").
- Default to the earliest meaningful entry point.
- Ask the user to confirm. Do NOT start capturing until the user agrees on the starting screen.

### Step 1: Capture screenshots

Use the existing Playwright capture workflow (same as Options 1-3):
1. Launch headed Chrome via Playwright
2. Navigate to the **user-confirmed** starting URL
3. At EACH screen, BEFORE moving on:
   - **Hover** over interactive elements (buttons, menus, avatars) to reveal hidden options
   - **Click every dropdown/menu** and capture the open state as a separate screenshot
   - **Expand collapsible sections** (chevrons, "..." buttons)
   - **Check for alternative paths**: if a dropdown has N options, that's N branches to explore
4. Capture each distinct screen state as a numbered screenshot
5. Deduplicate by file size — keep only visually distinct captures
6. Name each screenshot by page/action (e.g., "01-items-list.png", "02-create-dropdown-open.png")
7. If any screen/element is **inaccessible** (auth wall, file upload required, shadow DOM, iframe, insufficient permissions), note it for the capture summary — NEVER silently skip it.

**If the user provides screenshot files instead of a URL**, skip Playwright and use the provided images directly.

### Step 1.5: Present capture summary to the user

**MANDATORY before building.** Present a structured summary:
- **Captured screens**: numbered list with descriptions
- **Flow paths**: main path + all branches
- **Branches explored**: which decision options were captured
- **NOT captured / inaccessible**: explicitly list what was skipped and why
- **Suggested follow-ups**: propose additional captures if relevant

Wait for user confirmation before proceeding to Step 2.

### Step 2: Create FigJam file

```javascript
create_new_file({
  fileName: "{Flow Name} — Screen Flow",
  planKey: PLAN_KEY,        // from whoami
  editorType: "figjam"      // CRITICAL: figjam, not design
})
```

### Step 3: Upload screenshots

**IMPORTANT: Use the `figma` MCP server's `upload_assets`, NOT `plugin:figma`'s version.** The `plugin:figma` version fails silently on FigJam files. If you have two Figma MCP servers available (e.g. `mcp__figma__upload_assets` and `mcp__plugin_figma_figma__upload_assets`), use the one that returns upload URLs successfully.

```javascript
// Request upload URLs (one per screenshot, max 5 per call)
upload_assets({
  fileKey: FILE_KEY,
  count: N    // number of screenshots (1-5 per call, max 4 recommended)
})
// Returns submitUrl for each slot

// POST each screenshot to its submitUrl via curl
// curl -s -X POST -F "file=@01-items-list.png" SUBMIT_URL_1
// curl -s -X POST -F "file=@02-create-menu.png" SUBMIT_URL_2
// ... etc

// Each POST returns: { imageHash, placedOnNodeId }
// The placedOnNodeId is the frame created on the FigJam canvas
```

Each uploaded image becomes a frame with an image fill on the current page. Save the `placedOnNodeId` values — you'll use these to position frames and create connectors in the next step.

**Known issue:** If `upload_assets` returns an error or empty response on a FigJam file, try the other Figma MCP server. The two servers (`figma` and `plugin:figma`) have different FigJam compatibility.

### Step 4: Load FigJam skill

Load the `figma-use-figjam` skill before calling `use_figma`. This provides FigJam-specific API context (Connector, Sticky, ShapeWithText, Section).

Also load `figma-use` for general Plugin API rules.

### Step 5: Position screenshots and create connectors

Use `use_figma` on the FigJam file. Pass the `placedOnNodeId` values from the upload response as the screenshot frame IDs.

```javascript
await figma.loadFontAsync({ family: "Inter", style: "Medium" });

// IDs from upload_assets responses (placedOnNodeId values)
var screenIds = ["5:26", "6:26", "7:26", "8:26"];
var screenNames = ["Items List", "Create Menu", "Step 1: Select Metric", "Item Details"];
var connLabels = ["Clicks Create", "Selects With Metric", "Clicks Next"];

var MAX_W = 480;
var GAP = 150;

// 1. Scale and position uploaded frames left-to-right
var X = 0;
for (var i = 0; i < screenIds.length; i++) {
  var node = figma.getNodeById(screenIds[i]);
  if (node) {
    // Preserve aspect ratio — scale to max-width
    var origW = node.width;
    var origH = node.height;
    var scale = Math.min(MAX_W / origW, 1); // don't upscale
    var newW = origW * scale;
    var newH = origH * scale;
    node.resize(newW, newH);
    node.x = X;
    node.y = 100;
    node.name = screenNames[i];
    X += newW + GAP;
  }
}

// 2. Create connectors between adjacent screenshots
for (var i = 0; i < screenIds.length - 1; i++) {
  var conn = figma.createConnector();
  conn.connectorStart = { endpointNodeId: screenIds[i], magnet: "AUTO" };
  conn.connectorEnd = { endpointNodeId: screenIds[i + 1], magnet: "AUTO" };
  conn.connectorLineType = "ELBOWED";
  conn.text.characters = connLabels[i];
}

// 3. Add detailed sticky labels below each screenshot
for (var i = 0; i < screenIds.length; i++) {
  var sticky = figma.createSticky();
  sticky.text.characters = screenDescriptions[i]; // Full description, NOT just screen name
  var node = figma.getNodeById(screenIds[i]);
  if (node) {
    sticky.x = node.x;
    sticky.y = node.y + node.height + 20;
    // Let sticky grow — measure AFTER setting text, don't assume fixed size
  }
}

// 4. Title — prominent ShapeWithText banner, NOT a regular sticky
await figma.loadFontAsync({ family: "Inter", style: "Bold" });
var title = figma.createShapeWithText();
title.shapeType = "ROUNDED_RECTANGLE";
title.text.characters = "Item Creation — Screen Flow";
title.text.fontSize = 32;
title.resize(800, 80);
title.x = 0;
title.y = -120; // Above the flow with clear separation
```

### Step 6: Add annotations for decisions and branching

For screens with multiple paths (decision points):
- Create a ShapeWithText diamond or a blue sticky with the question
- Create additional connectors to branch targets
- Position branch screenshots below the main flow row

### Step 7: Add "Not Captured" and "Follow-ups" sections

At the bottom of the FigJam canvas (below all screenshots), add two annotation sections.

**CRITICAL SPACING RULE:** FigJam stickies **cannot be resized** (`sticky.resize()` throws an error). They auto-size to fit text content, typically **200-280px tall** for multi-sentence descriptions. You MUST reserve at least **400px** vertical gap between the "Not Captured" section header and the "Suggested Follow-ups" section header. Using less (e.g. 220px) causes follow-ups to overlap Not Captured stickies.

**Spacing formula:**
- `NC_HEADER_Y = bottomY + 100` (100px below last sticky in the flow)
- `NC_STICKIES_Y = NC_HEADER_Y + 70` (below Not Captured header)
- `FU_HEADER_Y = NC_HEADER_Y + 400` (400px gap — accounts for header + stickies + breathing room)
- `FU_STICKIES_Y = FU_HEADER_Y + 70` (below Follow-ups header)

**Horizontal layout:** Place stickies side-by-side with **420px horizontal gap** between them (stickies auto-size to ~200px wide, so 420px gives ~220px gap between edges).

```javascript
await figma.loadFontAsync({ family: "Inter", style: "Bold" });
await figma.loadFontAsync({ family: "Inter", style: "Medium" });

// Spacing constants
var NC_HEADER_Y = bottomY + 100;
var NC_STICKIES_Y = NC_HEADER_Y + 70;
var FU_HEADER_Y = NC_HEADER_Y + 400;  // 400px gap — NOT less!
var FU_STICKIES_Y = FU_HEADER_Y + 70;

// --- "Not Captured / Inaccessible" section ---
var ncHeader = figma.createShapeWithText();
ncHeader.shapeType = "ROUNDED_RECTANGLE";
await figma.loadFontAsync(ncHeader.text.fontName);
ncHeader.text.characters = "NOT Captured / Inaccessible";
ncHeader.text.fontSize = 16;
ncHeader.resize(400, 50);
ncHeader.x = 0;
ncHeader.y = NC_HEADER_Y;
ncHeader.fills = [{ type: "SOLID", color: { r: 253/255, g: 225/255, b: 225/255 } }]; // fire-10ish

// One red-tinted sticky per item
var ncItems = [
  "SMS verification — requires real phone number to receive OTP code.",
  "Order confirmation — requires completing payment with real method."
];
for (var i = 0; i < ncItems.length; i++) {
  var s = figma.createSticky();
  await figma.loadFontAsync(s.text.fontName);
  s.text.characters = ncItems[i];
  s.fills = [{ type: "SOLID", color: { r: 253/255, g: 225/255, b: 225/255 } }];
  s.x = i * 420;  // 420px apart horizontally
  s.y = NC_STICKIES_Y;
}

// --- "Suggested Follow-ups" section ---
var fuHeader = figma.createShapeWithText();
fuHeader.shapeType = "ROUNDED_RECTANGLE";
await figma.loadFontAsync(fuHeader.text.fontName);
fuHeader.text.characters = "Suggested Follow-ups";
fuHeader.text.fontSize = 16;
fuHeader.resize(400, 50);
fuHeader.x = 0;
fuHeader.y = FU_HEADER_Y;
fuHeader.fills = [{ type: "SOLID", color: { r: 235/255, g: 243/255, b: 216/255 } }]; // grass-10ish

var fuItems = [
  "Capture validation error states on empty form submission.",
  "Capture promo code flow — enter invalid then valid code.",
  "Capture installment payment flow with bank partner selection."
];
for (var i = 0; i < fuItems.length; i++) {
  var s = figma.createSticky();
  await figma.loadFontAsync(s.text.fontName);
  s.text.characters = fuItems[i];
  s.fills = [{ type: "SOLID", color: { r: 235/255, g: 243/255, b: 216/255 } }];
  s.x = i * 420;
  s.y = FU_STICKIES_Y;
}
```

### Step 8: Optional — Group into Sections by phase

```javascript
var section = figma.createSection();
section.name = "Phase 1: Entry";
// Resize to contain the phase's screenshots
section.resizeWithoutConstraints(totalWidth, totalHeight);
// Move screenshots into the section
section.appendChild(screenshot1);
section.appendChild(screenshot2);
```

## Branching Layout

When the flow branches (e.g., creation type dropdown with 3 options), arrange branches vertically:

```
                          ┌──────────┐
               ┌─────────>│With Metric│──────> ...
               │          └──────────┘
┌──────────┐   │          ┌──────────┐
│  Create  │───┼─────────>│Without   │──────> ...
│  Menu    │   │          │Metric    │
└──────────┘   │          └──────────┘
               │          ┌──────────┐
               └─────────>│  Bulk    │──────> ...
                          │ Import   │
                          └──────────┘
```

- Main path continues horizontally (left-to-right)
- Branch alternatives stack vertically with 120px gap
- Each branch connector gets a label ("With Metric", "Without Metric", "Bulk Import")
- Branches that reconverge get a connector back to the merge point

## Workflow Differences from Figma Design Types

| Step | Options 1–3 (Figma Design) | Option 4 (Screen Flow / FigJam) |
|---|---|---|
| **Step 2: Screenshots** | Optional | **REQUIRED** — core of the diagram |
| **Step 5: File creation** | `create_new_file({ editorType: "design" })` | `create_new_file({ editorType: "figjam" })` |
| **Step 5: Skill loading** | Load `figma-use` skill | Load `figma-use` AND `figma-use-figjam` skills |
| **Step 5: Styling** | `setFill()`, `setStroke()`, `makeText()` with Loveship palette | **None** — FigJam uses its own styling |
| **Step 5: Images** | `upload_assets` for thumbnails (200x125px) | `upload_assets` for full screenshots |
| **Step 5: Connectors** | `createVector()` with arrow caps | `createConnector()` with text labels |
| **Step 5: Annotations** | Text nodes with Loveship palette colors | `createSticky()` notes |
| **Step 6: Validation** | Structural + visual audit (node count, screenshot) | **Visual check only** — all screenshots connected |

**Do NOT use for Screen Flow:**
- `setFill()`, `setStroke()`, or Loveship palette constants
- `makeText()` with font/color parameters
- `figma.loadFontAsync()` for design system fonts (only load if setting connector/sticky text)
- Structural validation audit code

## Extraction Guidance

Screen Flows need screenshots + navigation paths:

1. **Screenshots** — capture every distinct screen in the flow (REQUIRED)
2. **Screen names** — short label for each screenshot
3. **Navigation triggers** — what action moves from screen A to screen B
4. **Decision points** — where does the flow branch? what are the options?
5. **Error/failure paths** — what happens when something goes wrong?
6. **Back/cancel paths** — where does "Back" or "Cancel" go?

**Extract as a screen list + paths:**

```
Screen Flow: Item Creation

Screenshots captured:
  01. Items List
  02. Create Menu (dropdown)
  03. Step 1: Select Metric (modal)
  04. Step 2: Item Details (modal)
  05. Item Detail (view)
  06. Edit Item

Paths:
  01 --Clicks Create--> 02
  02 --With Metric--> 03
  02 --Without Metric--> 04
  02 --Bulk Import--> [separate flow]
  03 --Next--> 04
  03 -.Cancel.-> 01
  04 --Save--> 05
  04 -.Cancel.-> 01
  01 --Clicks Row--> 05
  05 --Edit--> 06
  05 -.Back.-> 01
  06 --Save--> 05
  06 -.Cancel.-> 05
```

Confirm with user before building.
