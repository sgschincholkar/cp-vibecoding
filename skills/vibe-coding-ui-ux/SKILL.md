---
name: vibe-coding-ui-ux
description: |
  UI/UX design intelligence. Guides style selection across 67 UI styles, 161 color palettes,
  57 typography pairings, 25 chart types, and 99 UX guidelines.
  Use when: (1) vibe-coding-ideate reaches Topic 10 and user needs guidance,
  (2) User says "help me pick a design style", "suggest colors", "I don't know what style",
  (3) Before generating DESIGN_SYSTEM.md to ensure design direction is clear.
  Saves selections to progress.txt for vibe-coding-doc-design.
---

# Vibe Coding — UI/UX Design Intelligence

Guide visual design decisions. Output feeds directly into DESIGN_SYSTEM.md.

## Activation Router

Check progress.txt for project type. Check user message for keywords:

```
User says "skip" / "use defaults" / "just pick something"
→ READ: ## Skip Path (auto-select by project type)

User provides their own hex color ("my brand is #C0392B")
→ READ: ## Custom Color Expansion

Otherwise → proceed through Steps 1-5 below
```

**Jump directly to the matched path (## Skip Path or ## Custom Color Expansion). If proceeding through Steps 1-5, read one step at a time — do not read all steps before starting Step 1.**

---

## Step 1 — Style Direction

Show 8 categories, ask user to pick one (or describe their own):

**Modern / Clean:** Minimal | Neo-brutalist | Material | Flat 2.0
**Expressive:** Glassmorphism | Claymorphism | Neumorphism | Gradient-rich
**Professional:** Corporate | Finance/Banking | Healthcare | Legal/Consulting
**Playful / Consumer:** Candy/Gen-Z | Retro/Y2K | Cartoon/Illustrated | Organic
**Data / Technical:** Dashboard | Developer Tool | Analytics | AI-native
**Luxury / Premium:** Noir | Minimalist Luxury | Dark Premium | Editorial

Ask: "Which style direction resonates? Pick one or describe your own."

**Custom style handling:** If user describes a style not in the 8 categories (e.g., "steampunk", "brutalist minimalist mix", "vaporwave"):
1. Extract 3-5 keywords from their description
2. Find 2 closest catalog matches and ask: "Closer to [X] or [Y]?"
3. If still unique: create custom palette blend from their keywords
4. Document in progress.txt as `style: Custom — [description]`

**Mismatch warning:** If chosen style conflicts with project type, flag it:
"[Style] can work but may affect [credibility/legibility]. Consider [alternative] instead, or confirm to proceed."

---

## Step 2 — Color Palette Selection

Show 3 curated options based on project type:

**Technology / SaaS:** Ocean Depth (#2563EB/#7C3AED/#06B6D4) | Slate Modern (#0F172A/#334155/#F59E0B) | Midnight Gradient (#6366F1/#8B5CF6/#EC4899)
**Ecommerce:** Sunset Burst (#F97316/#EF4444/#FBBF24) | Cherry Pop (#E11D48/#F43F5E/#FB923C) | Luxury Black (#111827/#1F2937/#D4AF37)
**Health / Wellness:** Forest Calm (#166534/#15803D/#FDE68A) | Ocean Serenity (#0369A1/#0284C7/#BAE6FD) | Lavender Care (#7C3AED/#8B5CF6/#C4B5FD)
**Finance / Banking:** Trust Navy (#1E3A5F/#1E40AF/#10B981) | Gold Standard (#111827/#1F2937/#D4AF37) | Conservative Blue (#1D4ED8/#2563EB/#DBEAFE)
**Creative / Portfolio:** Mono Bold (#111827/#374151/#EF4444) | Dark Canvas (#0F172A/#1E293B/#F59E0B) | Vibrant Mix (#7C3AED/#6366F1/#10B981)
**Education / Non-profit:** Friendly Blue (#2563EB/#3B82F6/#FDE68A) | Growth Green (#047857/#059669/#FEF9C3)

Ask: "Which palette resonates? Or should I generate custom options from your brand colors?"

---

## Step 3 — Typography Pairing

Show 6 pairings based on chosen style:

**Modern / Clean:** Inter + Inter | Inter + Playfair Display | DM Sans + DM Serif Display
**Professional:** IBM Plex Sans + IBM Plex Serif | Source Sans + Merriweather | Roboto + Roboto Slab
**Expressive:** Outfit + Clash Display | Satoshi + Cabinet Grotesk | Space Grotesk + Space Mono
**Luxury:** Cormorant Garamond + Jost | Italiana + Lato | Libre Baskerville + Libre Franklin
**Playful:** Nunito + Nunito | Fredoka One + Poppins | Righteous + Open Sans

Ask: "Which typography pairing fits? Or describe the feel and I'll suggest the best match."

---

## Step 4 — Charts (if applicable)

Only ask if project includes dashboard or data features.

**Comparison:** Bar, Column, Grouped Bar, Stacked Bar
**Trend:** Line, Area, Sparkline, Multi-line
**Part-to-whole:** Pie, Donut, Treemap, Waffle
**Progress:** Gauge, Radial Bar, Progress Bar
**Tables:** Simple Table, Pivot Table, Sparkline Table

---

## Step 5 — Save Selections

Write to `progress.txt`:
```
UI_UX_SELECTIONS:
  style: [chosen style]
  style_description: [1 sentence]
  palette_name: [name]
  color_primary: #XXXXXX
  color_secondary: #XXXXXX
  color_accent: #XXXXXX
  font_body: [Font Name]
  font_heading: [Font Name or "same as body"]
  font_mono: JetBrains Mono
  typography_pairing_name: [pairing name]
  charts: [list if applicable]
  dark_mode: [enabled | disabled | system]
```

Announce: "Selections saved. vibe-coding-doc-design will use these for DESIGN_SYSTEM.md — no manual re-entry needed."

---

## Skip Path

Auto-select based on project type from progress.txt and confirm with user:

| Project Type | Style | Palette | Typography |
|-------------|-------|---------|------------|
| SaaS/App | Minimal | Ocean Depth | Inter + Inter |
| Ecommerce | Flat 2.0 | Sunset Burst | DM Sans + DM Serif Display |
| Dashboard | Dashboard | Slate Modern | Inter + Inter |
| Landing | Gradient-rich | Midnight Gradient | Outfit + Clash Display |
| Content/Blog | Minimal | Corporate Trust | Source Sans + Merriweather |
| Portfolio | Neo-brutalist | Mono Bold | Satoshi + Cabinet Grotesk |

Show proposal, ask: "Confirm these defaults? (yes / customize)"

---

## Custom Color Expansion

When user provides their own primary hex (e.g., `#C0392B`):

1. Primary: user's hex
2. Secondary: analogous color (±30 degrees on color wheel, similar lightness)
3. Accent: complementary (opposite on color wheel or triadic)
4. primary-hover: darken primary by 10-15%
5. primary-active: darken primary by 20-25%

Present as palette proposal with all hex values. Ask for approval before saving.
