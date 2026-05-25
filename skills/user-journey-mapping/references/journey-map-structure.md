# Journey Map Structure

A journey map is a flat table grid. Columns are phases, rows are experience dimensions. Every cell is a plain rectangle with text.

## Grid Layout

```
           │ Awareness │ Consideration │ Decision │ Onboarding │ Usage │ Advocacy │
───────────┼───────────┼───────────────┼──────────┼────────────┼───────┼──────────┤
Persona    │                    [spans full width]                                │
───────────┼───────────┼───────────────┼──────────┼────────────┼───────┼──────────┤
Screenshots│ [thumb]   │ [thumb]       │ [thumb]  │ [thumb]    │[thumb]│ [thumb]  │
───────────┼───────────┼───────────────┼──────────┼────────────┼───────┼──────────┤
Actions    │ Searches  │ Compares      │ Signs up │ Tutorial   │ Uses  │ Shares   │
───────────┼───────────┼───────────────┼──────────┼────────────┼───────┼──────────┤
Touchpoints│ Google    │ Website       │ Signup   │ Email      │ App   │ Social   │
───────────┼───────────┼───────────────┼──────────┼────────────┼───────┼──────────┤
Thoughts   │ "Is there │ "How does it  │ "Will it │ "Where do  │ "This │ "My team │
           │ better?"  │  compare?"    │  work?"  │  I start?" │ works"│ needs it"│
───────────┼───────────┼───────────────┼──────────┼────────────┼───────┼──────────┤
Emotions   │    😐     │      🤔       │    😟    │     😊     │  😀   │    🎉    │
───────────┼───────────┼───────────────┼──────────┼────────────┼───────┼──────────┤
Pain Points│ Too many  │ Hard to       │ Pricing  │ Steep      │ Missing│ No easy  │
           │ options   │ compare       │ unclear  │ curve      │feature│ referral │
───────────┼───────────┼───────────────┼──────────┼────────────┼───────┼──────────┤
Opportun.  │ SEO       │ Comparison    │ Clear    │ Quick      │Feature│ Referral │
           │ content   │ page          │ pricing  │ start      │roadmap│ program  │
```

## Row Label Structure (CRITICAL — follow exactly)

Every row has a label cell on the left. The label cell uses **HORIZONTAL auto-layout** so the accent bar automatically stretches to the full row height.

```
Label Frame (HORIZONTAL auto-layout, FIXED width 160px, height: FILL from row, gap: 0, padding: 0)
  ├── Accent Bar (FRAME, FIXED width 3px, height: FILL, fills: sky-50, NO auto-layout)
  └── Text Container (VERTICAL auto-layout, width: FILL, height: FILL, centerY)
      └── Label Text (TEXT, 12px Semibold, night800)
```

### Code pattern for label frame:

```javascript
async function buildLabel(parent, text) {
  // Label frame — HORIZONTAL auto-layout so bar stretches to full height
  var lf = figma.createAutoLayout("HORIZONTAL", { name: "Label: " + text, itemSpacing: 0 });
  lf.resize(160, 80);
  lf.fills = [];
  lf.paddingTop = 0; lf.paddingBottom = 0; lf.paddingLeft = 0; lf.paddingRight = 0;
  parent.appendChild(lf);
  lf.layoutSizingVertical = "FILL";  // stretches with row height

  // Accent bar — FIXED width, FILL height (auto-stretches)
  var bar = figma.createFrame();
  bar.name = "Accent Bar";
  bar.resize(3, 80);
  bar.fills = []; // set via setFill below
  bar.layoutMode = "NONE";
  lf.appendChild(bar);
  bar.layoutSizingVertical = "FILL";  // KEY: bar takes full parent height
  setFill(bar, C.SKY_50);

  // Text container — vertically centered
  var tc = figma.createAutoLayout("VERTICAL", { name: "Text Container" });
  tc.fills = [];
  tc.paddingLeft = 16; tc.paddingRight = 12;
  tc.primaryAxisAlignItems = "CENTER";  // vertical center
  lf.appendChild(tc);
  tc.layoutSizingHorizontal = "FILL";
  tc.layoutSizingVertical = "FILL";

  var lt = await makeText(tc, text.toUpperCase(), 12, "Semibold", C.NIGHT800);
  return lf;
}
```

**Rules:**
1. The accent bar MUST use `layoutSizingVertical: "FILL"` inside a HORIZONTAL auto-layout parent. This ensures the bar stretches to the full row height regardless of how tall the content cells grow. **NEVER use absolute positioning (`layoutMode: "NONE"`) on the label frame itself** — this was the old approach and causes the bar to stay at a fixed height.
2. The text container uses VERTICAL auto-layout with `primaryAxisAlignItems: "CENTER"` for vertical centering. The text is always vertically centered within the full row height.
3. The label frame has NO fills, NO strokes — transparent background with just the bar and text.
4. The label frame uses `layoutSizingVertical: "FILL"` from the parent row, so it matches the tallest content cell.
5. NEVER put severity indicators (MINOR/MAJOR/CRITICAL dots or text) inside a row label frame OR as direct children of the row frame. Severity belongs ONLY inside phase content cells.
6. NEVER put row label TEXT as a direct child of the row frame. Label text belongs ONLY inside the label frame's text container.

## Row Frame Structure

Each row is a horizontal container:

```
Row Frame (HORIZONTAL auto-layout, itemSpacing: 1, no fills, no strokes)
  ├── Label Frame (FIXED 160px, FILL height)
  ├── Phase 1 Cell (FILL width, FILL height)
  ├── Phase 2 Cell (FILL width, FILL height)
  ├── Phase 3 Cell (FILL width, FILL height)
  ├── Phase 4 Cell (FILL width, FILL height)
  └── Phase 5 Cell (FILL width, FILL height)
```

**Rules:**
1. Row frames have `itemSpacing: 1` (1px gap between columns — acts as grid lines).
2. Row frames have NO fills and NO strokes — transparent.
3. Content cells use `setFill(cell, C.WHITE)` with `setStroke(cell, C.NIGHT200)` for subtle borders.
4. Content cells use VERTICAL auto-layout with `primaryAxisAlignItems: "MIN"` (top-aligned) and consistent padding: `paddingTop: 16, paddingBottom: 16, paddingLeft: 16, paddingRight: 16`.
5. All content cells use `layoutSizingVertical: "FILL"` so all cells in a row match the tallest cell's height.
6. Text in content cells has `layoutSizingHorizontal: "FILL"` to wrap properly within the cell.

---

## Rows Explained

**Persona** (header, full width): Name, role, goal. One sentence each.

**Screenshots** (MANDATORY when available): If screenshots were captured during Step 2 (via live URL, video, or provided files), this row MUST appear directly below the phase headers and above the Actions row. One thumbnail per phase column showing the actual screen the user sees at that stage.

Screenshot cell structure:
```
Cell Frame (VERTICAL auto-layout, itemSpacing: 8, padding: 12px)
  ├── Image Frame (FILL width, corner radius 4px, image fill from upload_assets)
  └── Caption Text (12px Semibold, night600, centered)
```

Rules:
1. Cell fill: `setFill(cell, C.WHITE)` with `setStroke(cell, C.NIGHT200)` — same as other content cells.
2. Image: uploaded via `upload_assets`, resized to fill cell width. Use `cornerRadius: 4` on the image frame. Set `layoutSizingHorizontal: "FILL"` so the image scales with the column.
3. Caption: short screen name below the image (e.g., "Homepage", "Search Results", "Product Detail"). Use 12px Semibold in `night600` (`#6C6F80`).
4. Row height: taller than other rows (~170px) to accommodate the thumbnail + caption.
5. The Screenshots row label follows the same Label Frame structure as all other rows (accent bar + "SCREENSHOTS" text, using the auto-layout label pattern).
6. If screenshots were NOT captured (user chose "No" in Step 2), omit this row entirely — do NOT leave an empty placeholder.
7. Map each screenshot to its corresponding phase. If a phase has no matching screenshot, leave that cell with a "No screenshot" caption in `night500` (`#ACAFBF`) instead of an image.

**Actions**: Short verb phrases — what the user does in each phase. "Reads reviews", "Fills form", "Invites team".

**Touchpoints**: Where the interaction happens — website, app, email, social, support, in-person.

**Thoughts**: First-person quotes. "Is this worth the price?", "Where do I start?"

**Emotions**: Single emoji per phase. 😀 positive, 😐 neutral, 😟 negative, 🤔 uncertain, 😤 frustrated, 🎉 delighted. Just text — no charts or curves.

**Pain Points**: Each pain point is a **single horizontal row** with severity indicator inline. **Severity indicators belong ONLY in the content cells — NEVER in the row label column.**

```
[dot 8px] [SEVERITY label] [description text]
```

Layout: HORIZONTAL auto-layout, 8px gap, vertically centered. Example:
- `🔴 CRITICAL  Stats redirect breaks flow completely`
- `🟠 MAJOR  No icon size/format requirements shown`
- `🟡 MINOR  Wrong landing page after agreement`

NEVER stack the dot, label, and description vertically. They must be on ONE LINE. The frame uses `layoutMode: "HORIZONTAL"`, `itemSpacing: 8`, `counterAxisAlignItems: "CENTER"`.

Severity levels:
- **CRITICAL** (red dot) — blocks task completion, no workaround
- **MAJOR** (orange dot) — significant friction, users may fail
- **MINOR** (yellow dot) — creates friction but doesn't prevent completion

**Opportunities**: Actionable improvements with green arrow prefix. "→ Add inline tier comparison on pricing page."

## Phases

Not all maps need all phases. Common sets:

- **SaaS**: Awareness → Consideration → Decision → Onboarding → Usage → Retention → Advocacy
- **E-commerce**: Discovery → Browse → Cart → Checkout → Delivery → Post-purchase
- **Feature-specific**: Discovery → First Use → Core Loop → Power Features → Team Expansion

Pick phases that match the user's scenario. 4–7 phases is the sweet spot.

## Sizing

- Total width: 1600–2400px depending on phase count
- Column width: 220–320px (equal, FILL from parent)
- Row height: auto (driven by tallest content cell — all cells use FILL height)
- Row label column: 160px FIXED on the left
- Cell padding: 16px all sides (generous whitespace for readability)
- Phase header height: 48px
- Row gap (dividers): 1px `night200` between rows
- Cell gap: 1px between columns
- Root frame padding: 48px all sides

### Visual polish guidelines:
- **Generous whitespace**: 16px cell padding minimum. Content should breathe.
- **Subtle grid lines**: 1px `night200` (`#EBEDF5`) borders on cells, not heavy outlines.
- **Phase headers**: Light gray fill (`night100` / `#F5F6FA`), semibold text, centered.
- **Row dividers**: 1px `night200` rectangles spanning full width between rows.
- **Accent bar**: 3px wide sky-50 (`#009ECC`) bar on left of every row label, FULL height of row.
- **Typography hierarchy**: Titles 30px Bold, labels 12px Semibold (uppercase), body 14px Regular, captions 12px Semibold.
- **Color coding**: Use tinted backgrounds for severity pills (grass-10 for Low, sun-10 for Medium, fire-10 for High) — not raw color fills.

See `references/design-tokens.md` for the Loveship palette and text styles.

## Proven Build Order (2 calls — Rule #68)

**Call 1: Structure + screenshots + first 2 content rows (~50% of the map)**
- Create root auto-layout frame (VERTICAL, white fill, 48px padding)
- Title + subtitle
- Phase headers row (HORIZONTAL, NIGHT100 phase cells)
- Screenshots row (if screenshots captured) — move uploaded image nodes into cells
- Actions row
- Touchpoints row
- Return root ID

**Call 2: Remaining 5 content rows (~50% of the map)**
- Get root by ID
- Thoughts row
- Emotions row (color-coded per phase)
- Pain Points row (with severity dots: CRITICAL/MAJOR/MINOR)
- User Effort row (with pills: Low/Medium/High)
- Opportunities row (green text)
- Return completion status

**Screenshot thumbnail sizing (Rule #69):**
```javascript
// Calculate thumbnail height from actual image dimensions
var cellWidth = 216; // COLW - 24px padding
var imgH = Math.round(cellWidth * (originalHeight / originalWidth));
img.resize(cellWidth, imgH);
img.cornerRadius = 4;
img.layoutSizingHorizontal = "FILL";
```

**This 2-call pattern is PROVEN** across Amazon Product Review (6 phases, 8 rows, no screenshots) and Amazon Checkout (6 phases, 9 rows, 5 screenshots). It replaces the old row-by-row approach.
