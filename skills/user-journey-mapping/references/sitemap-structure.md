# Sitemap / Information Architecture Structure

A sitemap is a top-down hierarchical tree showing all pages and sections in a product. Each level represents navigation depth.

## Visual Style (Loveship Light Theme)

The sitemap uses the **Loveship light theme** — white background, dark text, SKY accent on section headers:
- **White background** (`#FFFFFF`) root frame
- **Section headers**: SKY_5 (`#F5FDFF`) fill + SKY_50 (`#009ECC`) 1.5px border + emoji icon + Semi Bold 15px SKY_70 text
- **Sub-section headers**: NIGHT100 (`#F5F6FA`) fill + NIGHT300 (`#E1E3EB`) 1px border + Semi Bold 13px NIGHT700 text
- **Child page nodes**: transparent fill + NIGHT400 (`#CED0DB`) 1px border + Medium 13px NIGHT800 text
- **External links**: gray NIGHT500 (`#ACAFBF`) text + " ↗" suffix
- **Page type annotations**: inline suffix text in NIGHT500, 10px Regular — NOT below the node
- **No design system icon imports** — use emoji for section headers (simpler, no library dependency)
- **No connectors** in vertical layout — auto-layout structure and indentation convey hierarchy

## Node Types

| Type | Level | Fill | Stroke | Corner Radius | Text |
|---|---|---|---|---|---|
| **Section Header** | 1 | `#F5FDFF` (SKY_5) | `#009ECC` (SKY_50), 1.5px | 8 | Emoji + 15px Semi Bold `#006B8A` (SKY_70) |
| **Sub-section** | 1.5 | `#F5F6FA` (NIGHT100) | `#E1E3EB` (NIGHT300), 1px | 5 | 13px Semi Bold `#474A59` (NIGHT700) |
| **Page** | 2+ | transparent | `#CED0DB` (NIGHT400), 1px | 5 | 13px Medium `#303240` (NIGHT800) + page type suffix 10px `#ACAFBF` |
| **External** | any | transparent | `#CED0DB` (NIGHT400), 1px | 5 | 13px Medium `#ACAFBF` (NIGHT500) + " ↗" |

## Page Type Annotations

Inline as a suffix in the same node row (NOT below the node — saves vertical space):

| Page Type | Label | Examples |
|---|---|---|
| Landing | `LANDING` | Home, marketing pages |
| Dashboard | `DASHBOARD` | Analytics, overview pages |
| List | `LIST` | Tables, search results, inventories |
| Detail | `DETAIL` | Item detail, profile, settings page |
| Form | `FORM` | Create/edit forms, wizards |
| Settings | `SETTINGS` | Configuration, preferences |
| Auth | `AUTH` | Login, signup, password reset |
| External | `EXTERNAL` | Links to other products/sites (also has ↗ in name) |

## Layout Rules

**3-column vertical layout (PROVEN — use for all sitemaps with >20 sections):**
- **Root frame**: non-auto-layout, white fill, initial 1800×6000, resize-to-fit at end
- **3 columns** at X positions: `COL1=60`, `COL2=540`, `COL3=1020`
- Each column is ~460px wide — sections fill available width via auto-layout HUG
- **Sections are positioned absolutely** within the root frame (grp.x, grp.y)
- **Balance sections across columns** — distribute roughly equal total height per column
- **Vertical spacing**: estimate each section's height (~40px per child + 60px header) before positioning the next section below

**Section group structure (PROVEN — copy exactly):**
```javascript
async function secHdr(parent, title, emoji, x, y) {
  var grp = figma.createAutoLayout("VERTICAL", { name: title, itemSpacing: 6 });
  grp.fills = [];
  grp.paddingBottom = 16;
  parent.appendChild(grp);
  grp.x = x; grp.y = y;

  var hdr = figma.createAutoLayout("HORIZONTAL", { name: "Hdr:" + title, itemSpacing: 8 });
  hdr.paddingLeft = 12; hdr.paddingRight = 16; hdr.paddingTop = 10; hdr.paddingBottom = 10;
  hdr.cornerRadius = 8;
  setFill(hdr, C.SKY_5);
  setStroke(hdr, C.SKY_50, 1.5);
  grp.appendChild(hdr);

  await makeText(hdr, emoji, 16, "Regular", C.SKY_50);
  await makeText(hdr, title, 15, "Semi Bold", C.SKY_70);
  return grp;
}
```

**Child page nodes (PROVEN — copy exactly):**
```javascript
async function ch(parent, name, pageType, indent, isExternal) {
  var row = figma.createAutoLayout("HORIZONTAL", { name: name, itemSpacing: 0 });
  row.fills = [];
  parent.appendChild(row);
  if (indent > 0) {
    var s = figma.createFrame();
    s.resize(indent, 1);
    s.fills = [];
    row.appendChild(s);
    s.layoutSizingVertical = "FIXED";
  }
  var nd = figma.createAutoLayout("HORIZONTAL", { name: name, itemSpacing: 6 });
  nd.paddingLeft = 10; nd.paddingRight = 10; nd.paddingTop = 5; nd.paddingBottom = 5;
  nd.cornerRadius = 5;
  setStroke(nd, C.NIGHT400, 1);
  nd.fills = [];
  row.appendChild(nd);
  await makeText(nd, isExternal ? name + " ↗" : name, 13, "Medium", isExternal ? C.NIGHT500 : C.NIGHT800);
  if (pageType) await makeText(nd, pageType, 10, "Regular", C.NIGHT500);
  return row;
}
```

**Sub-section headers (for nested groups like "Навушники та аксесуари"):**
```javascript
async function subSec(parent, name, indent) {
  var row = figma.createAutoLayout("HORIZONTAL", { name: name, itemSpacing: 0 });
  row.fills = [];
  parent.appendChild(row);
  if (indent > 0) {
    var s = figma.createFrame();
    s.resize(indent, 1);
    s.fills = [];
    row.appendChild(s);
    s.layoutSizingVertical = "FIXED";
  }
  var nd = figma.createAutoLayout("HORIZONTAL", { name: name, itemSpacing: 6 });
  nd.paddingLeft = 10; nd.paddingRight = 10; nd.paddingTop = 5; nd.paddingBottom = 5;
  nd.cornerRadius = 5;
  setFill(nd, C.NIGHT100);
  setStroke(nd, C.NIGHT300, 1);
  row.appendChild(nd);
  await makeText(nd, name, 13, "Semi Bold", C.NIGHT700);
  return row;
}
```

**Indentation levels:**
- **Level 1** (direct children of section): `indent = 16`
- **Level 2** (grandchildren, e.g., children of a sub-section): `indent = 32`
- **Level 3** (great-grandchildren): `indent = 48`

## Detailed by Default (CRITICAL — always produce detailed sitemaps)

**Sitemaps are ALWAYS detailed by default.** The user should never need to ask for "more detail" — the first render must already include 3+ levels of depth. A shallow sitemap that only shows the top navigation bar is a failure, not a deliverable.

**Minimum depth requirements:**
- **Level 1**: All top-level sections/categories from the navigation
- **Level 2**: All sub-pages within each section (discovered by navigating INTO each section)
- **Level 3**: Notable sub-sub-categories, tabs, and nested pages (discovered by exploring each L2 page)
- **External links**: All outbound links (documentation, support portals, partner sites)

**For e-commerce sites (like Rozetka, Amazon, eBay):**
- Map ALL top-level product categories from the mega-menu or category navigation
- Drill into each category to list its sub-categories (e.g., "Electronics → Laptops, Phones, TVs, Audio")
- Note category page types (LIST, FILTER, DETAIL)
- Map non-product sections: account, cart, wishlist, orders, help center, etc.

**For SaaS apps:**
- Explore every sidebar section, every dropdown in the top nav, every tab bar within pages
- Map settings pages and their sub-sections
- Map user account pages (profile, billing, API keys, etc.)

**Scoping for very large sites (>150 nodes):** If the full detailed tree would exceed ~150 nodes, propose a scope to the user BEFORE exploring: "The full site has ~200+ pages. Should I map everything, or focus on [key sections]?" Do NOT silently reduce depth. The user decides.

## Mandatory Drill-Down (CRITICAL — never skip)

**A sitemap with only top-level categories is NOT a sitemap — it's a navigation bar.** You MUST drill at least 2 levels deep into every section. If a category page shows sub-categories, those sub-categories MUST appear in the tree.

**Rules:**
1. **Navigate INTO every top-level section** to discover its sub-categories. Reading the sidebar alone is NOT enough — you must visit each category page.
2. **If the tree exceeds ~25 nodes, build section-by-section.** Render 3-6 sections per `use_figma` call (30-55 nodes — the proven sweet spot). Count rendered nodes after each batch. Continue until ALL sections are complete.
3. **Never silently omit sub-categories.** If a section page shows 8 sub-categories, all 8 must appear in the tree. If the total node count would exceed what fits in one frame, split into multiple frames or use the vertical layout.
4. **Present the COMPLETE tree** (with all levels) to the user for confirmation before rendering. The tree should show at minimum: Root → Sections (L1) → Sub-categories (L2) → Notable sub-sub-categories (L3).
5. **For large sites (>50 nodes):** Use the vertical section-based layout (see below). Build it section-by-section — one `use_figma` call per 2-3 sections. This prevents timeout and ensures completeness.
6. **Scoping:** If the full site has hundreds of pages, propose a scope to the user BEFORE exploring: "Should I map the full site, or focus on [top 6-8 sections]?" Don't silently skip categories.

**Build-section-by-section pattern (MANDATORY for all sitemaps with >15 nodes):**
```
Call 1: Create root frame + title + Section 1 (with all children) + Section 2
  → Save ROOT_ID. Report: "Rendered 2/8 sections (23/95 nodes)"
Call 2: Add Section 3 + Section 4
  → On success: Report: "Rendered 4/8 sections (47/95 nodes)"
  → On FAILURE: Retry Section 3 alone. If still fails, skip + log error. Continue to Section 4.
Call 3: Add Section 5 + Section 6
  → Report progress
Call 4: Add Section 7 + Section 8 + legend
Call 5: Draw connectors/spines between section headers
Call 6: Resize root frame to fit all content + final node count audit
```

Each call returns created node IDs. The next call references the root frame ID to append more sections.

**Error recovery rules (CRITICAL — never fail silently):**
- If a `use_figma` call **times out**: reduce batch size to 1 section per call, retry.
- If a `use_figma` call **throws an API error**: log the error, skip that section, continue with next.
- If a section has **too many child nodes** (>30): split that section into 2 calls (first half + second half of children).
- **After ALL sections are attempted**: run node count audit. Report any missing sections with error reasons.
- **Never present an incomplete sitemap as finished.** If sections were skipped, explicitly tell the user: "Sections X and Y could not be rendered because [reason]. The sitemap contains N of M expected nodes."
- **Progress reporting is mandatory.** After each batch call, report: "Rendered sections: [list]. Nodes: N/M. Remaining: [list]."

## Extraction from Sources

**Live URL (Playwright):**
1. Navigate to the app root
2. Read the sidebar/navigation to discover all top-level sections
3. **Navigate INTO each section page** to discover sub-categories — use `browser_evaluate` to extract category links from the page content
4. For sections with many sub-categories, also check for "Комп'ютерні комплектуючі" style grouped sub-sub-categories
5. Record the URL pattern for each page (e.g., `/products/:id/settings`)
6. Identify page types from URL patterns and content

**Codebase (routes):**
1. Read router config (routes.js, App.tsx, next.config.js)
2. Extract all route paths
3. Group by path segments (e.g., `/portal/products/...` = Products section)
4. Identify nested routes = parent-child relationships
5. Check for auth guards to mark protected sections

**Verbal / PRD:**
1. Ask: "What are the main sections of your product?"
2. For each section: "What pages are inside?"
3. For each page: "What type is it?" (dashboard, form, list, detail)
4. Build the tree from the answers

## Output Format (before rendering)

Present as an indented tree for confirmation:

```
Home (LANDING)
├── Products (SECTION)
│   ├── Product List (LIST)
│   ├── Product Detail (DETAIL)
│   │   ├── Overview (DASHBOARD)
│   │   ├── Settings (SETTINGS)
│   │   └── Analytics (DASHBOARD)
│   └── Create Product (FORM)
├── Organization (SECTION)
│   ├── Profile (DETAIL)
│   ├── Members (LIST)
│   └── Billing (SETTINGS)
├── Support (EXTERNAL ↗)
└── Documentation (EXTERNAL ↗)
```

Confirm with user before rendering.

## Optimal Batch Sizing (PROVEN)

**3-6 sections per `use_figma` call (30-55 nodes).** This is the proven sweet spot:
- Too few sections per call = too many API calls, slow and hits rate limits
- Too many sections per call = script too long, risk of timeout

**Proven batch pattern from Rozetka (24 sections, 164 nodes, 4 calls):**
```
Call 1: Root frame + title + 3 sections (35 nodes) — includes helpers, palette, font loading
Call 2: 6 sections (44 nodes) — reuses helpers pattern
Call 3: 7 sections (55 nodes) — largest batch, still fast
Call 4: 8 sections + resize-to-fit (30 nodes) — final batch with audit
```

## Build Order (PROVEN — follow exactly)

### Call 1: Root frame + title + first 3 sections

Every `use_figma` call MUST start with the full helper set (palette, setFill, setStroke, makeText, secHdr, ch, subSec). Paste these into every call:

```javascript
// === MANDATORY PREAMBLE — paste into every use_figma call ===
function h(hex){return{r:parseInt(hex.slice(1,3),16)/255,g:parseInt(hex.slice(3,5),16)/255,b:parseInt(hex.slice(5,7),16)/255};}
var C={WHITE:"#FFFFFF",NIGHT50:"#FAFAFC",NIGHT100:"#F5F6FA",NIGHT200:"#EBEDF5",NIGHT300:"#E1E3EB",NIGHT400:"#CED0DB",NIGHT500:"#ACAFBF",NIGHT600:"#6C6F80",NIGHT700:"#474A59",NIGHT800:"#303240",SKY_5:"#F5FDFF",SKY_50:"#009ECC",SKY_70:"#006B8A"};
function setFill(n,hex){n.fills=[{type:"SOLID",color:h(hex)}];}
function setStroke(n,hex,w){n.strokes=[{type:"SOLID",color:h(hex)}];n.strokeWeight=w||1;}
await figma.loadFontAsync({family:"Inter",style:"Regular"});
await figma.loadFontAsync({family:"Inter",style:"Semi Bold"});
await figma.loadFontAsync({family:"Inter",style:"Bold"});
await figma.loadFontAsync({family:"Inter",style:"Medium"});
// ... then paste secHdr(), ch(), subSec(), makeText() from Layout Rules above
```

Then create the root frame and first sections:
```javascript
var root = figma.createFrame();
root.name = "Product Name — Sitemap";
root.resize(1800, 6000);
setFill(root, C.WHITE);
root.clipsContent = false;

// Title
var tf = figma.createAutoLayout("VERTICAL", { name: "Title", itemSpacing: 6 });
tf.fills = [];
root.appendChild(tf);
tf.x = 60; tf.y = 40;
await makeText(tf, "Product Name — Sitemap", 30, "Bold", C.NIGHT800);
await makeText(tf, "domain.com  •  Type  •  ~N pages  •  3-level depth", 14, "Regular", C.NIGHT600);

// Divider
var div = figma.createRectangle();
div.resize(1680, 1);
setFill(div, C.NIGHT300);
root.appendChild(div);
div.x = 60; div.y = 110;

// First 3 sections at COL1_X=60, COL2_X=540, COL3_X=1020
// ... use secHdr() and ch() to build each section

return { rootId: root.id, sections: 3, nodes: N };
```

### Calls 2-N: Append remaining sections

```javascript
var root = await figma.getNodeByIdAsync("ROOT_ID_FROM_CALL_1");
// ... paste helpers again
// ... build 3-6 more sections with secHdr() and ch()
return { sections: totalSoFar, nodesAdded: N, totalNodes: M };
```

### Final call: Resize root frame to fit content

```javascript
var root = await figma.getNodeByIdAsync("ROOT_ID");
var minX=Infinity,minY=Infinity,maxX=-Infinity,maxY=-Infinity;
for(var child of root.children){
  minX=Math.min(minX,child.x); minY=Math.min(minY,child.y);
  maxX=Math.max(maxX,child.x+child.width); maxY=Math.max(maxY,child.y+child.height);
}
var PAD=60;
var shiftX=PAD-minX, shiftY=PAD-minY;
for(var child of root.children){child.x+=shiftX;child.y+=shiftY;}
root.resize(maxX-minX+PAD*2, maxY-minY+PAD*2);
```

### Post-render audit (MANDATORY)

```javascript
var root = await figma.getNodeByIdAsync("ROOT_ID");
var sections = root.children.filter(n => n.type === "FRAME" && n.layoutMode === "VERTICAL" && n.name !== "Title");
return { sectionCount: sections.length, sectionNames: sections.map(s => s.name) };
```

Compare section count and names to the confirmed tree. Report any missing sections.

## Legend

Legend is **optional** — the visual hierarchy (section headers with emoji + accent styling vs indented child nodes) is self-explanatory. If included, keep it minimal.
