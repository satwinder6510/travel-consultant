# Cruise Consultant - Rulebook & Design Decisions

**Last Updated:** 2026-02-16

This document tracks all UX decisions, bug fixes, and refinements made to the cruise consultant app. Review before making changes.

---

## UX Flow

### 1. Initial State
- **Clean prompt only** - no filter buttons shown initially
- Welcome message: "What kind of cruise are you looking for?"
- User types their request in natural language

### 2. After First Response
- Claude acknowledges: "I found X cruises matching your search"
- **Filter buttons appear** with message: "Click as many as apply to narrow your results"
- Buttons grouped: Type, Region, When, Duration, Budget

### 3. Filter Selection
- Multi-select: users can click multiple buttons
- Selected buttons turn blue
- Count updates live in header as filters toggle
- Green "Find Cruises" button appears when filters selected
- Filters **accumulate** across conversation (don't reset on each turn)

### 4. Results Display (after 2+ interactions)
- Cards appear showing best cruise options
- **Smart display logic:**
  - Single operator (e.g., "Royal Caribbean"): Show cheapest 5 from that operator
  - Multiple operators: Show cheapest per cruise line for variety
- Header adapts: "Best Royal Caribbean prices" vs "Best price per cruise line"

---

## Data Rules

### Date Filtering
- **Only show future cruises** (start date >= today)
- Today's date passed to Claude in prompt
- Claude instructed: "NEVER mention 2025 - it's now 2026!"

### Price Filtering
- **Exclude £0 prices** and missing prices
- Sort by price ascending for "cheapest" displays

### Region Matching
- Case-insensitive
- Uses `includes()` not exact match (handles "Caribbean & Mexico" etc.)

---

## Prompt Rules for Claude

### First Response (userTurns === 1)
- Acknowledge their search
- State the count: "I found X cruises"
- Point them to filter buttons
- **Do NOT list specific cruises yet**

### Subsequent Responses (userTurns >= 2)
- Show best options (max 5)
- Single operator: list ships/dates/prices
- Multiple operators: list cheapest per line
- End with: "Use the filter buttons below to narrow further"
- **Ask ONE refinement question** (not multiple)

### Always
- Only reference cruises from the provided data
- Never invent cruise names, ships, or prices
- Include sailing dates in format: "26 Apr 2026"

---

## Card Display

### When to Show
- After 2+ user interactions
- When `displayCruises.length > 0`

### What to Show
- Operator name (bold)
- Ship name
- Region + nights
- Sailing date with emoji: ⛵ 16 Mar 2026
- Price: £X,XXX
- Type badge: ocean/river

### Data Source
- Uses `displayCruises` (cached at same time as prompt data)
- Ensures cards match what Claude describes

---

## Filter Logic

### Accumulated Filters
- `accumulatedFilters` Set persists across conversation
- New selections ADD to accumulated, not replace
- Clicking button after Claude's question maintains context
- "Clear" button resets everything

### Filter Matching (in `filterCruises()`)
1. Start with future cruises + valid prices
2. Type: river/ocean keywords
3. Region: destination keywords map to region field
4. Duration: specific ranges (3-5, 7, 10-14 nights)
5. Budget: price ranges
6. Luxury: rating field
7. Dates: month/season keywords

---

## Known Issues Fixed

| Issue | Fix |
|-------|-----|
| Cards showing different data than Claude | Use `displayCruises` cached at prompt build time |
| Past dates (2025) suggested | Filter to future only + tell Claude current date |
| £0 prices appearing | Filter out price <= 0 |
| Mediterranean showing for Caribbean search | Case-insensitive region matching |
| 1-night cruises for "10-14 nights" | Explicit range parsing for button values |
| Button click loses context | Accumulated filters across conversation |
| Multiple questions overwhelming | Prompt says "ONE question only" |
| Question loop never showing results | After 2 turns, show results + continue refining |
| Single cruise line showing 1 result | Detect single operator, show multiple options |

---

## File Structure

```
cruise-consultant/
├── index.html      # Main app (single file)
├── worker.js       # Cloudflare Worker (API proxy)
├── wrangler.toml   # Worker config
├── RULEBOOK.md     # This file
└── *.json          # Local data files (ships, operators)
```

Data hosted on Cloudflare R2:
- `cruise-data-full.json` (101,638 cruises)

---

## Console Debugging

Debug logs added at each filter step:
- "Filtering with:" - shows search string
- "After date/price filter:" - count after base filter
- "After ocean/river filter:" - count after type
- "After region filter (X):" - count after region
- "After X nights filter:" - count after duration

Check browser console (F12) when filters seem wrong.

---

## Deployment

1. Push to GitHub: `git push`
2. GitHub Pages auto-deploys from main branch
3. Cache-busting meta tags added (no-cache)
4. Hard refresh (Ctrl+Shift+R) if changes don't appear
5. Title includes "v2" to verify deployment

---

## API

- Worker URL: `https://travel-consultant.satwinder-a59.workers.dev/`
- Model: `claude-sonnet-4-20250514`
- Max tokens: 8192
- API key stored in Cloudflare Worker env (ANTHROPIC_API_KEY)
