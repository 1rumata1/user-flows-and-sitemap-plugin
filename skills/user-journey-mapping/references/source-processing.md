# Source Processing

How to extract flow data from each input type. For every source, you're looking for the same things: entry point, screens, actions, decisions (with branches), errors, and end point. Journey maps also need: phase, touchpoint, emotion, pain point, opportunity. Sitemaps need: page hierarchy, section groupings, page types, navigation depth. Screen Flows need: screen names, navigation triggers, conditional routes.

## Text Docs (PRD, Spec, Wiki)

Look for step-by-step behavior descriptions, acceptance criteria ("Given/When/Then"), state transitions, screen lists, navigation descriptions, and error handling sections. Documents usually describe only the happy path — ask about error cases.

## Video / Recording

Claude can't watch video directly. Options for extracting screenshots:

1. **Transcript**: Ask for Loom auto-transcript, YouTube captions, or Otter.ai export. Walk through: "What's the first screen? What does the user do? What happens next? Any choices? What if it fails?"
2. **User-provided timestamps**: Ask the user to note timestamps of key screens, then extract frames at those points using ffmpeg or similar.
3. **Manual walkthrough**: If no transcript available, interview step by step and ask the user to take screenshots at each key moment.
4. **Screen recording file**: If the user shares an .mp4/.mov, extract key frames at transition points using `ffmpeg -i video.mp4 -vf "select=gt(scene\,0.3)" -vsync vfr frame_%03d.png`

## Live Web App URL

Use **Playwright** to capture screens interactively:

1. Install: `npm install playwright` + `npx playwright install chromium`
2. Launch a **headed** (visible) Chrome browser: `chromium.launch({ headless: false, channel: 'chrome' })`
3. Navigate to the user's URL. If the app requires auth, the user logs in manually in the visible browser window.
4. Auto-capture screenshots on URL change + periodic interval (12–15s). Save to `/tmp/screenshots/` with sequential numbering.
5. Tell the user the browser is open and to walk through the flow.
6. When done, deduplicate by file size (same size = same page), keep only distinct screens.
7. Upload unique screenshots to Figma via `upload_assets` + `curl POST` with raw bytes.

**macOS tip**: If the browser isn't visible, use `osascript -e 'tell application "Google Chrome" to activate'` to bring it to the foreground.

**Auth-required apps — NEVER silently skip (Rule #65):**

When you navigate to a URL and get redirected to a login page (sign-in form, OAuth redirect, 401/403), you MUST stop and ask the user to log in. Do NOT silently produce "Auth required" placeholders.

**Auth handling protocol:**
1. **Detect the auth wall.** Check if the current URL contains `/signin`, `/login`, `/ap/signin`, `/auth`, or if the page title contains "Sign In", "Log In".
2. **Open a persistent headed browser:**
   ```javascript
   // Use persistent context so the login session survives restarts
   var browser = await chromium.launchPersistentContext('/tmp/app-session', {
     channel: 'chrome', headless: false
   });
   var page = browser.pages()[0] || await browser.newPage();
   await page.goto(targetUrl);
   ```
3. **Bring to foreground (macOS):**
   ```bash
   osascript -e 'tell application "Google Chrome" to activate'
   ```
4. **Ask the user:**
   > "I hit a login wall at [URL]. The browser is open — please sign in, then tell me when you're ready and I'll resume capturing."
5. **Wait for user confirmation.** Do NOT proceed until they confirm.
6. **Check the post-login URL (Rule #66).** Most sites redirect to their homepage after login, NOT to the intended page. After user confirms:
   - Check `page.url()` — if it's the homepage or unexpected, navigate back manually
   - For checkout flows: go to cart page first, then click "Proceed to Checkout" again
   - For review flows: navigate directly to the review creation URL
   - Verify you're on the intended authenticated page before capturing
   ```javascript
   // After user confirms login:
   var currentUrl = page.url();
   if (currentUrl.includes('/?') || currentUrl.endsWith('.com/')) {
     // Redirected to homepage — navigate back to intended page
     await page.goto(intendedUrl);
     // For checkout flows: go to cart page, then proceed through checkout again
   }
   ```
7. **Resume capturing** from where you left off. Also capture the sign-in page itself as a screenshot — it IS a real screen in the journey (Rule #67).
8. **Only use placeholders if the user explicitly declines to log in.** In that case, list every skipped screen in the capture summary as "NOT captured — user declined auth".

URL patterns often reveal structure (`/signup` → `/verify` → `/dashboard`). Look for navbars, breadcrumbs, and step indicators.

### Sitemap Extraction from Live URLs

When the goal is a **sitemap / IA diagram** (not a user flow), extraction requires **exhaustive, detailed navigation discovery**. Do NOT rely on a single page snapshot — you must systematically explore every navigable surface.

**CRITICAL: Sitemaps are DETAILED by default.** You MUST drill at least 3 levels deep (Root → Sections → Sub-pages → Notable children). A sitemap with only top-level navigation items is NOT acceptable — it's a navigation bar, not a sitemap. For e-commerce sites, this means mapping product categories AND their sub-categories. For SaaS apps, this means every sidebar section AND its tabs/sub-pages.

**PROVEN EXTRACTION STRATEGY (Rule #55): Use `browser_evaluate` for programmatic extraction.**

Instead of manually hovering/clicking through menus (slow, fragile), use Playwright's `browser_evaluate` to run JavaScript that extracts navigation links programmatically. This is 10x faster.

**Step 1: Extract main categories from the homepage.**
```javascript
// Navigate to homepage, then run:
browser_evaluate(() => {
  const cats = [];
  document.querySelectorAll('a').forEach(a => {
    const href = a.href;
    const text = a.textContent.trim();
    // Adapt the URL pattern for each site (e.g., /c\d+/ for category IDs, /category/ for slugs)
    if (text && href && href.match(/\/c\d+\//) && text.length > 2 && text.length < 80) {
      cats.push({ text, href });
    }
  });
  const seen = new Set();
  return cats.filter(c => { if (seen.has(c.href)) return false; seen.add(c.href); return true; });
});
```

**Step 2: Navigate to 3-5 key category pages and extract sub-categories from each.**
Repeat the `browser_evaluate` pattern on each category page to discover L2 and L3 sub-pages.

**Step 3: Extract utility, footer, and account links separately.**
```javascript
browser_evaluate(() => {
  const results = { footer: [], utility: [] };
  document.querySelectorAll('footer a').forEach(a => { /* ... */ });
  // Also check for /cabinet/, /wishlist, /cart, /checkout, /tracking, /comparison, etc.
  return results;
});
```

**Step 4: Compile the tree and present for confirmation.**

**ALSO do the manual discovery checklist for items that `browser_evaluate` can miss:**

1. **Dropdown menus** — hover each nav item to check for dropdowns that load dynamically
2. **Avatar / account menu** — click to reveal Account, Settings, Sign out
3. **Organization / workspace selector** — click to reveal options
4. **Tab bars within pages** — navigate into sections to discover tabs
5. **Footer links** — about, careers, help, legal pages
6. **Shadow DOM / web components** — query shadow roots if needed

**After discovery, present the complete indented tree and get user confirmation before rendering.** If the tree has more than ~80 nodes, propose a scoping strategy:
- "Full sitemap" — everything, rendered across multiple batches
- "Org-level only" — just the organization structure, skip product drill-down
- "Product-level only" — just one product's pages in detail
- "Top 2 levels" — sections and their direct children only

### Screen Flow Extraction from Live URLs

Screen Flows REQUIRE screenshots. Use Playwright to capture each screen:

1. **Capture every distinct screen** in the flow via auto-screenshot on URL change
2. **Name each screenshot** with a short label (e.g., "01-achievements-list")
3. **Record navigation triggers** — what action moves from screen A to screen B
4. **Capture decision points** — dropdowns, modals, branching dialogs
5. **Focus on the specific flow** — don't need to discover every page, just the screens in this user task

## Screenshots

Identify each screen's name, key UI elements, and available actions. Look for breadcrumbs, progress bars, and step indicators that reveal sequence. Ask the user to confirm the order if ambiguous.

## FigJam Board

Use `get_figjam_metadata` to read structure. Sticky notes = steps, shapes = nodes, connectors = flow direction, sections = stages. Follow connector arrows to reconstruct sequence.

## Figma Design File

Use `get_metadata` to read pages and frames. Frame names often reveal sequence ("01-Login", "02-Dashboard"). Look for prototype connections — they explicitly define the flow.

## Verbal Description

Interview: "Walk me through the experience from the user's perspective, starting at the beginning." For each step: "What do they see? What do they do? What happens? Any choices? What if something goes wrong?" Read the full list back for confirmation.

## Codebase

Read router/navigation config (routes.js, App.tsx, navigation.ts) for the page inventory. Read page components for user actions (buttons, forms, links). Follow navigation calls and redirects. Check auth guards for login-required flows.

## Analytics / Funnel Data

Use page view sequences for actual paths, drop-off points for friction identification, conversion rates per step. Best used as annotations on top of a flow extracted from another source.

---

## Output Format

**User Flow** — present as a numbered list:

```
Flow: [Name]
Persona: [Who]
Goal: [What they're trying to do]

1. [Start] First screen
2. [Action] What user does
3. [Screen] What they see next
4. [Decision] Choice? → Yes: step N / No: step M
5. [Error] What happens on failure → back to step N
6. [End] Final screen
```

**Sitemap / IA** — present as an indented tree:

```
Sitemap: [Product Name]

Home (LANDING)
├── Section A (SECTION)
│   ├── Page 1 (LIST)
│   ├── Page 2 (DETAIL)
│   │   ├── Sub-page (SETTINGS)
│   │   └── Sub-page (FORM)
│   └── Page 3 (DASHBOARD)
├── Section B (SECTION)
│   ├── Page 4 (LIST)
│   └── Page 5 (FORM)
├── External Link (EXTERNAL ↗)
└── Another External (EXTERNAL ↗)
```

Page types: LANDING, DASHBOARD, LIST, DETAIL, FORM, SETTINGS, AUTH, EXTERNAL.

**Screen Flow** — present as screenshot list + paths:

```
Screen Flow: [Name]

Screenshots captured:
  01. Login Page
  02. Dashboard
  03. Settings
  04. Profile Tab
  05. 2FA Setup

Paths:
  01 --Submits--> 02
  02 --Clicks Settings--> 03
  03 --Clicks Profile Tab--> 04
  03 --Clicks Security Tab--> 05
  03 -.Back.-> 02
```

Confirm with user before rendering.
