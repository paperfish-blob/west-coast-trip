# Skill: Generate Interactive HTML Schedule

Generate `schedule.html` from `itinerary.md`. Run this command whenever the itinerary changes.

## What to do

1. Read `itinerary.md` from the project root
2. Parse every day section using the rules below
3. Generate and **overwrite** `schedule.html` using the HTML spec below
4. Report: how many days were parsed, any days with no activities, any rows with missing required columns

---

## Parsing Rules

### Day sections
- Each day starts with a heading matching: `## Day N — <Date> | <Location>`
  - N = integer day number
  - Date = e.g. `May 1`, `June 15`
  - Location = everything after the `|` on the same line, trimmed
- Collect all consecutive table rows under that heading until the next `## Day` heading or end of file
- Ignore HTML comments (`<!-- ... -->`) and lines that don't match the table format

### Activity rows
Each row has 3–4 pipe-delimited columns (ignore the header row and separator row):
| Column   | Index | Required | Notes |
|----------|-------|----------|-------|
| Time     | 0     | Yes      | Parse as HH:MM 24-hour. If unparseable, treat as `00:00` and flag it |
| Activity | 1     | Yes      | Display text for the grid cell |
| Category | 2     | Yes      | Normalise to lowercase. Map unknowns to `other` |
| Notes    | 3     | No       | Raw text, shown in tooltip/expanded view |

### Category → colour mapping
| Category      | CSS colour token |
|---------------|-----------------|
| sightseeing   | `#3b82f6` (blue) |
| food          | `#f97316` (orange) |
| travel        | `#6b7280` (gray) |
| accommodation | `#22c55e` (green) |
| other         | `#a855f7` (purple) |

---

## HTML Generation Spec

Produce a **single self-contained HTML file** — no external CSS or JS files, no CDN links. All styles in a `<style>` block, all scripts in a `<script>` block.

### Page structure

```
<html>
  <head> ... styles ... </head>
  <body>
    <header>         — trip title + Prev/Next page controls
    <div.grid-wrap>  — scrollable area
      <table.schedule-grid>
        <thead>      — sticky row: "Time" cell + one <th> per day in page
        <tbody>      — one <tr> per hour slot 06:00–23:00
    <script>         — page navigation logic
  </body>
</html>
```

### Grid layout rules

- **Time range**: 06:00 to 23:00, one row per hour (18 rows total)
- **Time column**: first column, sticky (`position: sticky; left: 0`), width 60px, label `HH:00`
- **Day columns**: one column per day in the current page, equal width (`min-width: 140px`)
- **Header row**: sticky (`position: sticky; top: 0`), each day cell shows:
  - Line 1: `Day N` (bold)
  - Line 2: date string (e.g. `May 1`)
  - Line 3: location (smaller, muted)
- **Activity cells**: if an activity's time falls within a slot, render a coloured card inside that cell
  - Card background = category colour at 20% opacity, left border 3px solid at full category colour
  - Card content: activity name (bold, small font), and time in small muted text
  - If the Notes column is non-empty, add a `data-notes` attribute and a small ⓘ icon

### Pagination

- Show 7 days per page
- Calculate total pages: `Math.ceil(totalDays / 7)`
- Prev/Next buttons in the header; disable Prev on page 1, disable Next on last page
- Page indicator: `Week 1 of 2` (or `Days 1–7 of 12`, whichever reads better)
- Navigation is pure JS — no page reload, just hide/show columns

### Interactivity

- **Hover** on an activity card: show a tooltip with the Notes text (if any). Use a CSS tooltip (`:hover + .tooltip`) — no JS required for this
- **Click** on an activity card: toggle an `.expanded` class that shows a detail panel below the card with full Notes text and category badge
- Clicking elsewhere on the page closes any open expanded card

### Responsive

- Below 640px screen width: show only 1 day at a time, with Prev/Next day arrows replacing the page arrows
- Time column remains sticky on mobile

### Visual polish

- Background: `#f8fafc`
- Grid lines: `1px solid #e2e8f0`
- Header background: `#1e293b` (dark), text white
- Time column background: `#f1f5f9`
- Font: system-ui stack (`system-ui, -apple-system, sans-serif`)
- Scrollbar styling (webkit): thin, subtle

---

## Output

Write the completed HTML to `schedule.html` in the project root, overwriting any existing file.

Then reply with a short summary:
- Number of days parsed
- Total activities
- Any warnings (missing columns, unknown categories, unparseable times)
- Confirm `schedule.html` was written successfully
