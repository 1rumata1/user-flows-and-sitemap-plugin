# Design Tokens — UUI Loveship Theme Reference

> Source of truth for all styling in flow diagrams and journey maps.
> This plugin uses the **EPAM UUI Loveship** theme ([https://uui.epam.com/?theme=loveship](https://uui.epam.com/?theme=loveship)) for look and feel.
> Colors are applied as **raw hex values** from the Loveship palette — NOT from a Figma design system library.
> No `bindColor()` or `importVariableByKeyAsync()` is needed. Set fills/strokes directly.

---

## Styling Approach

This plugin does **NOT** use Figma design system libraries (no EDS, no library variable binding). Instead, all colors, typography, and spacing follow the **UUI Loveship** theme specification. Colors are applied directly as solid hex fills.

**Why:** The Loveship theme provides a complete, self-contained design language. Using raw hex values from its palette keeps diagrams portable (no library dependencies) and visually consistent with UUI-based products.

---

## UUI Loveship Color Palette

### Night (Neutral Gray) Scale — primary structural palette

| Token | Hex | Use |
|---|---|---|
| `night50` | `#FAFAFC` | Lightest background tint |
| `night100` | `#F5F6FA` | App background, surface higher |
| `night200` | `#EBEDF5` | Surface highest, subtle fills |
| `night300` | `#E1E3EB` | Divider light, subtle borders |
| `night400` | `#CED0DB` | Dividers, control borders, skeleton |
| `night500` | `#ACAFBF` | Disabled text, placeholder text, control icons |
| `night600` | `#6C6F80` | Tertiary text, icons |
| `night700` | `#474A59` | Secondary text |
| `night800` | `#303240` | Primary text, headings |
| `night900` | `#1D1E26` | Darkest background (dark mode app bg) |

### Sky (Primary / Info) — accent color

| Token | Hex | Use |
|---|---|---|
| `sky-5` | `#F5FDFF` | Primary tint background |
| `sky-10` | `#E1F4FA` | Info background, highlight |
| `sky-20` | `#A0DDEE` | Primary light |
| `sky-50` | `#009ECC` | **Primary action, links, focus, accent** |
| `sky-60` | `#0086AD` | Primary hover |
| `sky-70` | `#006B8A` | Primary dark, info text |

### Grass (Success / Accent)

| Token | Hex | Use |
|---|---|---|
| `grass-5` | `#FCFFF5` | Success tint background |
| `grass-10` | `#EBF3D8` | Success light fill |
| `grass-50` | `#67A300` | **Success, positive indicators** |
| `grass-60` | `#528500` | Success hover |
| `grass-70` | `#396F1F` | Success text |

### Fire (Critical / Error)

| Token | Hex | Use |
|---|---|---|
| `fire-5` | `#FEF6F6` | Error tint background |
| `fire-10` | `#FDE1E1` | Error light fill |
| `fire-50` | `#FF4242` | **Error, critical indicators** |
| `fire-60` | `#E22A2A` | Error hover |
| `fire-70` | `#AD0000` | Error text |

### Sun (Warning)

| Token | Hex | Use |
|---|---|---|
| `sun-5` | `#FFFCF5` | Warning tint background |
| `sun-10` | `#FFEDC9` | Warning light fill |
| `sun-50` | `#FCAA00` | **Warning indicators** |
| `sun-60` | `#FF9000` | Warning hover |
| `sun-70` | `#BD5800` | Warning text |

### Additional Palette Colors (for charts, tags, categories)

| Color | Hex (base -50) | Use |
|---|---|---|
| Orange | `#FF8B3E` | Additional category |
| Fuchsia | `#EA4386` | Additional category |
| Purple | `#B114D1` | Additional category |
| Violet | `#773CEC` | Additional category |
| Cobalt | `#0F98FF` | Additional category |
| Cyan | `#14CCCC` | Additional category |
| Mint | `#4FC48C` | Additional category |

---

## Semantic Color Mapping for Diagrams

### Node Color Assignments (User Flow)

| Node Type | Fill Hex | Stroke Hex | Text Color |
|---|---|---|---|
| Root frame | `#FFFFFF` (surface-main) | — | — |
| Start / End (pill) | `#009ECC` (sky-50) | — | `#FFFFFF` |
| Step / Screen | `#FFFFFF` (surface-main) | `#CED0DB` (night400) | `#303240` (night800) |
| Decision | `#F5FDFF` (sky-5) | `#009ECC` (sky-50) | `#303240` (night800) |
| Error | `#FEF6F6` (fire-5) | `#FF4242` (fire-50) | `#AD0000` (fire-70) |
| Connector line | — | `#CED0DB` (night400) | — |
| Arrow polygon fill | `#6C6F80` (night600) | — | — |
| Branch label | — | — | `#6C6F80` (night600) |
| Legend frame | `#F5F6FA` (night100) | `#E1E3EB` (night300) | — |

### Journey Map Color Assignments

| Element | Fill Hex | Stroke Hex | Text Color |
|---|---|---|---|
| Root frame | `#FFFFFF` | — | — |
| Phase header cells | `#F5F6FA` (night100) | — | `#303240` (night800) |
| Content cells | `#FFFFFF` | `#EBEDF5` (night200) | `#303240` (night800) |
| Row label accent bar | `#009ECC` (sky-50) | — | — |
| Row label text | — | — | `#303240` (night800) |
| Divider lines | `#E1E3EB` (night300) | — | — |
| Pain point: CRITICAL dot | `#FF4242` (fire-50) | — | `#AD0000` (fire-70) |
| Pain point: MAJOR dot | `#FF9000` (sun-60) | — | `#BD5800` (sun-70) |
| Pain point: MINOR dot | `#FCAA00` (sun-50) | — | `#BD5800` (sun-70) |
| Effort pill: Low | `#EBF3D8` fill / `#67A300` text | — | `#396F1F` (grass-70) |
| Effort pill: Medium | `#FFEDC9` fill / `#FCAA00` text | — | `#BD5800` (sun-70) |
| Effort pill: High | `#FDE1E1` fill / `#FF4242` text | — | `#AD0000` (fire-70) |
| Opportunities text | — | — | `#396F1F` (grass-70) |
| Emotion labels (positive) | — | — | `#67A300` (grass-50) |
| Emotion labels (negative) | — | — | `#FF4242` (fire-50) |
| Emotion labels (neutral) | — | — | `#6C6F80` (night600) |
| Thoughts (quotes) | — | — | `#474A59` (night700) |
| Touchpoints text | — | — | `#6C6F80` (night600) |
| Screenshot captions | — | — | `#6C6F80` (night600) |

---

## Color Application Pattern

Since we do NOT use Figma library variables, apply colors directly. Every `use_figma` call should use this helper:

```javascript
function h(hex) {
  return { r: parseInt(hex.slice(1,3),16)/255, g: parseInt(hex.slice(3,5),16)/255, b: parseInt(hex.slice(5,7),16)/255 };
}

// Loveship Palette — paste into every use_figma call
var C = {
  // Surfaces
  WHITE:      "#FFFFFF",
  NIGHT50:    "#FAFAFC",
  NIGHT100:   "#F5F6FA",
  NIGHT200:   "#EBEDF5",
  NIGHT300:   "#E1E3EB",
  NIGHT400:   "#CED0DB",
  NIGHT500:   "#ACAFBF",
  NIGHT600:   "#6C6F80",
  NIGHT700:   "#474A59",
  NIGHT800:   "#303240",
  NIGHT900:   "#1D1E26",
  // Primary (Sky)
  SKY_5:      "#F5FDFF",
  SKY_10:     "#E1F4FA",
  SKY_50:     "#009ECC",
  SKY_60:     "#0086AD",
  SKY_70:     "#006B8A",
  // Success (Grass)
  GRASS_5:    "#FCFFF5",
  GRASS_10:   "#EBF3D8",
  GRASS_50:   "#67A300",
  GRASS_70:   "#396F1F",
  // Error (Fire)
  FIRE_5:     "#FEF6F6",
  FIRE_10:    "#FDE1E1",
  FIRE_50:    "#FF4242",
  FIRE_70:    "#AD0000",
  // Warning (Sun)
  SUN_5:      "#FFFCF5",
  SUN_10:     "#FFEDC9",
  SUN_50:     "#FCAA00",
  SUN_60:     "#FF9000",
  SUN_70:     "#BD5800",
};

// Apply fills/strokes directly — no variable binding
function setFill(node, hex) {
  node.fills = [{ type: "SOLID", color: h(hex) }];
}
function setStroke(node, hex, width) {
  node.strokes = [{ type: "SOLID", color: h(hex) }];
  node.strokeWeight = width || 1;
}
```

**NEVER use `bindColor()` or `importVariableByKeyAsync()`.** Set fills/strokes directly with `setFill()` and `setStroke()`.

---

## Typography

### Font Family

UUI Loveship uses **Source Sans Pro**. In Figma, load it before setting text:

```javascript
await figma.loadFontAsync({ family: "Source Sans Pro", style: "Regular" });
await figma.loadFontAsync({ family: "Source Sans Pro", style: "Semibold" });
await figma.loadFontAsync({ family: "Source Sans Pro", style: "Bold" });
```

**Fallback**: If Source Sans Pro is unavailable in the Figma file, fall back to **Inter** (Figma's default).

### Text Creation Pattern

```javascript
async function makeText(parent, content, fontSize, fontWeight, colorHex) {
  var t = figma.createText();
  var style = fontWeight || "Regular";
  try {
    await figma.loadFontAsync({ family: "Source Sans Pro", style: style });
    t.fontName = { family: "Source Sans Pro", style: style };
  } catch(e) {
    await figma.loadFontAsync({ family: "Inter", style: style === "Semibold" ? "Semi Bold" : style });
    t.fontName = { family: "Inter", style: style === "Semibold" ? "Semi Bold" : style };
  }
  t.characters = content;
  t.fontSize = fontSize;
  t.fills = [{ type: "SOLID", color: h(colorHex || C.NIGHT800) }];
  parent.appendChild(t);
  return t;
}
```

**Do NOT use `importStyleByKeyAsync()` for text styles.** Set font properties directly.

### Typography Scale

| Role | Font Size | Font Weight | Color | Use |
|---|---|---|---|---|
| `page_title` | 30px | Bold (700) | `#303240` (night800) | Diagram title |
| `page_desc` | 16px | Regular (400) | `#474A59` (night700) | Subtitle, persona description |
| `section_title` | 14px | Semibold (600) | `#303240` (night800) | Phase headers, row labels |
| `body` | 14px | Regular (400) | `#303240` (night800) | Cell content, actions, touchpoints |
| `body_secondary` | 14px | Regular (400) | `#474A59` (night700) | Thoughts (quotes) |
| `body_tertiary` | 14px | Regular (400) | `#6C6F80` (night600) | Touchpoints, captions |
| `label` | 12px | Semibold (600) | `#6C6F80` (night600) | Severity badges, effort pills, screenshot captions |
| `caption` | 11px | Regular (400) | `#ACAFBF` (night500) | Subtle annotations |

---

## Spacing (6px base grid)

UUI Loveship uses a **6px grid**. Map to diagram spacing:

| Token | Px | Use |
|---|---|---|
| 1 unit | 6 | Micro gaps |
| 2 units | 12 | Cell padding, row gaps |
| 3 units | 18 | Between sections |
| 4 units | 24 | Node padding, standard gaps |
| 5 units | 30 | Content area padding |
| 6 units | 36 | Large section breaks |
| 8 units | 48 | Frame padding |

## Border Radius

| Px | Use |
|---|---|
| 3 | Default (UUI base radius) |
| 4 | Screenshot thumbnails |
| 6 | Step nodes, cards, accordions |
| 9999 | Start/end pills |

## Shadows (optional, for elevated frames)

```javascript
// Level 1 — subtle card shadow
var SHADOW_1 = [
  { type: "DROP_SHADOW", color: { r: 0.114, g: 0.118, b: 0.149, a: 0.05 }, offset: { x: 0, y: 0 }, radius: 3, visible: true, blendMode: "NORMAL" },
  { type: "DROP_SHADOW", color: { r: 0.114, g: 0.118, b: 0.149, a: 0.1 }, offset: { x: 0, y: 3 }, radius: 6, visible: true, blendMode: "NORMAL" },
];

// Level 2 — elevated panel shadow
var SHADOW_2 = [
  { type: "DROP_SHADOW", color: { r: 0.114, g: 0.118, b: 0.149, a: 0.05 }, offset: { x: 0, y: 6 }, radius: 12, visible: true, blendMode: "NORMAL" },
  { type: "DROP_SHADOW", color: { r: 0.114, g: 0.118, b: 0.149, a: 0.1 }, offset: { x: 0, y: 3 }, radius: 12, visible: true, blendMode: "NORMAL" },
  { type: "DROP_SHADOW", color: { r: 0.114, g: 0.118, b: 0.149, a: 0.05 }, offset: { x: 0, y: 0 }, radius: 6, visible: true, blendMode: "NORMAL" },
];
```

---

## Migration Notes (from EDS)

If upgrading from the previous EDS-based styling:

| Old (EDS) | New (Loveship) |
|---|---|
| `bindColor(node, "fills", key, hex)` | `setFill(node, hex)` |
| `bindColor(node, "strokes", key, hex)` | `setStroke(node, hex, width)` |
| `makeText(parent, content, styleKey, hex)` | `makeText(parent, content, fontSize, fontWeight, hex)` |
| Dark background (`#0e0e0e`) | Light background (`#FFFFFF`) |
| White text on dark | Dark text (`#303240`) on light |
| Inter font family | Source Sans Pro font family |
| EDS library variable keys | Raw hex from Loveship palette |
| `importStyleByKeyAsync()` | Direct `fontSize` + `fontName` |
| `search_design_system()` for tokens | Use `C.` constants from palette above |

**Key visual change:** The Loveship theme is a **light theme** by default. Root frames use white (`#FFFFFF`) backgrounds with dark text (`#303240`). This is the opposite of the previous dark-mode EDS styling.
