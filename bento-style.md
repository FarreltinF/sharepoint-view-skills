# Bento Style

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
Applies a "Bento" visual style to a SharePoint document library — modular grid layout with card-like blocks, warm peach/cream palette, clear hierarchy, soft rounded corners, and subtle visual contrast. Uses the Bento design system tokens: primary (#FAD4C0), secondary (#80A1C1), surface (#FFF5E6), success/warning/danger, and 4/8px rounded corners. Use when the user asks to "apply bento style", "style my library bento", "make it look bento", or "apply warm/peach/cream style".

## Prerequisites
- The document library must have version history enabled.
- Works best with libraries that have Status, Progress, and Assigned To columns, but adapts to whatever metadata is available.

## Style Tokens
- **Primary:** rgba(250,212,192,1) — #FAD4C0 — warm peach accent
- **Secondary:** rgba(128,161,193,1) — #80A1C1 — cool blue accent
- **Success:** rgba(22,163,74,1) — #16A34A
- **Warning:** rgba(217,119,6,1) — #D97706
- **Danger:** rgba(220,38,38,1) — #DC2626
- **Surface:** rgba(255,245,230,1) — #FFF5E6 — cream background
- **Text:** rgba(17,24,39,1) — #111827 — near-black
- **Neutral:** rgba(255,245,230,1) — #FFF5E6
- **Border radius sm:** 4px
- **Border radius md:** 8px
- **Spacing sm/md:** 4px / 8px

## Execution Steps

### Step 1-7: Same as figma-tracking-style — with the same safety rules: only create week columns if none exist, only populate data for newly created columns, and never overwrite existing data.
Follow the same data collection and week bucket computation steps as figma-tracking-style (discover schema, list documents, get version history, compute week buckets, generate summaries, create columns, populate data).

### Step 8: Create the View
Create a new view named "Bento Tracking" with fields: DocIcon, LinkFilename, Status, Progress, Deadline, Week5, Week4, Week3, Week2, Week1. Do NOT include FileLeafRef.

### Step 9: Apply Column Formatting (Bento Style)

All formatters use editSchemaXml with CustomFormatter XML attribute for persistence.

#### Status Column
Rounded pill badges (border-radius: 4px) with inline editing.
- Draft: rgba(100,116,139,1) bg, white text
- In Review: rgba(128,161,193,1) bg, white text (secondary blue)
- Revising: rgba(217,119,6,1) bg, white text (warning)
- Approved: rgba(22,163,74,1) bg, white text (success)
- Published: rgba(250,212,192,1) bg, rgba(17,24,39,1) text (primary peach)

#### Progress Column
Mini progress bar with cream track (rgba(255,245,230,1)), border-radius 4px.
- <50%: rgba(220,38,38,1) danger
- 50-79%: rgba(217,119,6,1) warning
- >=80%: rgba(22,163,74,1) success

#### Deadline Column
Calendar icon in cream circle (border-radius: 8px). Overdue: red bg + Warning icon.

#### Week Columns
Bento card blocks: cream bg rgba(255,245,230,1), peach border 1px solid rgba(250,212,192,1), border-radius 8px. Show details popup in secondary blue rgba(128,161,193,1) with cream popup bg.

## Key Design Decisions
- Warm peach primary + cream surface = Bento signature palette
- border-radius 8px on cards, 4px on pills
- Secondary blue for interactive elements
- Published status uses primary peach bg + dark text
- editSchemaXml for all persistent formatting

## EVAL

### Test Data Requirements
- **Type:** documents
- **Minimum count:** 6
- **Schema/columns needed:** Status (Choice), Progress (Number), Deadline (DateTime), Week1-5 (Note)
- **Sample data description:** 6 documents with varying statuses, progress, deadlines, and week data

### Expected Results
- **Result 1:** Bento Tracking view with correct column order
- **Result 2:** Cream Bento cards with peach border on week cells
- **Result 3:** Status pills with correct Bento colors
- **Result 4:** Progress bar with cream track
- **Result 5:** Deadline with cream circle icon container
- **Result 6:** Show details popup with cream bg and peach border
- **UI-driven:** yes
