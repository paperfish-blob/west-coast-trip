# Plan: West Coast Trip — Interactive HTML Schedule Generator

## Context
This project manages a trip itinerary in Markdown and provides a Claude skill that converts it into a visual, interactive HTML schedule on demand. The HTML layout uses **days as columns** and **hour-by-hour time slots as rows** (like a planner grid).

**The system is fully flexible — the skill auto-detects however many days are in `itinerary.md` and builds the HTML accordingly. No hard-coded day count.**

---

## Project Structure

```
west-coast-trip/
├── PLAN.md                           ← this file
├── itinerary.md                      ← fill in your trip content here
├── schedule.html                     ← generated output (auto-overwritten by skill)
└── .claude/
    └── commands/
        └── generate-schedule.md     ← skill file with all conversion rules
```

---

## How to Use

1. **Edit `itinerary.md`** — add or update your days following the format below
2. **Run `/generate-schedule`** in Claude Code — Claude reads the MD and writes `schedule.html`
3. **Open `schedule.html`** in any browser to view your interactive schedule

---

## `itinerary.md` Format Rules

Each day must follow this exact structure for the skill to parse it correctly:

```markdown
## Day N — Month Day | City, State
| Time  | Activity     | Category    | Notes          |
|-------|--------------|-------------|----------------|
| 09:00 | Activity name | sightseeing | optional notes |
```

**Day heading:** `## Day N — Month Day | City, State`
- N = day number (1, 2, 3 ...)
- Date format: `May 1`, `June 3`, etc.
- Location after `|`: City, State (or Country for international)

**Table columns:**
| Column   | Required | Values / Notes |
|----------|----------|----------------|
| Time     | Yes      | HH:MM 24-hour format |
| Activity | Yes      | Short name shown on the grid |
| Category | Yes      | `sightseeing`, `food`, `travel`, `accommodation`, `other` |
| Notes    | No       | Shown on hover/click in the HTML |

---

## HTML Output Features

- **Grid layout**: Time column on the left, one column per day
- **7 days per page** with Prev/Next navigation (pages scale automatically)
- **Color coding** by category:
  - Blue — sightseeing
  - Orange — food
  - Gray — travel
  - Green — accommodation
  - Purple — other
- **Hover** over an activity to see Notes
- **Click** an activity to expand full details
- **Sticky** header row and time column while scrolling
- **Responsive** — collapses to single-day view on mobile
- **Self-contained** — pure HTML/CSS/JS, no external dependencies

---

## Skill Invocation

```
/generate-schedule
```

Claude will:
1. Read `itinerary.md`
2. Parse all `## Day N` sections
3. Write/overwrite `schedule.html`
4. Report how many days were parsed and flag any formatting issues

---

## Verification Checklist

- [ ] Open `schedule.html` in browser — grid renders correctly
- [ ] Edit a day in `itinerary.md`, re-run `/generate-schedule`, verify HTML updates
- [ ] Prev/Next navigation works across pages
- [ ] Mobile view looks correct (browser devtools responsive mode)
- [ ] Hover and click interactions work on activity cards
