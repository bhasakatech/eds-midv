# AEM Coder (aemcoder.adobe.io) — Tool Analysis
## Challenges, Resolutions, Usage Modes & Migration Options

**Project:** `eds-events-gehealthcare` (bhasakatech / eds-events-gehealthcare)  
**Tool:** Experience Modernization Console — [aemcoder.adobe.io](https://aemcoder.adobe.io)  
**Documentation source:** [Adobe Experience League — October 9, 2026](https://experienceleague.adobe.com/en/docs/experience-manager-cloud-service/content/ai-in-aem/agents/brand-experience/modernization/console)

---

## Table of Contents

1. [What is AEM Coder](#1-what-is-aem-coder)
2. [Challenges & How to Resolve Them](#2-challenges--how-to-resolve-them)
3. [How Many Ways to Use This Tool](#3-how-many-ways-to-use-this-tool)
4. [Migration Options Available](#4-migration-options-available)
5. [Application to This Project](#5-application-to-this-project)
6. [Governance & Deployment Flow](#6-governance--deployment-flow)

---

## 1. What is AEM Coder

`aemcoder.adobe.io` is Adobe's **Experience Modernization Console** — a hosted, browser-based, AI-assisted development environment for Edge Delivery Services (EDS). It connects directly to your GitHub repository and lets developers and non-developers interact with their EDS site using natural-language prompts, with no local environment required.

**Core capabilities at a glance:**

| Capability | Description |
|---|---|
| Interactive AI chat | Natural-language prompts drive all operations |
| Live AEM preview | See changes rendered in real-time |
| Content file browser | Browse pages and markdown files |
| Document Authoring sync | Upload migrated content directly to AEM/DA |
| Code browser + diff viewer | Review every file change before it ships |
| GitHub integration | Create pull requests from inside the console |
| Figma integration | Design-to-code block migration |

---

## 2. Challenges & How to Resolve Them

### 2.1 Access & Setup Challenges

| # | Challenge | Root Cause | Resolution |
|---|---|---|---|
| C1 | **Access is gated** — the console is not freely available | Early-access program with managed onboarding | Contact your Adobe account manager to request access. Access must be approved before any work can begin at `aemcoder.adobe.io`. |
| C2 | **GitHub app not installed** — console cannot inspect code | AEM Code Connector app missing from repo | Install the [AEM Code Connector](https://github.com/apps/aem-code-connector) GitHub app on the `eds-events-gehealthcare` repository. |
| C3 | **Code sync not working** | AEM Code Sync GitHub app not installed | Install [AEM Code Sync](https://github.com/apps/hlx-code-sync). For this project (boilerplate-based), this is already done. Verify at GitHub → Settings → Integrations. |
| C4 | **No write access token for push/PR** | GitHub token in Cloud Manager is read-only | In Cloud Manager → Repositories → Manage Access Tokens, edit the token to include write access. The repository is re-validated automatically. |
| C5 | **fstab.yaml mismatch** (AEM Authoring projects) | Site name or host is wrong in `fstab.yaml` | For this project: `fstab.yaml` already points to `author-p139816-e1765605.adobeaemcloud.com`. Verify the org (`bhasakatech`) and repo (`eds-events-gehealthcare`) values are correct and commit to `main` before connecting. |

### 2.2 Content Migration Challenges

| # | Challenge | Root Cause | Resolution |
|---|---|---|---|
| C6 | **Protected / intranet content not scraped** | Content is behind authentication, VPN, or firewall | Use **SLICC** (browser-context scraping tool) to access protected content. Alternatively, allow-list the following agent IPs in your firewall: `34.228.136.112`, `54.90.51.39`, `3.224.194.242` |
| C7 | **Complex dynamic content not captured** | JavaScript-heavy SPAs or interaction-dependent content | Client-side rendered content is supported if accessible via a direct URL. Content hidden via CSS (tabs, accordions, carousels) is also captured since it is present in the DOM. For server-side dynamic data, custom block development is required separately. |
| C8 | **Bulk import fails** | Single-page migration not run first | Always migrate at least one representative page per template first. This creates the import infrastructure (parsers, transformers, templates) that bulk import depends on. |
| C9 | **Block CSS does not match source design** | Design tokens not extracted before block styling | Complete Phase 1 of design migration (generates `styles/brand.css`) before styling individual blocks. Block CSS references these tokens. |
| C10 | **Navigation links render as buttons** | Document Authoring auto-wraps links in button classes | The `header.js` block should include code to strip button classes from nav links. The agent's navigation skill handles this automatically. |
| C11 | **Dynamic Media / Scene7 image URLs broken** | Images re-imported as static instead of preserving DM URLs | The agent detects DM/Scene7 sources and preserves them as DM URLs automatically during migration. No manual intervention needed. |

### 2.3 AI Context & Prompt Challenges

| # | Challenge | Root Cause | Resolution |
|---|---|---|---|
| C12 | **Agent loses context mid-conversation** | AI context window has a finite size; long sessions compact earlier instructions | (a) Add an `INSTRUCTIONS.md` or `CONTEXT.md` file to the repo with project rules, naming conventions, and brand guidelines. (b) Start new sessions with: `"Warm up by reading the project documentation"`. (c) If context is lost, clear chat and restart with the warm-up prompt. |
| C13 | **Agent makes wrong block mappings** | Ambiguous source HTML structure or missing project context | Use iterative prompting: let the agent ask clarifying questions, then refine. Do not front-load all requirements in the first prompt. |
| C14 | **Block critique never reaches 85% threshold** | Source and migrated content differ structurally | Critique runs a maximum of three iterations. If 85% is not reached, manually review the diff viewer in the Code view and apply targeted CSS or structural fixes via follow-up prompts. |
| C15 | **Agent generates inaccurate responses** | LLM limitations | Always review AI-generated code in the diff viewer before staging. Never push without human review. Code is reviewed in GitHub before merge. |

### 2.4 AEM Authoring (Universal Editor) Specific Challenges

| # | Challenge | Root Cause | Resolution |
|---|---|---|---|
| C16 | **Wrong boilerplate used** | Standard EDS boilerplate used instead of xwalk version | For UE/AEM Authoring projects use [`aem-block-collection-xwalk`](https://github.com/adobe-rnd/aem-block-collection-xwalk) as the template. This project already uses `aem-boilerplate-xwalk` — correct. |
| C17 | **`paths.json` causes errors in AEM authoring flow** | `paths.json` conflicts with AEM authoring setup | Delete `paths.json` and commit to `main` before connecting the console to an AEM Authoring project. |
| C18 | **Content upload package fails** | Admin permissions not granted in console settings | In Settings → Agent Permissions, enable: `Allow LLM to access admin.hlx.page on my behalf`. |
| C19 | **Content not visible after upload** | Preview not triggered after upload | After uploading, go to Document Authoring (`da.live`), open the page, and click the paper plane icon → Preview. |

### 2.5 Code Quality & Deployment Challenges

| # | Challenge | Root Cause | Resolution |
|---|---|---|---|
| C20 | **Linting fails after agent writes code** | AI-generated code may not match project ESLint/Stylelint config | Run `npm run lint:fix` after any agent-generated code changes. The console's Push modal triggers PR checks that include linting. |
| C21 | **Changes not appearing in preview** | Code sync delay | Code sync takes 3–5 seconds after a push. Hard-refresh with `Ctrl+Shift+R`. |
| C22 | **Images showing `about:error`** | `createOptimizedPicture` not called, or called after DOM restructuring | In block JS, call `createOptimizedPicture` only after the final DOM structure is in place. The agent's debugging skill handles this pattern automatically if prompted. |
| C23 | **PSI score drops after migration** | Unoptimized assets or heavy JS committed to repo | All assets committed to git must be optimized. Use `lazy-styles.css` for non-critical CSS and `delayed.js` for MarTech. Images from authors are auto-optimized; git-committed images are not. |
| C24 | **Workspace reset loses all work** | "Reset workspace" clicked accidentally | This action is irreversible. Ensure all changes are staged and pushed to GitHub before touching the Danger Zone. |

---

## 3. How Many Ways to Use This Tool

The console supports **four distinct usage modes**, from fully automated to fully manual:

### Mode 1: Plan Mode (Analyze Only)
- Toggle the **lightbulb icon** in the prompt input
- The agent analyzes your request and describes its intended approach — no files are changed
- Use this to understand what the agent will do before committing
- Best for: scoping a migration, reviewing a block strategy, assessing impact

```
Example: "Plan how to migrate https://events.gehealthcare.com/events/rsna-2026"
→ Agent outlines steps, identifies blocks, lists files it will create — but writes nothing
```

### Mode 2: Execute Mode (AI-Driven Development)
- Toggle the **magic wand icon** in the prompt input
- The agent carries out the full plan and writes actual file changes
- Changes appear in the Code → Changes view as diffs before you stage/push
- Best for: migrations, block creation, design extraction, bulk imports

```
Example: "Migrate the design from https://events.gehealthcare.com"
→ Agent creates brand.css, styles.css, updates all block CSS files
```

### Mode 3: Prompt Queue (Automated Sequential Execution)
- Click the **clock icon** in the prompt input
- Queue multiple prompts to run back-to-back automatically
- Each queued prompt executes only after the previous one completes
- Best for: unattended long-running workflows (overnight migrations)

```
Example queue:
  1. "Migrate the page https://events.gehealthcare.com/events/rsna-2026"
  2. "Migrate the design from https://events.gehealthcare.com"
  3. "Critique site"
  4. "Create documentation for this project"
```

### Mode 4: Inspect Mode (Targeted Context)
- Click the **select icon** in the Content Preview panel
- Click any element on the live preview page to attach it to your prompt
- The agent receives that element's HTML as direct context
- Best for: fixing specific layout issues, targeting a broken block, precise style corrections

```
Example: Click on the event-hero title → "Fix the font size of this element to match the source"
```

### Mode 5: File Context Mode (Code-Level Targeting)
- In Code → Files view, select any file or highlight specific lines
- Click **Add to chat** to send that content to the chat panel
- Best for: asking the agent to modify or explain a specific block JS or CSS

```
Example: Select lines 45–78 of event-hero.js → "Refactor this section to handle missing media gracefully"
```

### Mode 6: Demo Mode (Exploration Without a Repo)
- Available automatically on first sign-in, before connecting your own project
- A pre-configured demo site is provided for exploring all features
- No GitHub connection required
- Best for: onboarding, training, exploring capabilities before committing your repo

---

## 4. Migration Options Available

The tool provides **five distinct migration pathways**, each suited to a different source and target scenario.

---

### Option A: Live Website → Document Authoring (Standard Migration)

**When to use:** Migrating an existing public website into EDS with Document Authoring (Google Docs / SharePoint) as the CMS.

**Source:** Any publicly accessible website URL  
**Target:** Document Authoring (da.live) + EDS GitHub repo  
**Tooling used in this project:** `fstab.yaml` with DA mount point

**Workflow:**

```
1. Scrape source page (auto)
   ↓
2. Identify section structure (auto)
   ↓
3. Authoring analysis — follow David's Model (auto)
   ↓
4. Map content to block variants (auto + interactive)
   ↓
5. Generate markdown document (auto)
   ↓
6. Create import infrastructure: parsers, transformers, page templates (auto)
   ↓
7. Preview result in console (auto)
   ↓
8. Upload to Document Authoring via "Upload content" (manual)
   ↓
9. Preview in AEM → Push code to GitHub PR (manual)
```

**Example prompts:**
```
"Migrate this page: https://events.gehealthcare.com/events/rsna-2026"
"Migrate these pages: URL1, URL2, URL3"
"Run bulk import on all event detail pages"
```

**Time estimate:** Single page: 5–15 min. Bulk: scales with page count.

---

### Option B: Live Website → AEM Authoring / Universal Editor

**When to use:** Migrating into EDS with AEM Sites + Universal Editor as the authoring environment (this project's current setup — `fstab.yaml` points to `author-p139816-e1765605.adobeaemcloud.com`).

**Source:** Any publicly accessible website URL  
**Target:** AEM Sites (Universal Editor) + EDS GitHub repo  

**Additional steps vs. Option A:**

```
1. Use aem-block-collection-xwalk boilerplate (✓ already done for this project)
2. Delete paths.json from repo root (action required)
3. Create AEM Site using "AEM Site with Edge Delivery Services Template" in Sites console
4. Connect AEM Code Connector to repo
5. Run standard migration (same as Option A)
6. Validate with "Errors" button in content panel
7. Upload as content package → Upload to AEM (not DA)
   → Downloads a .zip content package and uploads to AEM host
8. Preview via UE → Push code → PR
```

**Example prompts:**
```
"Migrate this page: https://events.gehealthcare.com/events/rsna-2026"
"Upload the content to AEM"
"Validate the RSNA page"
```

---

### Option C: Figma Design → EDS Block (Design-First Migration)

**When to use:** A redesigned experience exists in Figma and blocks need to be built from design components. No live source site required.

**Source:** Figma file URL (with `fileKey` and `node-id`)  
**Target:** EDS block files (JS, CSS, JSON model)  
**Requires:** Figma personal access token configured in Settings → Credentials

**Workflow:**

```
1. Agent reads Figma node structure (layers, components)
2. Extracts global design tokens → styles/brand.css (colors, typography, spacing)
3. Creates block-specific CSS custom properties
4. Maps to closest existing block in block library
5. Downloads images and icons from Figma to workspace
6. Generates markdown test content following EDS block structure
7. Previews result and compares visually to Figma design
```

**Example prompts:**
```
"Migrate the block https://figma.com/design/{fileKey}?node-id={nodeId} to Edge Delivery Services"
"Migrate the Event Hero block from https://figma.com/file/{fileKey}"
```

**Required Figma token scopes:**
- `file_content:read`, `file_metadata:read`, `library_assets:read`, `library_content:read`, `team_library_content:read`, `file_dev_resources:read`, `projects:read`

---

### Option D: Figma Redesign + Live Content (Hybrid Migration)

**When to use:** An existing website is being redesigned. Figma provides the new design system; the live site provides the content. Both sources are used simultaneously.

**Source 1:** Live website (content source)  
**Source 2:** Figma (design / block collection source)  
**Target:** EDS GitHub repo

**Workflow:**

```
Step 1 — Build block collection from Figma:
"Create the EDS block collection from https://figma.com/design/{fileKey}?node-id={nodeId}"
  → Creates block variants derived from Figma components
  → Extracts design tokens

Step 2 — Migrate content into Figma-derived blocks:
"Migrate these pages and map content to the Figma-derived block collection: URL1, URL2"
  → Scrapes live site for content
  → Maps content into the new Figma-derived blocks (not the old ones)
  → Validates content against source site
  → Validates visual output against Figma design
```

**Key difference from Option C:** Content comes from the live site, not from Figma. Design comes from Figma, not from the live site.

---

### Option E: New Page from Figma (No Source Website)

**When to use:** A page does not exist on the current live site and needs to be created entirely from a Figma frame.

**Source:** Figma frame or page  
**Target:** New EDS markdown page + block files

**Example prompts:**
```
"Migrate this Figma page to Edge Delivery Services: https://figma.com/design/{fileKey}?node-id={nodeId}"
```

**Notes:**
- Works best with a single Figma frame, not an entire file
- The frame should be organized into clear page sections
- Sections are mapped to existing blocks, default content, or new variants
- Dynamic features (search, store locator, calculators) require additional block development beyond what the migration produces

---

### Option F: Bulk Import (Scale-Out)

**When to use:** After a single representative page has been migrated (Option A or B), run the same migration logic across hundreds of pages.

**Dependency:** At least one representative page must be migrated first (creates parsers + transformers).

**Recommended workflow:**

```
1. Migrate one representative page (establishes import infra)
2. Run bulk import on a small test set (5–10 pages)
3. Review and fix any issues with parsers or transformers
4. Scale up to the full page list
```

**Constraints:**
- One template per bulk import run (no mixing templates)
- All parsers are validated before the run starts
- Mixing templates in a single run is not supported

**Example prompts:**
```
"Run the bulk import on these pages: URL1, URL2, URL3"
"Bulk import all event detail pages"
```

---

### Migration Option Comparison

| Option | Source | Target | Authoring CMS | Figma Needed | Bulk Support |
|---|---|---|---|---|---|
| A — Live → DA | Public website | Document Authoring | Google Docs / SharePoint | No | Yes (after first page) |
| B — Live → AEM UE | Public website | AEM Sites + UE | Universal Editor | No | Yes (after first page) |
| C — Figma → Block | Figma design | EDS blocks only | Either | Yes | No (one block at a time) |
| D — Figma Redesign + Live | Figma + Live site | Full EDS site | Either | Yes | Yes (after block collection built) |
| E — New page from Figma | Figma frame | New EDS page | Either | Yes | No |
| F — Bulk Import | Prior migration infra | All pages same template | Either | No | Yes (purpose-built) |

---

## 5. Application to This Project

**Repository:** `bhasakatech/eds-events-gehealthcare`  
**Authoring type:** AEM Authoring with Universal Editor (`fstab.yaml` → AEM Cloud host)  
**Applicable migration path:** **Option B** (Live → AEM Authoring / Universal Editor)

### Current Block Inventory (27 blocks)

The following blocks exist in this project. The console can be used to critique, improve, or add new blocks to each.

| Block | Status | Recommended Console Action |
|---|---|---|
| `event-hero` | Custom (active) | Critique against live source; add video variant |
| `event-list` | Custom | Critique against source |
| `activity-list` | Custom | Critique against source |
| `theater-sessions` | Custom | Critique against source |
| `hero`, `hero-media`, `hero-text` | Custom | Consolidate or validate variants |
| `cards`, `feature-cards` | Standard-derived | Critique for brand alignment |
| `content-band` | Custom | Validate design tokens |
| `call-to-action` | Custom | Validate against brand.css tokens |
| `announcement-bar` | Custom | Validate |
| `columns` | Standard | Validate |
| `faq-details` | Custom | Map to accordion pattern |
| `product`, `product-list`, `products-solutions` | Custom | Validate data model |
| `form` | Custom | Validate accessibility |
| `video-text` | Custom | Validate Vidyard integration |
| `quote`, `offer` | Custom | Validate |
| `header`, `footer` | Navigation | Use Navigation Setup skill |
| `section-nav` | Custom | Validate |
| `user-profile` | Custom | Validate |
| `fragment`, `experience-fragment` | Standard | No changes needed |
| `asset-picker` | Custom | No migration needed |

### Pre-Connection Checklist for This Project

- [ ] Request access at `aemcoder.adobe.io` via Adobe account manager
- [ ] Install [AEM Code Connector](https://github.com/apps/aem-code-connector) on `bhasakatech/eds-events-gehealthcare`
- [ ] Verify `fstab.yaml` is correct (`author-p139816-e1765605.adobeaemcloud.com`, org `bhasakatech`)
- [ ] Delete `paths.json` from repo root and commit to `main` (required for AEM Authoring)
- [ ] Ensure GitHub token in Cloud Manager has write access (for push + PR creation)
- [ ] Add Figma personal access token in Settings → Credentials (for design-first workflows)
- [ ] Enable "Allow LLM to access admin.hlx.page" in Settings → Agent Permissions
- [ ] Create `INSTRUCTIONS.md` at repo root with brand guidelines, naming conventions, and block rules

### Recommended First Prompts (After Connection)

```
Warm up:
"Warm up by reading the project documentation"

Site scope:
"Scope site https://events.gehealthcare.com"

Single page migration:
"Migrate this page: https://events.gehealthcare.com/events/rsna-2026"

Design extraction:
"Migrate the design from https://events.gehealthcare.com"

Block validation:
"Critique event-hero block"

Documentation generation:
"Create documentation for this project"
```

---

## 6. Governance & Deployment Flow

The console enforces a developer-review gate at every step. Nothing goes live without human approval.

```
Prompt in console
       ↓
Agent generates file changes
       ↓
Changes appear in Code → Changes view (diffs)
       ↓
Developer reviews diffs → Stages changes (+)
       ↓
Push modal → Create PR or push to current branch
       ↓
GitHub PR checks run (code sync, lint, PSI)
       ↓
Human reviewer inspects the preview URL in PR description
       ↓
PR merged to main → AEM Code Sync updates production
```

**Key governance rules:**
- All changes require staging before push — nothing is auto-deployed
- PR descriptions must include a preview URL (`https://{branch}--eds-events-gehealthcare--bhasakatech.aem.page/{path}`)
- Linting (`npm run lint`) must pass before merging
- PageSpeed Insights score of 100 is the target for production pages

---

## References

- [Experience Modernization Console](https://experienceleague.adobe.com/en/docs/experience-manager-cloud-service/content/ai-in-aem/agents/brand-experience/modernization/console)
- [Getting Started — Document Authoring](https://experienceleague.adobe.com/en/docs/experience-manager-cloud-service/content/ai-in-aem/agents/modernization/getting-started)
- [Getting Started — AEM Authoring](https://experienceleague.adobe.com/en/docs/experience-manager-cloud-service/content/ai-in-aem/agents/brand-experience/modernization/getting-started-aem-authoring)
- [Prompting Guide](https://experienceleague.adobe.com/en/docs/experience-manager-cloud-service/content/ai-in-aem/agents/brand-experience/modernization/prompting-guide)
- [Experience Modernization Agent Overview](https://experienceleague.adobe.com/en/docs/experience-manager-cloud-service/content/ai-in-aem/agents/brand-experience/modernization/overview)
- [Project Documentation Skill](https://experienceleague.adobe.com/en/docs/experience-manager-cloud-service/content/ai-in-aem/agents/brand-experience/modernization/project-documentation)

*Content was rephrased for compliance with licensing restrictions.*
