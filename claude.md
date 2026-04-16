# F1 Fan Art Design Tool — Project Spec

## Current Implementation State (as of Apr 2026)

Key divergences from the original spec below:

- **Script tag**: plain `<script>` (NOT `type="module"`) — required for `file://` compatibility
- **`{{STYLE_NOTES}}` token removed** — style direction is now baked into each template. Do not re-add a style notes field or token.
- **Multi-image per version**: `ver.images: string[]` (was `ver.image_path: string`). Migration handled by `migrateDesign()`. File naming: `{slug}_{product}_v{vNum}_{imgIdx}.{ext}`
- **Cropped elements**: saved to `images/crops/{team}_{slug}_{edition}_v{vNum}_{type}_{n}.png` — version is part of the filename so crops from different design versions never overwrite each other
- **Products**: T-Shirt and Hoodie only (cap/tote removed)
- **Templates**: `db/templates.json` in repo, editable via hidden JSON editor tab (triple-click header text to unlock)
- **New version from vault**: `newVersionFromVault(designId)` — creates blank new version without visiting generator
- **Extraction prompt**: shown in modal gallery after first upload — user copies and runs with image in AI tool to get separate PNGs
- **Crop tool**: canvas-based region selector, saves to `images/crops/`, 3 region types (front/back/sleeve)
- **Championship years**: auto-populate from `driver.championship_years[]` when a champ edition driver is selected
- **Primary AI image generation tools**: Artlist AI (main, flat lay style) and ChatGPT/DALL-E (iteration and quick edits). How To reflects this.

---

## Overview

A lightweight, standalone HTML single-page application (no build step, no framework, just HTML + Tailwind CDN + vanilla JS) used by three collaborators — **Ayush**, **Anagh**, and **Vanshika** — to manage AI image generation prompts and design assets for an F1 fan art streetwear brand.

All persistence is via the **GitHub API** using a Personal Access Token (PAT). JSON files in the repo act as the database; images are stored as base64-encoded blobs in the repo.

---

## Legal Design Rules (enforce throughout the app)

These rules apply to every prompt generated. The app must surface them as a persistent reminder banner on Tab 1 and enforce them via a Legal Compliance Checklist before saving a prompt.

| Rule | What's allowed | What's NOT allowed |
|---|---|---|
| Team identity | Colors inspired by team palette, original fan-art badge | Official team logos, exact Prancing Horse, charging bull, star compass — any trademarked insignia |
| Typography | Plain neutral sans-serif for team name | Replicating official team font styles or treatments |
| Helmet art | Original illustrated outline, inspired silhouette | Exact replica of a real helmet livery |
| Car silhouette | Generic era-accurate side profile (e.g. "V10-era F1 car") | Any livery colors, sponsor logos, or team-identifiable markings on the car |
| Badge / crest | Original artistic creation inspired by heritage | Ferrari shield, Red Bull shield, McLaren chevron, Mercedes star — any registered trademark shape |
| Number | Driver's actual racing number (factual info) | Official number treatment with team-branded styling |
| Circuit layout | Original fine-line art of a circuit map | Any official FIA/Tilke trademarked circuit artwork |
| Crown graphic | Original artistic crown element | Any licensed crown imagery |
| Text on garment | Nickname (original), plain team name | "FAN ART" text, official sponsor names, official taglines |

**Example violation to avoid — the original Leclerc base prompt included:**
> _"a Ferrari-style shield badge (yellow with black prancing horse)"_

This is NOT allowed. The corrected version is:
> _"an original artistic crest in yellow featuring a rearing horse silhouette — a fan-created graphic inspired by Italian racing heritage, not an official logo or trademark"_

The app should display the legal rules as a collapsible callout on Tab 1 so all three users stay aligned.

---

## Tech Stack

| Layer | Choice | Reason |
|---|---|---|
| Markup | HTML5 (single file) | No build step, portable |
| Styling | Tailwind CSS (CDN) | Utility-first, no config needed |
| Logic | Vanilla JS (ES modules in `<script type="module">`) | Lightweight, no dependencies |
| Persistence | GitHub REST API v3 | Free, version-controlled, accessible to all 3 users |
| Images | GitHub repo (base64 via API) | Same repo, no extra service |
| Auth | GitHub PAT entered per-session (never stored in code) | Simple, secure |

---

## GitHub Repo Structure

```
f1-designs/                          ← your GitHub repo root
├── db/
│   ├── drivers.json                 ← master list of teams + drivers + nicknames
│   ├── designs.json                 ← all design records (prompts, assignees, status)
│   └── assignments.json             ← which drivers are assigned to whom
├── images/
│   └── {driver_slug}/
│       ├── front.png                ← front-of-shirt design image
│       ├── back.png                 ← back-of-shirt design image
│       └── print_ready.png          ← background-removed PNG for print
├── index.html                       ← THE APP
└── README.md
```

---

## Data Models

### `db/drivers.json`
```json
[
  {
    "slug": "leclerc",
    "display_name": "Charles Leclerc",
    "first_name": "Charles",
    "last_name": "Leclerc",
    "nickname": "IL PREDESTINATO",
    "number": 16,
    "team": "Ferrari",
    "team_colors": {
      "primary": "#E8002D",
      "secondary": "#FFD700"
    },
    "championship_years": [],
    "unique_element": "Monaco street circuit layout silhouette (home GP)",
    "front_badge_description": "an original artistic crest in yellow featuring a rearing horse silhouette — a fan-created graphic inspired by Italian racing heritage, not an official logo or trademark",
    "back_graphic_description": "A large outlined Monaco street circuit layout silhouette — the hairpin, tunnel exit and Swimming Pool complex rendered as original illustrated fine line art — in subtle red/gold",
    "style_notes": "Modern F1 streetwear aesthetic, passionate and precise. High contrast Ferrari red, gold, white on black. Colors inspired by Italian motorsport heritage.",
    "is_championship_edition": false
  }
]
```

### `db/designs.json`
```json
[
  {
    "id": "leclerc_tshirt_v1",
    "driver_slug": "leclerc",
    "product_type": "tshirt",
    "tshirt_color": "#000000",
    "tshirt_color_name": "Black",
    "assigned_to": "Anagh",
    "status": "partially_uploaded",
    "prompt_version": 3,
    "manual_override": false,
    "prompt_fields": {
      "first_name": "Charles",
      "last_name": "Leclerc",
      "number": "16",
      "nickname": "IL PREDESTINATO",
      "team_name": "Scuderia Ferrari",
      "primary_color": "Ferrari red",
      "secondary_color": "gold",
      "front_badge": "an original artistic crest in yellow featuring a rearing horse silhouette — a fan-created graphic inspired by Italian racing heritage, not an official logo or trademark",
      "back_graphic": "A large outlined Monaco street circuit layout silhouette rendered in subtle red/gold fine line art",
      "champ_years": "",
      "style_notes": "Modern F1 streetwear, passionate and precise. High contrast Ferrari red, gold, white on black.",
      "unique_element": "Monaco circuit hairpin and tunnel exit layout",
      "is_championship_edition": false
    },
    "prompt_text": "< full assembled prompt text stored here >",
    "legal_checklist_confirmed": true,
    "images": {
      "front": {
        "path": "images/leclerc/front.png",
        "uploaded_by": "Anagh",
        "uploaded_at": "2026-04-15T10:00:00Z"
      },
      "back": null,
      "print_ready": null
    },
    "approved_by": null,
    "approved_at": null,
    "notes": "V3 — replaced Ferrari shield with original crest to comply with legal rules"
  }
]
```

**Image slot keys**: `front` | `back` | `print_ready`. Each slot is independently uploadable. A record is `fully_uploaded` only when all three slots are non-null.

### `db/assignments.json`
```json
{
  "Ayush":    ["hamilton", "verstappen", "norris", "alonso"],
  "Anagh":    ["leclerc", "sainz_c", "russell"],
  "Vanshika": ["piastri", "zhou", "albon"]
}
```

---

## Application Layout

### Global Elements (always visible)
- **Header**: "F1 FAN ART STUDIO" wordmark
- **User selector**: Dropdown — Ayush / Anagh / Vanshika (sets active user for session; drives default filter in Tab 2)
- **GitHub config**: Collapsible panel — PAT input (password field), repo owner, repo name. "Connect" button validates by fetching `db/drivers.json`. Connection status shown as colored dot.
- **Tab bar**: `PROMPT GENERATOR` | `DESIGN VAULT`

---

## Tab 1: Prompt Generator

### Prompt Architecture — Locked vs Editable

The prompt has two layers:

**Locked sections (consistent across all designs — not exposed in the form):**
- Opening product mockup description line
- Front: left-chest typography structure, accent line format, neckline description
- Back: number positioning, crown description, racing streaks format, bottom text format, legal disclaimer line

**Editable sections (driver/design-specific — controlled by form fields):**
- All fields listed in the table below

The form is NOT a freeform editor — it edits **named slots** inside a fixed template. The assembled full prompt appears in the live preview, but only the variable parts are editable via form fields.

A **"Manual Override"** toggle unlocks the full prompt textarea for direct editing and flags the record `manual_override: true`.

---

### Step 1 — Select Driver & Product

- **Team dropdown** → filters driver dropdown
- **Driver dropdown** → populates all form fields from `drivers.json` defaults, then overlays last saved record from `designs.json` if one exists
- **Product type** selector: T-Shirt / Hoodie / Cap / Tote (pill buttons)
- **Assigned to** badge shown (from `assignments.json`)
- If the active user is not the assigned person, show soft warning: _"This driver is assigned to [Name]."_

---

### Step 2 — Prompt Form Fields

All fields auto-populate from (in priority order): last saved `designs.json` record → `drivers.json` defaults.

| Field | Type | Token in prompt | Notes |
|---|---|---|---|
| First name | text (display only) | `{{FIRST_NAME}}` | Read from driver record |
| Last name | text | `{{LAST_NAME}}` | Editable |
| Driver number | number | `{{NUMBER}}` | Read from driver record |
| Nickname | text | `{{NICKNAME}}` | Hero text on back |
| Team name (back bottom text) | text | `{{TEAM_NAME}}` | Plain text only |
| Primary design color | color picker + text label | `{{PRIMARY_COLOR}}` | e.g. "Ferrari red" |
| Secondary design color | color picker + text label | `{{SECONDARY_COLOR}}` | e.g. "gold" |
| **T-shirt base color** | color picker + text label | _not injected into prompt_ | Stored as `tshirt_color` + `tshirt_color_name`. Internal tracking only. Clearly labeled: _"Base color — for internal use, not included in prompt"_ |
| Front chest badge description | textarea | `{{FRONT_BADGE}}` | Must be original fan art — no trademark language |
| Back graphic / unique element | textarea | `{{BACK_GRAPHIC}}` | Circuit layout, helmet outline, car silhouette, etc. |
| Championship years | text (comma-separated) | `{{CHAMP_YEARS}}` | Leave blank for non-champions |
| Style direction notes | textarea | `{{STYLE_NOTES}}` | Mood, heritage reference, contrast |
| Is championship edition | toggle | switches template | Changes which prompt template is used |

**Legal Compliance Checklist** (shown below form, must all be checked before saving):
- [ ] Front badge is original fan art — no trademarked shape reproduced
- [ ] No official logos or licensed marks anywhere in the prompt
- [ ] Team name is plain neutral sans-serif only — no official branding style described
- [ ] Back graphic uses original illustrated art — not a reproduction of official artwork
- [ ] Car silhouette (if used) has no livery colors or sponsor markings

The Save button is disabled until all 5 boxes are checked. Confirmation state stored as `legal_checklist_confirmed: true` on the record.

---

### Step 3 — Live Prompt Preview

- Large monospace read-only textarea, updates in real time as fields change
- Unfilled `{{tokens}}` highlighted in amber
- **"Copy Prompt"** — copies to clipboard, toast confirmation
- **"Save Prompt"** — writes/updates `designs.json`, increments `prompt_version`, sets `status: "prompt_ready"` if was `"draft"`
- **"Manual Override"** toggle — unlocks full textarea; flags `manual_override: true`; shows warning _"Manual mode: edits won't sync back to form fields"_

---

### Prompt Templates

#### Standard Driver Template
```
Flat lay product mockup of a black oversized boxy fit t-shirt (drop shoulders, wide sleeves, slightly cropped length) on a neutral gray background, showing front and back views side by side. Premium heavyweight cotton texture.

Front design: Minimal and clean motorsport aesthetic. On the left chest, small typography reading "{{FIRST_NAME}} {{LAST_NAME}}" — "{{FIRST_NAME}}" in white and "{{LAST_NAME}}" in {{PRIMARY_COLOR}}. Directly below in very small fine-print accent text: "{{NICKNAME}}" in gold italics. A thin horizontal accent line underneath in white and gold extending slightly outward. On the right chest, {{FRONT_BADGE}}. No collar (crew neck), slightly thick neckline ribbing.

Back design: Large bold "{{NUMBER}}" in {{PRIMARY_COLOR}}, centered and slightly oversized for streetwear impact. Above it, large spaced-out white text "{{NICKNAME}}" as the hero element. Diagonal racing streaks (one {{PRIMARY_COLOR}}, one {{SECONDARY_COLOR}}) slicing across the back dynamically. {{BACK_GRAPHIC}} overlapping the number slightly on one side. Bottom text: "{{TEAM_NAME}}" in plain neutral sans-serif white type, no logo, no official branding style, small {{PRIMARY_COLOR}} underline beneath.

Style direction: {{STYLE_NOTES}} Slightly oversized graphics scaled for streetwear proportions. Soft shadows, realistic fabric folds, premium heavy cotton feel. No official logos or licensed marks.
```

#### Championship Edition Template (`is_championship_edition = true`)
```
Flat lay product mockup of a black oversized boxy fit t-shirt (drop shoulders, wide sleeves, slightly cropped length) on a neutral gray background, showing front and back views side by side. Premium heavyweight cotton texture.

Front design: Minimal and clean motorsport aesthetic. On the left chest, small typography reading "{{LAST_NAME}} · {{NUMBER}}" — "{{LAST_NAME}}" in white and "{{NUMBER}}" in {{PRIMARY_COLOR}}. Directly below in very small fine-print accent text: "{{NICKNAME}}" in gold italics. A thin horizontal accent line underneath in white and gold extending slightly outward. On the right chest, {{FRONT_BADGE}}. No collar (crew neck), slightly thick neckline ribbing.

Back design: Large bold "1" in {{PRIMARY_COLOR}} with an original artistic gold champion's crown illustration integrated above the number — centered and slightly oversized for streetwear impact. Above the crown, large spaced-out white text "{{NICKNAME}}" as the hero element. Diagonal racing streaks (one {{PRIMARY_COLOR}}, one {{SECONDARY_COLOR}}) slicing across the back dynamically. {{BACK_GRAPHIC}} overlapping the number slightly on one side. Bottom text: "{{TEAM_NAME}}" in plain neutral sans-serif white type, no logo, no official branding style, small {{PRIMARY_COLOR}} underline beneath. Below that, championship years in gold spaced-out fine text: "{{CHAMP_YEARS}}".

Style direction: {{STYLE_NOTES}} Slightly oversized graphics scaled for streetwear proportions. Soft shadows, realistic fabric folds, premium heavy cotton feel. No official logos or licensed marks.
```

---

## Tab 2: Design Vault

### Filters / Controls
- **Assignee filter**: All / Ayush / Anagh / Vanshika (defaults to active user)
- **Status filter**: All / Draft / Prompt Ready / Partially Uploaded / Fully Uploaded / Approved
- **Team filter**: dropdown
- **Driver filter**: dropdown (scoped to team)
- **Completeness filter**: All / Missing front / Missing back / Missing print-ready

### Card Grid

Each card shows:
- Driver name + nickname badge
- Product type tag + t-shirt color swatch dot
- Assigned-to label
- Status pill (color coded)
- **Three image slot thumbnails** labeled `FRONT` / `BACK` / `PRINT READY`
  - Filled: thumbnail preview
  - Empty: dashed placeholder with upload icon
- "View / Edit" button → opens modal

**Status auto-computed:**
- `draft` — no prompt saved
- `prompt_ready` — prompt saved, 0 images
- `partially_uploaded` — 1 or 2 of 3 slots filled
- `fully_uploaded` — all 3 slots filled
- `approved` — manually marked (any user can approve)

---

### Upload Flow (per image slot)

Each slot (`front`, `back`, `print_ready`) is independently uploadable:

1. Click slot placeholder or thumbnail
2. File picker — PNG or JPG only; warn if > 5MB
3. Preview shown in modal
4. Optional: edit prompt note for this specific render
5. **"Upload"** → encode to base64, commit to `images/{driver_slug}/{slot}.png` via GitHub API, update `designs.json`

Replacing an existing image: show _"Replace existing image?"_ confirm before overwrite.

---

### Image View / Edit Modal

- Driver name, nickname, product type in header
- T-shirt color swatch + name displayed
- **Three image panels**: Front / Back / Print Ready — each with thumbnail, "Replace" button, "Download" button
- **Prompt used** — assembled prompt text, read-only; "Edit Prompt" button jumps to Tab 1 pre-filled
- **Prompt version** badge (e.g. "v3") + `manual_override` warning if set
- **Legal checklist status** — green checkmark or amber warning
- **Notes** — free text, editable inline, auto-saves on blur
- **"Mark Approved"** — sets `status: "approved"`, records approver + timestamp
- **"Download All"** — downloads all available images as a zip (use JSZip from CDN)

---

## GitHub API Operations

All via `https://api.github.com/repos/{owner}/{repo}/contents/{path}`

| Operation | API Call |
|---|---|
| Read JSON | GET → decode base64 content → JSON.parse |
| Write/Update JSON | GET SHA first → PUT with base64-encoded content + SHA |
| Upload image (new) | PUT with base64-encoded bytes, no SHA needed |
| Replace image | GET SHA first → PUT with SHA |
| Download image | GET → decode base64 → trigger browser download |

**Always fetch current SHA before any write** to prevent conflicts.

### Conflict Handling
Second concurrent PUT will fail (wrong SHA). Show error: _"Save conflict — someone else saved first. Refresh and re-apply your changes."_

---

## UI Design Direction

**Aesthetic**: Dark motorsport editorial — paddock data terminal crossed with streetwear brand lookbook.

- **Background**: `#0a0a0a` with subtle carbon-fiber texture via CSS repeating-linear-gradient
- **Primary accent**: `#E8002D` racing red — buttons, active states
- **Gold accent**: `#C9A84C` — status, championship elements, legal warnings
- **Typography**: `Orbitron` (Google Fonts) for headers/labels; `IBM Plex Mono` for prompt preview
- **Cards**: `#1a1a1a` bg, `1px #2a2a2a` border, red left-border on hover
- **Image slots**: Dashed `#333` when empty, solid `#2a2a2a` when filled
- **Status pills**:
  - Draft → `#444` gray
  - Prompt Ready → `#1d4ed8` blue
  - Partially Uploaded → `#b45309` amber
  - Fully Uploaded → `#15803d` green
  - Approved → `#7c3aed` purple
- **Legal warning callout**: Gold background `#C9A84C` with dark text, collapsible, always on Tab 1
- **Animations**: Slide-in on tab switch, slot pulse on successful upload, clipboard bounce on copy

---

## File to Build

**Single file: `index.html`**

- Tailwind via CDN `<script src="https://cdn.tailwindcss.com">`
- Google Fonts (`Orbitron`, `IBM Plex Mono`) via `<link>`
- JSZip via CDN (for "Download All")
- All JS in `<script type="module">`
- No npm, no build, no server — open directly in browser

---

## Implementation Notes for Claude Code

1. **Start with `db/drivers.json`** — seed all 22 2026 grid drivers + 4 championship edition drivers. Every `front_badge_description` and `back_graphic_description` must be legal-compliant (no trademark language).

2. **Build GitHub API module first** — `readFile(path)`, `writeFile(path, content, sha)`, `uploadImage(path, base64, sha?)`, `downloadImage(path)`. Test against the real repo before building UI.

3. **Prompt template system** — store both templates as JS template literals. Write an `assemblePrompt(fields, isChampionship)` function that substitutes all `{{tokens}}`. Unfilled tokens render as `[MISSING: TOKEN_NAME]` highlighted in amber in the preview.

4. **T-shirt color field** — displayed prominently in the form with label _"Base color — for internal tracking only, not included in prompt"_. Stored as `tshirt_color` (hex) + `tshirt_color_name` (text label) on the design record. The hex value drives a color swatch in the card and modal.

5. **Build Tab 1 first** — form → live preview → copy → save to `designs.json`.

6. **Build Tab 2 second** — card grid → per-slot upload → modal.

7. **PAT security**: `sessionStorage` only. Show disclaimer: _"Your PAT is stored in session memory only and is never sent anywhere except api.github.com."_

8. **Image size warning**: Toast if file > 5MB before upload.

9. **Offline fallback**: Cache last-fetched `designs.json` + `drivers.json` in `localStorage`. Show stale-data banner if GitHub API is unreachable.

10. **Legal checklist**: Disable Save button until all 5 items checked. Store `legal_checklist_confirmed: true` on record.

---

## Driver Data Seed (2026 Grid)

**Standard grid (22 drivers):**
Hamilton (Ferrari), Leclerc (Ferrari), Russell (Mercedes), Antonelli (Mercedes), Norris (McLaren), Piastri (McLaren), Verstappen (Red Bull), Lawson (Red Bull), Alonso (Aston Martin), Stroll (Aston Martin), Gasly (Alpine), Doohan (Alpine), Hulkenberg (Sauber), Bortoleto (Sauber), Albon (Williams), Sainz C. (Williams), Tsunoda (Racing Bulls), Hadjar (Racing Bulls), Ocon (Haas), Bearman (Haas), Lindblad (TBC), + 1 TBC seat.

**Championship edition (4 drivers, `is_championship_edition: true`):**
- Hamilton → Mercedes (last title), 7x, `"THE GOAT"`
- Verstappen → Red Bull Racing, 4x, `"SUPER MAX"`
- Alonso → Renault F1, 2x, `"EL PLAN"`
- Norris → McLaren, 1x, `"THE PAPERBOY"`

All badge/graphic descriptions in the seed data must be legal-compliant originals — review against the Legal Design Rules table above before finalizing.

---

## MVP Scope (build in this order)

- [ ] GitHub API connect + validate
- [ ] Load `drivers.json`, populate dropdowns
- [ ] Tab 1: Form fields with driver auto-population
- [ ] Tab 1: T-shirt color picker (non-prompt field, clearly labeled)
- [ ] Tab 1: Live prompt assembly with token substitution
- [ ] Tab 1: Unfilled token highlighting in preview
- [ ] Tab 1: Legal compliance checklist (blocks save)
- [ ] Tab 1: Copy prompt to clipboard
- [ ] Tab 1: Save prompt → `designs.json`
- [ ] Tab 1: Manual override toggle
- [ ] Tab 2: Card grid from `designs.json` with all filters
- [ ] Tab 2: Three-slot image display per card
- [ ] Tab 2: Per-slot independent upload → GitHub commit + record update
- [ ] Tab 2: Replace image confirmation
- [ ] Tab 2: View modal with all three images
- [ ] Tab 2: Download individual + Download all as zip
- [ ] Tab 2: Approve + notes
- [ ] Polish: animations, empty states, error toasts, offline fallback