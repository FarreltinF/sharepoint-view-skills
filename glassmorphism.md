#### name: "glassmorphism"

## Safety: Design-Only Skill

This skill applies **visual styling only** — it does NOT add, modify, or overwrite any existing columns or data in the library/list. It reads the current schema and applies column formatting and view formatting to the existing structure. No data is created, deleted, or changed.

- If columns referenced in the design (e.g., Status, Progress, Deadline) do not exist, skip formatting for those columns gracefully.
- Never call `update_list_items_v2` to modify existing item data.
- Never delete or rename existing columns.

## Empty Library Behavior

If the library/list has **no items**, create sample items to illustrate the design style:

### Sample Data (hardcoded)
Create 4 sample items with the following data (adapt column names to match existing schema):

| Title/Name | Status | Progress | Deadline | Notes |
|---|---|---|---|---|
| Q2 Marketing Plan | In Review | 72 | (2 weeks from today) | Updated targeting strategy · Added competitor analysis |
| Product Launch Brief | Draft | 25 | (1 week from today) | Initial outline complete · Pending stakeholder input |
| Engineering Roadmap | Approved | 95 | (3 weeks from today) | Final review passed · Ready for publication |
| Budget Forecast FY27 | Revising | 48 | (yesterday, to show overdue) | Revised cost projections · Awaiting finance sign-off |

Only create sample items if the library has zero items. If there are existing items, use them as-is and only apply formatting.

description: "Frosted glass effect with translucent layers, subtle blur, and luminous borders for depth and modern elegance. Applies glassmorphism design system styling to SharePoint list and library views. Use when the user asks for glassmorphism style, frosted glass design, glass UI, translucent/blur styling, or liquid glass effect on a view."

## Glassmorphism Design System for SharePoint

### Design Intent
Apply a modern glassmorphism aesthetic: translucent frosted-glass surfaces, soft luminous borders, airy frost shadows, clean high-contrast typography with depth and layering. Default is light frost mode — cool blue-gray surface with translucent frosted white glass layers on top.

### Design Tokens (Light Frost Glass — Default)

#### Colors (rgba only)
| Token | rgba | Use |
|-------|------|-----|
| Surface | rgba(241,245,249,1) | Cool frost blue-gray — cell background, page surface |
| CardBg | rgba(255,255,255,0.85) | Frosted white — popup backgrounds |
| Glass | rgba(255,255,255,0.65) | Translucent white — frosted card bg |
| GlassBorder | rgba(148,163,184,0.25) | Soft slate — frosted glass edge |
| Primary | rgba(59,130,246,1) | Vivid blue — highlights, links, headers |
| Secondary | rgba(147,51,234,1) | Vivid purple — Sparkle icon, Published badge |
| Success | rgba(22,163,74,1) | Deep green — approved, SortUp icon |
| Warning | rgba(217,119,6,1) | Deep amber — revising |
| Danger | rgba(220,38,38,1) | Vivid red — overdue, error |
| Text | rgba(15,23,42,1) | Dark slate — primary text |
| Muted | rgba(100,116,139,1) | Slate-500 — secondary text, bullet summaries |
| EmDash | rgba(203,213,225,1) | Slate-300 — empty cell indicator |
| StatusDraft | rgba(100,116,139,1) | Slate — Draft badge |

#### Typography
- Font: system stack (-apple-system, "Segoe UI", Roboto, sans-serif)
- Title/Project name: 13px weight 600, letter-spacing -0.2px
- Labels: 10px weight 700 uppercase letter-spacing 0.8px
- Body: 12-13px weight 400
- Card header: 11px weight 700
- Card body: 11px weight 400
- Show details link: 10px weight 600
- Deadline date: 12px weight 700, letter-spacing -0.3px
- Deadline label: 9px weight 700 uppercase, letter-spacing 0.8px

#### Borders and Shadows
- Column cards: 1px solid rgba(148,163,184,0.25), box-shadow 0px 4px 16px rgba(148,163,184,0.15)
- Badge/status pills: 1px solid rgba(148,163,184,0.25), box-shadow 0px 2px 8px rgba(148,163,184,0.12)
- Popup callout: border-radius 8px, bg rgba(255,255,255,0.85)
- Border radius: 8px on cards, 16px on pills/badges, 4px on small elements

### Critical: Light Frost Full Cell Background

Every column formatter MUST use the **margin/padding bleed technique** to fill the entire cell with the frost surface color:

```json
{
  "elmType": "div",
  "style": {
    "background-color": "rgba(241,245,249,1)",
    "margin": "-200px",
    "padding": "200px",
    "overflow": "hidden",
    "display": "flex",
    "align-items": "center"
  }
}
```

**Why -200px?** SharePoint cells have internal padding. A small negative margin only covers part of the cell, leaving default-colored strips. Using -200px with matching padding ensures the frost background extends well beyond the cell boundaries, clipped by `overflow: hidden`.

**This pattern must be on the root div of EVERY column formatter.**

### Component Patterns

#### Title/Project Column (Text)
Frost bg with dark text, no special formatting:
- Root: margin -200px, padding 200px, Surface bg rgba(241,245,249,1)
- Text: 13px weight 600, Text color rgba(15,23,42,1), letter-spacing -0.2px

#### Status Badge Mapping
Map Choice values from schema dynamically using expression syntax:
- Draft/Inactive → StatusDraft rgba(100,116,139,1) bg, white text
- In Review/In Progress → Primary rgba(59,130,246,1) bg, white text
- Revising/Warning → Warning rgba(217,119,6,1) bg, white text
- Approved/Success → Success rgba(22,163,74,1) bg, white text
- Published/Complete → Secondary rgba(147,51,234,1) bg, white text

Badge style: border-radius 16px, padding 2px 10px, font-size 12px, font-weight 600, 1px solid GlassBorder, box-shadow 0px 2px 8px rgba(148,163,184,0.12)

#### Progress Bar
- Container: Glass bg rgba(255,255,255,0.65), 1px solid GlassBorder, border-radius 4px, height 14px
- Fill: border-radius 4px
  - < 25%: Danger rgba(220,38,38,1)
  - 25-49%: Warning rgba(217,119,6,1)
  - 50-79%: Primary rgba(59,130,246,1)
  - >= 80%: Success rgba(22,163,74,1)
- Percentage text: 12px weight 700, Text color rgba(15,23,42,1)

#### Deadline (DateTime) — Frost Glass Icon Box Style
Frosted icon box with date text and label, overdue state uses soft frosted red tint:
- **Icon box**: 36x36px, border-radius 8px, box-shadow 0px 4px 16px rgba(148,163,184,0.15)
- **Normal state**: Glass bg rgba(255,255,255,0.65), 1px solid GlassBorder, Calendar icon in Primary blue rgba(59,130,246,1), bold date text in Text color, "DEADLINE" label in Muted
- **Overdue state** (`@currentField <= @now`): Frosted red tint bg rgba(220,38,38,0.12), 1px solid rgba(220,38,38,0.25), Warning icon in rgba(220,38,38,1), date text in rgba(220,38,38,0.85), "OVERDUE" label in rgba(220,38,38,0.7)
- **Empty state**: content hidden via child div `display: none` (root always renders frost)
- **Design note**: Overdue uses soft frosted red instead of solid red block — stays aligned with the frost glass aesthetic

#### Note/Week Card (Multiline Text)
Each Note column cell renders as a frost glassmorphism card:
- **Empty cells**: centered em dash (—) in EmDash color rgba(203,213,225,1)
- **Active cells**: frosted glass card with 3 sections:
  - **Header row**: Green SortUp icon + bold header text in Primary blue
  - **Summary row**: Purple Sparkle icon + Muted body text, truncated at 32px max-height
  - **Show details link**: Primary blue text on `span` with `customCardProps` popup
- Card styling: Glass bg rgba(255,255,255,0.65), 1px solid GlassBorder, border-radius 8px, soft frost shadow
- Popup styling: CardBg bg rgba(255,255,255,0.85), border-radius 8px, Text color rgba(15,23,42,1), pre-wrap whitespace

### Workflow

1. **Get schema** — call `get_list_schema` to discover columns
2. **Map columns to roles**: Name (FileLeafRef/Title), Status (Choice), Progress (Number), People (User), Date (DateTime), Notes (Note)
3. **Build column formatters** — create a JSON formatter for each column using light frost glass tokens and expression syntax
4. **Apply ALL formatters via `editSchemaXml`** — embed the formatter JSON as a `CustomFormatter` attribute on the `<Field>` tag. Apply to EVERY column including Title and all Note columns.
5. **Every formatter root div** must use `margin: -200px`, `padding: 200px`, `background-color: rgba(241,245,249,1)`, `overflow: hidden`
6. **Create view** via `create_or_update_list` — include all referenced columns in viewFields

### Critical: How to Persist Column Formatting

**The `customFormatter` JSON property on column definitions is silently dropped by `create_or_update_list`.** It does NOT work.

**The ONLY method that persists** is embedding `CustomFormatter` as an XML attribute inside `editSchemaXml` on the `<Field>` tag.

**Escaping rules for CustomFormatter inside editSchemaXml:**
- Use `&quot;` for double quotes inside the JSON
- Use `&apos;` for single quotes inside expression strings
- Use `&lt;` and `&gt;` for angle brackets in expressions
- Use expression syntax (`=if(...)`) — much more compact than operator JSON

### Known Limitation: Name Column in Document Libraries

The **Name column** (`FileLeafRef` / `LinkFilename`) in document libraries is a **system computed field** with `NoCustomize="TRUE"`. It **cannot** have `CustomFormatter` applied. For **lists**, the Title column CAN be formatted (and should be styled with frost bg + dark text).

### Rules
- MUST use `editSchemaXml` with `CustomFormatter` attribute — never `customFormatter` JSON property
- MUST use expression syntax (`=if(...)`) for compact formatters
- MUST use rgba() colors — never rgb() or hex
- MUST use div/span/a — never button
- MUST use `margin: -200px` + `padding: 200px` + `overflow: hidden` on every root div for full frost cell bleed
- MUST use border-radius 8px on cards, 16px on pills
- MUST use soft frost shadows with rgba(148,163,184,0.15)
- MUST use vivid accent colors for strong contrast on light frost surfaces
- MUST apply formatters to ALL columns including Title — not just data columns
- `customCardProps` ONLY works on `span` elements — not `div`
- Name column CANNOT be formatted in document libraries — leave as default

## EVAL

### Test Data Requirements
- **Type:** list items
- **Minimum count:** 3
- **Schema needed:** Title/FileLeafRef, Status (Choice), Progress (Number 0-100), Deadline (DateTime DateOnly), Note columns (at least 3)
- **Sample data:** Items with varied statuses and progress values. Note columns with multiline text. Some cells empty for em dash test. Some deadlines in the past for overdue test.

### Expected Results
- **Result 1:** ALL cells have light frost background rgba(241,245,249,1) — no default white gaps
- **Result 2:** Title column shows dark text on frost surface
- **Result 3:** Status badges with correct vivid color mapping on frost surface
- **Result 4:** Progress bars with frosted glass container, vivid color-coded fill
- **Result 5:** Deadline column fully frosted — blue icon on glass box, overdue uses soft frosted red tint (not solid red)
- **Result 6:** Note/Week columns with frosted glass cards — translucent white visible against frost surface
- **Result 7:** Empty cells show centered em dash (—) in slate-300 on frost surface
- **Result 8:** "Show details" popup has frosted white bg with dark text
- **Result 9:** Formatting persists across page refreshes
- **Result 10:** No default white cell backgrounds anywhere — verified on all column types
- **UI-driven:** yes — frost glass visual appearance, cell bleed coverage, and contrast require visual confirmation
