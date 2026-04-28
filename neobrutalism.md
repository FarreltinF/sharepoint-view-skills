#### name: "neobrutalism"

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

description: "Modern take on brutalism with bold borders, vivid accent colors, and raw, high-contrast layouts on warm surfaces. Applies neobrutalism design system styling to SharePoint list and library views. Use when the user asks for neobrutalism style, brutalist design, or bold/chunky/raw UI styling on a view."

## Neobrutalism Design System for SharePoint

### Design Intent
Apply a modern neobrutalist aesthetic: thick dark borders, vivid flat colors, offset shadows, zero border-radius, high-contrast typography on warm cream surfaces.

### Design Tokens

#### Colors (rgba only)
| Token | rgba | Use |
|-------|------|-----|
| Primary | rgba(253,200,0,1) | Bold yellow — highlights, Sparkle icon, active |
| Secondary | rgba(67,45,215,1) | Electric purple — "Show details" link, in progress |
| Success | rgba(22,163,74,1) | Green — approved, complete, SortUp icon |
| Warning | rgba(217,119,6,1) | Amber — revising, caution |
| Danger | rgba(220,38,38,1) | Red — overdue, error |
| Surface | rgba(251,251,249,1) | Warm off-white background |
| Text | rgba(28,41,60,1) | Dark text, borders, and shadows |
| White | rgba(255,255,255,1) | Card background |
| Muted | rgba(100,116,139,1) | Slate — secondary text, bullet summaries |

#### Typography
- Font: system stack (-apple-system, "Segoe UI", Roboto, sans-serif)
- Title: 16px weight 800
- Labels: 10px weight 700 uppercase letter-spacing 0.8px
- Body: 12-13px weight 400
- Card header: 11px weight 700
- Card body: 11px weight 400
- Show details link: 10px weight 600

#### Borders and Shadows
- Column cards: 2px solid Text, shadow 2px 2px 0px Text
- Badge/elements: 2px solid Text
- Popup callout: 2px solid Text, 0px border-radius
- Border radius: 0px everywhere — no exceptions

### Component Patterns

#### Status Badge Mapping
Map Choice values from schema dynamically using expression syntax:
- Draft/Inactive → Muted bg, white text
- In Review/In Progress → Secondary bg, white text
- Revising/Warning → Warning bg, white text
- Approved/Success → Success bg, white text
- Published/Complete → Primary bg, dark text (WCAG contrast)

#### Progress Bar
- Container: 2px solid Text, 0px border-radius, height 14px
- Fill: < 50% Danger, 50-79% Primary, >= 80% Success
- Percentage text: 12px weight 800

#### Deadline (DateTime) — Icon Box Style
Square icon box with date text and label, overdue state changes entire box to red:
- **Icon box**: 36x36px, 2px solid Text border, 2px 2px 0px Text offset shadow, 0px border-radius
- **Normal state**: white bg, Calendar icon in Text color, bold date text, "DEADLINE" label in Muted
- **Overdue state** (`@currentField <= @now`): Danger red bg, Warning icon in white, red date text, "OVERDUE" label in Danger
- **Empty state**: hidden entirely (`display: none` when `toString(@currentField) == ''`)
- **Inline editing** enabled via `inlineEditField`
- Date text: 12px weight 800, letter-spacing -0.3px
- Label text: 9px weight 700 uppercase, letter-spacing 0.8px

#### Note/Week Card (Multiline Text)
Each Note column cell renders as a neobrutalism card:
- **Empty cells**: centered em dash (—) in `rgba(203,213,225,1)`
- **Active cells**: white card with 3 sections:
  - **Header row**: Green SortUp icon + bold header text (first line before `\n`)
  - **Summary row**: Yellow Sparkle icon + muted body text (lines after `\n`, truncated at 32px max-height)
  - **Show details link**: Purple "Show details" text on a `span` with `customCardProps` popup
- Card styling: 2px solid Text border, 2px 2px 0px Text shadow, 0px border-radius, white bg
- Popup styling: 2px solid Text border, 0px border-radius, pre-wrap whitespace, 300px max-width

### Workflow

1. **Get schema** — call `get_list_schema` to discover columns
2. **Map columns to roles**: Name (FileLeafRef/Title), Status (Choice), Progress (Number), People (User), Date (DateTime), Notes (Note)
3. **Build column formatters** — create a JSON formatter for each column using neobrutalism tokens and expression syntax
4. **Apply ALL formatters via `editSchemaXml`** — embed the formatter JSON as a `CustomFormatter` attribute on the `<Field>` tag inside `editSchemaXml`. Apply to EVERY styled column including all Note/Week columns.
5. **Create view** via `create_or_update_list` — include all referenced columns in viewFields

### ⚠️ Critical: How to Persist Column Formatting

**The `customFormatter` JSON property on column definitions is silently dropped by `create_or_update_list`.** It does NOT work.

**The ONLY method that persists** is embedding `CustomFormatter` as an XML attribute inside `editSchemaXml` on the `<Field>` tag:

```json
{
  "internalName": "Status",
  "displayName": "Status",
  "type": "Choice",
  "editSchemaXml": "<Field InternalName='Status' Type='Choice' DisplayName='Status' CustomFormatter='{...escaped JSON...}' />"
}
```

**Escaping rules for CustomFormatter inside editSchemaXml:**
- The formatter JSON goes inside a single-quoted XML attribute value
- Use `&quot;` for double quotes inside the JSON
- Use `&apos;` for single quotes inside expression strings (e.g., `=if(@currentField == &apos;Draft&apos;, ...)`)
- Use `&lt;` and `&gt;` for angle brackets in expressions
- Use expression syntax (`=if(...)`) instead of nested operator/operands JSON — it's shorter and fits within XML attribute limits

**Expression syntax is preferred over operator/operands JSON** because it's much more compact:
```
GOOD (expression): "=if(@currentField == 'Draft', 'rgba(100,116,139,1)', if(@currentField == 'In Review', 'rgba(67,45,215,1)', 'rgba(251,251,249,1)'))"
BAD (operator JSON): {"operator":"?","operands":[{"operator":"==","operands":["@currentField","Draft"]},"rgba(100,116,139,1)",{...nested...}]}
```

### ⚠️ Known Limitation: Name Column in Document Libraries

The **Name column** (`FileLeafRef` / `LinkFilename`) in document libraries is a **system computed field** with `NoCustomize="TRUE"`. It **cannot** have `CustomFormatter` applied via `editSchemaXml` or any other API method.

**What was tested and failed:**
- `editSchemaXml` on `FileLeafRef` (Type=File) → error: "can't be changed to the type Text"
- `editSchemaXml` on `LinkFilename` (computed field) → error: "Cannot complete this action"
- Both are system fields that SharePoint protects from schema modification

**Workaround options:**
- Accept the default Name column rendering — it still shows the document name with standard SharePoint styling
- For **lists** (not document libraries), the `Title` column CAN be formatted via `editSchemaXml`
- For a fully styled Name column in document libraries, use `preview_view_changes` with a `rowFormatter` — but this is **temporary** and will override per-column formatters

**Recommendation:** Leave the Name column as-is. The neobrutalism style is applied to all other columns (Status, Progress, Deadline, Week columns). The Name column uses SharePoint's default rendering which still looks clean alongside the styled columns.

### View-Level Formatting (rowFormatter)

View-level `rowFormatter` via `<CustomFormatter>` in ViewXml:
- **Can be previewed** via `preview_view_changes` — renders live in the user's current view
- **Cannot be persisted** via `create_or_update_list` — the `customFormatter` property on views is silently dropped
- Use `preview_view_changes` for live demos of full card layouts
- For permanent styling, use per-column formatters via `editSchemaXml`
- **Warning**: Using rowFormatter overrides per-column formatters — they won't render if a rowFormatter is active

### Column Formatter Templates

#### Status Column (Choice)
Uppercase pill badge with thick border, inline editing enabled:
```
editSchemaXml: <Field InternalName='Status' Type='Choice' DisplayName='Status'
  CustomFormatter='{formatter with inlineEditField, 2px border, 0px border-radius,
  color mapping per status value using expression syntax}' />
```

#### Progress Column (Number)
Chunky progress bar with 2px border container, color-coded fill, percentage text:
```
editSchemaXml: <Field InternalName='Progress' Type='Number' DisplayName='Progress'
  CustomFormatter='{formatter with 14px bar, 2px solid border, 0px border-radius,
  red <50 / yellow 50-79 / green >=80}' />
```

#### Deadline Column (DateTime) — VERIFIED WORKING
Square icon box with date and label. Overdue flips to red Warning box.

Structure:
1. **Root div**: flex row, gap 6px, min-height 36px, hidden when empty, inlineEditField enabled
2. **Icon box div**: 36x36px, 2px solid Text border, 2px offset shadow, 0px border-radius
   - Normal: white bg, Calendar icon in Text color
   - Overdue: Danger red bg, Warning icon in white
3. **Text div**: flex column
   - Date text: `@currentField.displayValue`, 12px weight 800, Text/Danger color
   - Label: "DEADLINE" or "OVERDUE", 9px weight 700 uppercase, Muted/Danger color

```
editSchemaXml: <Field InternalName='Deadline' Type='DateTime' DisplayName='Deadline'
  CustomFormatter='{icon box 36x36 with 2px border + shadow, Calendar/Warning icon,
  date text + DEADLINE/OVERDUE label, inlineEditField, hidden when empty}' />
```

#### Note/Week Columns (Note) — VERIFIED WORKING
Cards with thick borders, offset shadows, header/body split, and "Show details" popup.

Structure of each card:
1. **Root div**: overflow hidden, box-sizing border-box, min-height 70px, centered
2. **Empty state div**: shows em dash (—) when `@currentField == ''`
3. **Card div**: shows when not empty — 2px solid Text border, 2px 2px 0px Text shadow, 0px border-radius, white bg
   - **Header div**: SortUp icon (Success green) + bold first line via `=substring(@currentField, 0, indexOf(@currentField, '\n'))`
   - **Body div**: Sparkle icon (Primary yellow) + muted text via `=substring(@currentField, indexOf(@currentField, '\n') + 1, indexOf(@currentField, '\n') + 200)` — max-height 32px, overflow hidden
   - **Show details span**: "Show details" text in Secondary purple, `customCardProps` with click popup showing `@currentField` with pre-wrap in a bordered callout

**Critical Note column formatting rules:**
- `customCardProps` ONLY works on `span` elements — never `div`
- Popup `formatter.txtContent` must be simple `@currentField` — complex expressions break rendering
- `defaultHoverField` breaks Note field formatters entirely
- `button` elements with `customRowAction` break Note field formatters entirely
- All card elements must use `overflow: hidden`, `box-sizing: border-box`, `max-width: 100%` to stay bounded within cells

#### Name Column (FileLeafRef / LinkFilename) — CANNOT BE FORMATTED
System computed field in document libraries. `NoCustomize="TRUE"` prevents any `CustomFormatter` via API. See "Known Limitation" section above.

### Rules
- MUST use `editSchemaXml` with `CustomFormatter` attribute — never `customFormatter` JSON property
- MUST use expression syntax (`=if(...)`) for compact formatters that fit in XML attributes
- MUST use rgba() colors — never rgb() or hex
- MUST use div/span/a — never button
- MUST use 0px border-radius — square corners only
- MUST use hard offset box-shadow (2px 2px 0px) — zero blur
- MUST map status values from actual schema
- MUST apply formatters to ALL Note/Week columns — not just one
- `customCardProps` ONLY works on `span` elements — not `div`
- Name column CANNOT be formatted in document libraries — leave as default
- No gradients, no rounded corners, no soft shadows

## EVAL

### Test Data Requirements
- **Type:** documents
- **Minimum count:** 3
- **Schema needed:** FileLeafRef, Status (Choice), Progress (Number), Deadline (DateTime), Note columns (at least 3 weeks)
- **Sample data:** Documents with varied statuses and progress values. Week columns with multiline text (header line + bullet points). Some cells empty for em dash test. Some deadlines in the past for overdue test.

### Expected Results
- **Result 1:** Column formatters persisted via `editSchemaXml` with `CustomFormatter` attribute on ALL custom columns
- **Result 2:** Status badges with correct neobrutalism color mapping (thick borders, square corners, inline edit)
- **Result 3:** Chunky progress bars with 2px borders and color coding (red <50%, yellow 50-79%, green >=80%)
- **Result 4:** Deadline column with Calendar/Warning icon, red when overdue
- **Result 5:** Note/Week columns with neobrutalism cards — 2px border, 2px offset shadow, SortUp header, Sparkle body, "Show details" popup
- **Result 6:** Empty week cells show centered em dash (—)
- **Result 7:** "Show details" popup renders full content with pre-wrap in a bordered callout
- **Result 8:** Name column uses default SharePoint rendering (cannot be formatted via API — known limitation)
- **Result 9:** Formatting persists across page refreshes and view switches — does NOT require `preview_view_changes` to render
- **UI-driven:** yes — visual confirmation required for results 2-7
