# Skill: Generate Interactive HTML Schedule

> **Design system**: Before generating any HTML, apply the **UI UX Pro Max skill** (https://github.com/nextlevelbuilder/ui-ux-pro-max-skill) using the **Minimalist** style. Use its typography pairings (Inter font), spacing system, color palette, and accessibility rules (WCAG AA minimum contrast 4.5:1) to inform all visual decisions in the HTML spec below.

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
- If the only content under a day heading is an HTML comment containing the word `placeholder` (e.g. `<!-- placeholder — activities to be added -->`), mark that day as **placeholder** — it has no activities but should still appear in the grid
- Ignore all other HTML comments and lines that don't match the table format

### Activity rows
Each row has 3–4 pipe-delimited columns (ignore the header row and separator row):
| Column   | Index | Required | Notes |
|----------|-------|----------|-------|
| Time     | 0     | Yes      | Parse as HH:MM 24-hour. If unparseable, treat as `00:00` and flag it |
| Activity | 1     | Yes      | Display text for the grid cell |
| Category | 2     | Yes      | Normalise to lowercase. Map unknowns to `other` |
| Notes    | 3     | No       | Raw text, shown in tooltip/expanded view |

### Category → colour mapping

Minimalist palette — muted/desaturated tones, WCAG AA contrast on white:

| Category      | CSS colour token |
|---------------|-----------------|
| sightseeing   | `#2563eb` (blue) |
| food          | `#ea580c` (orange) |
| travel        | `#6b7280` (gray) |
| accommodation | `#16a34a` (green) |
| other         | `#9333ea` (purple) |

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
  - Card background = white with `1px solid #e5e7eb` border and `6px` border-radius; left border `2px solid` at full category colour
  - Card content: activity name (bold, small font), and time in small muted text (`#9ca3af`)
  - Hover state: subtle `box-shadow: 0 1px 4px rgba(0,0,0,0.08)` transition with `150ms ease`
  - If the Notes column is non-empty, add a `data-notes` attribute and a small ⓘ icon
- **Placeholder day columns**: if a day is marked as placeholder, render its column with:
  - Column header styled the same as normal but with `opacity: 0.5`
  - All cells in that column filled with a `#f9fafb` background and a subtle repeating diagonal stripe pattern (`repeating-linear-gradient(45deg, transparent, transparent 4px, #f3f4f6 4px, #f3f4f6 5px)`)
  - A single centred pill label "TBD" in the middle row of the column (`color: #9ca3af; font-size: 11px; font-weight: 500; letter-spacing: 0.05em; border: 1px solid #e5e7eb; border-radius: 999px; padding: 2px 8px`)
  - No hover or click behaviour on placeholder cells

### Pagination

- No pagination — render all days in a single horizontally scrollable grid
- No Prev/Next buttons; remove the page indicator
- The `<div.grid-wrap>` should have `overflow-x: auto` so the user can scroll right to see all days

### Interactivity

- **Hover** on an activity card: show a tooltip with the Notes text (if any). Use a CSS tooltip (`:hover + .tooltip`) — no JS required for this
- **Click** on an activity card: toggle an `.expanded` class that shows a detail panel below the card with:
  - An image fetched from Unsplash using the activity name and day location as the search query:
    `https://source.unsplash.com/featured/400x200?{activity}+{location}` (URL-encode both values)
  - Full Notes text below the image
  - Category badge
  - If the image fails to load (`onerror`), hide the `<img>` element gracefully — no broken icon
- Clicking elsewhere on the page closes any open expanded card

### Responsive

- Below 640px screen width: the grid scrolls horizontally — no day hiding or arrows needed
- Time column remains sticky on mobile (`position: sticky; left: 0`)

### Visual polish — Minimalist (UI UX Pro Max)

- **Font**: Load `Inter` from Google Fonts (`https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600&display=swap`); fallback `system-ui, -apple-system, sans-serif`
- **Background**: `#ffffff` (pure white)
- **Grid lines**: `1px solid #f3f4f6` (very light)
- **Header background**: `#111827` (near-black), text white
- **Time column background**: `#f9fafb`
- **Body text**: `#374151`
- **Muted/secondary text**: `#9ca3af`
- **Headings**: `letter-spacing: -0.01em`
- **Transitions**: `150ms ease` on hover states
- **Activity image** (in expanded panel): `width: 100%; height: 160px; object-fit: cover; border-radius: 6px; margin-bottom: 8px; display: block`; hidden via `display: none` on `onerror`
- **Scrollbar styling** (webkit): thin, subtle
- **Accessibility**: `role="grid"` on the table; `aria-label` on Prev/Next buttons; `focus-visible` ring (`outline: 2px solid #2563eb; outline-offset: 2px`) on all interactive elements; all text contrast ≥ 4.5:1 (WCAG AA)

---

## Output

Write the completed HTML to `index.html` in the project root, overwriting any existing file.

Then reply with a short summary:
- Number of days parsed (break down: X with activities, Y placeholder)
- Total activities
- Any warnings (missing columns, unknown categories, unparseable times) — placeholder days are **not** warnings
- Confirm `index.html` was written successfully
