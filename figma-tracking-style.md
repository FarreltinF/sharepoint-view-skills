# Figma Tracking Style

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

## Description
Creates a native SharePoint library view that displays documents as rows and the last 5 weeks as columns, forming a matrix where you can see exactly what changed for each document during each week. Uses Note columns with column formatting to render styled change cards. Use when the user asks to "track weekly updates", "show weekly document changes", "create a week-by-week view", "generate a document update matrix", or "show me what changed each week". Also applies when user says "apply figma style", "style my library", or "apply tracking style".

## Prerequisites
- The document library must have version history enabled.
- Works best with libraries that have Status, Progress, and Assigned To columns, but adapts to whatever metadata is available.

## Execution Steps

### Step 1: Discover Library Schema
Call `get_list_schema` on the current document library to identify:
- Available columns (especially Status, Progress, Assigned To, or equivalents)
- Version history settings
- Document count

### Step 2: List Documents
Call `list_items` to get all documents in the library. Collect:
- File name, ID, spItemUrl, serverRelativeUrl
- Current metadata values (status, progress, assigned to)

If more than 20 documents, ask the user to select which ones to track or use the most recently modified 6-10.

### Step 3: Get Version History
For each document, call `list_item_versions` to retrieve version history. Collect:
- Version number, timestamp, editor
- All metadata field values at each version

### Step 4: Compute Week Buckets
Call `execute_code` to process the version data:
- **Week boundaries**: Monday 00:00 to Sunday 23:59 (ISO weeks)
- **Week key format**: `YYYY-WNN` (e.g., `2026-W16`)
- **Change detection**: Compare first and last version within each week per document
- **Progress delta**: Calculate difference if Progress column exists — show as "fromX% → toY%" baseline format
- **Status transitions**: Track if status changed within the week — include in header line as "OldStatus → NewStatus"
- **Change count**: Number of versions saved during that week
- Select the **last 5 weeks** with any activity, sorted oldest-to-newest (left-to-right)

### Step 5: Generate Summaries
For each active cell (document x week with changes), generate bullet point changes:
- One bullet (•) per individual change
- Keep each bullet short (~30-50 chars) to fit within 255-char cell limit
- Number of bullets should match the change count in the header

### Step 6: Create Week Columns and Deadline Column
Call `create_or_update_list` to add columns:
- **5 Note columns** (plain text, no rich text) for week data
  - Display names use date ranges: e.g., "Apr 20 - Apr 26"
  - Add a star (★) to the current week's display name
  - Internal names: Week1 (newest) through Week5 (oldest)
  - Column order in view: Week5 (oldest, leftmost) → Week1 (newest, rightmost)
- **Deadline column** (DateTime, DateOnly format) for document deadlines

**Safety note:** If the library already has week/period Note columns, reuse them. Only create new columns if no Note columns exist AND the library has version history data to populate them.

### Step 7: Populate Week Column Data
Call `update_list_items_v2` to set each document's week column values.

**Safety note:** Only populate week columns with version history data if the columns were newly created in Step 6. Never overwrite existing Note column data.

**Data format** — plain text with newlines:
```
Line 1: 64% → 72% · Draft → In Review · 3 changes
• Updated OAuth2 flow docs
• Added rate limit headers
• Fixed /users response schema
```

Line 1 is the header: progress baseline (from% → to%) + optional status transition + change count.
Subsequent lines are bullet points (•) for each individual change.
Empty string for weeks with no activity.

**IMPORTANT**: Note fields have a 255-character limit via the update API. Keep total cell content under 255 characters. Use short bullet descriptions (~30-50 chars each).

### Step 8: Create the View
Call `create_or_update_list` to create a new view named "Weekly Update Tracking" with fields:
- `DocIcon`, `LinkFilename`, `Status`, `Progress`, `Deadline`, `Week5`, `Week4`, `Week3`, `Week2`, `Week1`

**IMPORTANT**: Do NOT include `FileLeafRef` — it duplicates `LinkFilename`.

### Step 9: Apply Column Formatting

#### Status Column
Colored pill badges with **solid colored backgrounds and white text** + **inline editing** — clicking the badge opens a dropdown to change the status value.
- Add `"inlineEditField": "@currentField"` on the root div (outermost element)
- Add `"cursor": "pointer"` on the root div
- **Alignment:** Root div must use `min-height: 36px`, `padding: 0 8px`, `display: flex`, `align-items: center` to align vertically with Progress and Deadline columns
- **Color scheme — solid backgrounds with white text (must match week column status pills exactly):**
  - Draft: `rgba(100,116,139,1)` bg, white text (slate)
  - In Review: `rgba(59,130,246,1)` bg, white text (blue)
  - Revising: `rgba(245,158,11,1)` bg, white text (amber)
  - Approved: `rgba(16,185,129,1)` bg, white text (emerald)
  - Published: `rgba(168,85,247,1)` bg, white text (purple)
- **CRITICAL:** Do NOT use light background + colored text for Status pills. They must use the same solid colored backgrounds as the week column transition pills for visual consistency.

#### Progress Column
Mini progress bar with colored fill + **inline editing**.
- Add `"inlineEditField": "@currentField"` on the root div
- **Alignment:** Root div must use `min-height: 36px`, `padding: 0 8px`, `display: flex`, `align-items: center` to align vertically with Status and Deadline columns
- ≥80%: green `rgba(16,185,129,1)`
- ≥50%: blue `rgba(59,130,246,1)`
- ≥25%: amber `rgba(245,158,11,1)`
- <25%: red `rgba(239,68,68,1)`
- Percentage text next to bar

#### Deadline Column
Calendar icon with date + **inline editing**. Turns red with warning icon when overdue.
- Add `"inlineEditField": "@currentField"` on the root div
- **Alignment:** Root div must use `min-height: 36px`, `padding: 0 8px`, `display: flex`, `align-items: center` to align vertically with Status and Progress columns
- **CRITICAL empty check:** Use `toString(@currentField) == ''` (NOT `@currentField == ''`) to detect empty DateTime fields. Hide ALL elements when empty.
- Only evaluate `@currentField <= @now` when `toString(@currentField)` is non-empty.

#### Week Columns (all 5 use the same formatter)
Each cell renders as:
- **Empty cells**: centered em dash (—) in `rgba(203,213,225,1)`
- **Active cells**: blue card with 4 sections:

**Section 1 — Progress header row:**
Green SortUp icon + bold blue header text showing ONLY the changes + progress portion of line 1. The header MUST strip the status transition text. Use `indexOf(@currentField, ' · Draft →')` (and similar for each status) to truncate. This prevents duplication with the status pill row below.

**Section 2 — Status transition row (conditional):**
Only shown when cell contains a status keyword before `→` (e.g., `Draft →`, `Review →`, `Revising →`, `Approved →`, `Published →`).
Uses abbreviated colored pills with **solid colored backgrounds and white text**:
- Draft: slate `rgba(100,116,139,1)` bg, white text
- In (for In Review): blue `rgba(59,130,246,1)` bg, white text
- Revise (for Revising): amber `rgba(245,158,11,1)` bg, white text
- Appr (for Approved): emerald `rgba(16,185,129,1)` bg, white text
- Pub (for Published): purple `rgba(168,85,247,1)` bg, white text
Format: `[FromPill] → [ToPill]` — BOTH pills must show the actual status abbreviation with the correct solid colored background.
**CRITICAL**: Check for specific status keywords (e.g., `indexOf(@currentField, 'Draft →')`) not just `→` alone — the progress line also contains `→`.

**Section 3 — Summary/bullets:**
Sparkle icon + grey text showing the bullet point changes, truncated at 32px max-height.

**Section 4 — Show details popup:**
Blue "Show details" text span with `customCardProps` popup on click showing full `@currentField` text with `white-space: pre-wrap` to preserve bullet points and newlines.

**Show details popup** uses `customCardProps` on a `span` element:
```json
{
  "elmType": "span",
  "txtContent": "Show details",
  "style": { "color": "rgba(59,130,246,1)", "font-size": "10px", "font-weight": "600", "cursor": "pointer", "padding-top": "2px" },
  "customCardProps": {
    "openOnEvent": "click",
    "directionalHint": "bottomCenter",
    "isBeakVisible": true,
    "formatter": {
      "elmType": "div",
      "style": { "padding": "12px", "max-width": "300px", "white-space": "pre-wrap", "font-size": "12px", "color": "rgba(51,65,85,1)", "line-height": "1.5" },
      "txtContent": "@currentField"
    }
  }
}
```

**Critical bounding rules** (prevents cards from overflowing cells):
- Root container: `overflow: hidden`, `box-sizing: border-box`, `max-width: 100%`
- Card div: `overflow: hidden`, `box-sizing: border-box`, `max-width: 100%`, `width: 100%`
- Header text: `overflow: hidden`, `text-overflow: ellipsis`, `white-space: nowrap`, `min-width: 0`
- Body text: `overflow: hidden`, `text-overflow: ellipsis`, `word-break: break-word`, `max-height: 32px`

**Critical Note field formatting constraints** (learned from testing):
- `customCardProps` ONLY works on `span` elements with simple `@currentField` — complex substring expressions inside the popup formatter break rendering
- `defaultHoverField` breaks Note field formatters entirely
- `button` elements with `customRowAction` break Note field formatters entirely
- `customCardProps` on `div` elements breaks Note field formatters entirely
- Always test formatter changes on ONE column first before applying to all 5

### Step 9.5: Format All Multi-line (Note) Columns with Blue Box Style

After applying week column formatting, scan the library schema for **any additional Note columns** (fieldType "Note") that are NOT the week columns already formatted. For each such column, apply the **blue box formatter** — the same visual card style used by week columns, but simplified (no progress/status parsing).

**Generic Note column blue box formatter:**
- **Empty cells**: centered em dash (—) in `rgba(203,213,225,1)`
- **Active cells**: blue card container with:
  - Background: `rgba(239,246,255,1)`
  - Border: `1px solid rgba(191,219,254,1)`
  - Border-radius: `6px`
  - Padding: `6px 8px`
  - Text displayed in `rgba(30,64,175,1)`, font-size `11px`, font-weight `600`
  - Text truncated with `overflow: hidden`, `text-overflow: ellipsis`
  - "Show details" popup span for full content on click

**Generic Note formatter JSON pattern:**
```json
{
  "$schema": "https://developer.microsoft.com/json-schemas/sp/v2/column-formatting.schema.json",
  "elmType": "div",
  "style": {
    "overflow": "hidden",
    "box-sizing": "border-box",
    "max-width": "100%",
    "padding": "4px"
  },
  "children": [
    {
      "elmType": "div",
      "style": {
        "display": "=if(@currentField == '', 'flex', 'none')",
        "justify-content": "center",
        "align-items": "center",
        "color": "rgba(203,213,225,1)",
        "font-size": "16px",
        "font-weight": "400",
        "min-height": "40px"
      },
      "txtContent": "—"
    },
    {
      "elmType": "div",
      "style": {
        "display": "=if(@currentField == '', 'none', 'flex')",
        "flex-direction": "column",
        "gap": "4px",
        "background-color": "rgba(239,246,255,1)",
        "border": "1px solid rgba(191,219,254,1)",
        "border-radius": "6px",
        "padding": "6px 8px",
        "overflow": "hidden",
        "box-sizing": "border-box",
        "max-width": "100%",
        "width": "100%"
      },
      "children": [
        {
          "elmType": "span",
          "style": {
            "font-size": "11px",
            "font-weight": "600",
            "color": "rgba(30,64,175,1)",
            "overflow": "hidden",
            "text-overflow": "ellipsis",
            "white-space": "nowrap",
            "min-width": "0"
          },
          "txtContent": "=if(indexOf(@currentField, '\n') > 0, substring(@currentField, 0, indexOf(@currentField, '\n')), @currentField)"
        },
        {
          "elmType": "span",
          "txtContent": "Show details",
          "style": {
            "color": "rgba(59,130,246,1)",
            "font-size": "10px",
            "font-weight": "600",
            "cursor": "pointer",
            "padding-top": "2px"
          },
          "customCardProps": {
            "openOnEvent": "click",
            "directionalHint": "bottomCenter",
            "isBeakVisible": true,
            "formatter": {
              "elmType": "div",
              "style": {
                "padding": "12px",
                "max-width": "300px",
                "white-space": "pre-wrap",
                "font-size": "12px",
                "color": "rgba(51,65,85,1)",
                "line-height": "1.5"
              },
              "txtContent": "@currentField"
            }
          }
        }
      ]
    }
  ]
}
```

**CRITICAL**: Only apply this formatter to Note columns that do NOT already have custom formatting from the week columns step. Check the internal name — skip any columns already formatted (e.g., Week1-Week5 / Period1-Period5).

### Step 10: Apply View Formatting
Apply alternating row backgrounds for readability:
```json
{
  "$schema": "https://developer.microsoft.com/json-schemas/sp/v2/row-formatting.schema.json",
  "additionalRowClass": "=if(@rowIndex % 2 == 0, 'ms-bgColor-neutralLighter', '')",
  "hideSelection": false
}
```

### Step 11: Navigate and Summarize
- Navigate the user to the new view
- Summarize: how many documents tracked, how many weeks covered, any documents with no recent activity

## Adaptations
- If the library has no Status column, omit status badges and status transitions in cell text
- If no Progress column, omit progress bars and progress deltas in cell text
- If fewer than 5 weeks of history, show only available weeks
- If a document has no versions in the time range, show all empty cells
- Map library-specific status values to the closest badge color; use slate for unknown statuses

## Key Design Decisions
- **Plain text in Note columns** — not JSON. SharePoint column formatting can reliably split plain text on newlines but struggles with JSON parsing via indexOf/substring on Note fields.
- **No FileLeafRef in view** — use DocIcon + LinkFilename to avoid duplicate Name columns.
- **Week order**: oldest on left, newest on right (natural reading direction for chronological data).
- **Star marker** (★) on current week column display name for quick identification.
- **Bounding constraints** — all card elements use `overflow: hidden`, `box-sizing: border-box`, and `max-width: 100%` to prevent blue cards from overflowing column cells.
- **Inline editing on Status** — uses `inlineEditField` so users can click the badge to change status directly.
- **Show details popup** — uses `customCardProps` on a `span` with simple `@currentField` reference. Complex expressions inside popup formatters break Note field rendering.
- **Progress baseline** — shows "from% → to%" format so users see where progress started and ended each week.
- **Deadline column** — DateTime field with overdue highlighting (red + warning icon when past due).
- **255-char cell limit** — Note fields via update API cap at 255 chars. Use short bullet points (~30-50 chars each).
- **Bullet point changes** — each change is a separate bullet (•) matching the change count. Visible in truncated card and fully readable via "Show details" popup.
- **Status transition pills** — abbreviated colored labels with solid colored backgrounds and white text (Draft, In, Revise, Appr, Pub) shown only when a status keyword precedes `→`. Hidden when no status change occurred (avoids false trigger from progress `→`).
- **Header strips status transition** — the header line only shows changes + progress, NOT the status transition. Status is shown as pills on the row below. Prevents duplication.
- **Column alignment** — Status, Progress, and Deadline formatters all use `min-height: 36px`, `padding: 0 8px`, `align-items: center` for consistent vertical alignment across rows.
- **Color consistency** — Status column pills use the same solid colored backgrounds as the week column status transition pills. Never use light bg + colored text for Status — always solid bg + white text.

- **Blue box on all Note columns** — every multi-line text column gets the blue card treatment automatically, not just week columns. This ensures visual consistency across the library. Generic Note columns use a simplified formatter (no progress/status parsing) but the same blue background, border, and "Show details" popup.

## EVAL

### Test Data Requirements
- **Type:** documents
- **Minimum count:** 6
- **Schema/columns needed:** Name (file), Status (choice: Draft/In Review/Revising/Approved/Published), Progress (number 0-100), Assigned To (person), Deadline (DateTime DateOnly)
- **Sample data description:** 6 documents with varying statuses, progress values, and deadlines. Week columns populated with header line + bullet point changes. Some cells should have status transitions, others should not. Some cells empty for em dash test. Some deadlines in the past for overdue test.

### Expected Results
- **Result 1:** A "Weekly Update Tracking" view is created with DocIcon, LinkFilename, Status, Progress, Deadline, and 5 week columns — no duplicate Name column
- **Result 2:** Active week cells show blue cards with green up-arrow, bold progress baseline header, optional status transition pills, truncated bullet points, and "Show details" link
- **Result 3:** Status transition row only appears on cells with actual status changes — not on cells that only have progress `→`
- **Result 4:** Clicking "Show details" opens a popup callout with all bullet points fully visible
- **Result 5:** Empty week cells show a centered em dash (—) in light grey
- **Result 6:** Status column shows colored pill badges; clicking opens inline edit dropdown
- **Result 7:** Progress column shows a mini progress bar with percentage
- **Result 8:** Deadline column shows calendar icon + date; overdue dates show red with warning icon
- **Result 9:** Alternating row backgrounds applied for readability
- **Result 10:** Blue cards stay bounded within column width; text truncates properly
- **UI-driven:** yes — visual layout, card rendering, popup behavior, status pills, bullet points, inline editing, and overdue highlighting require visual confirmation
