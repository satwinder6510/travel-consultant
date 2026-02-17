# Cruise Consultant - Rulebook & Design Decisions

**Last Updated:** 2026-02-17

This document tracks all UX decisions, bug fixes, and refinements made to the cruise consultant app. Review before making changes.

---

## UX Flow

### 1. Wizard Modal (3 Steps)
On load, a modal guides the user through key choices:

**Step 1: Cruise Type**
- River cruise (Danube, Rhine, Douro, etc.)
- Ocean cruise (Caribbean, Mediterranean, Alaska, etc.)

**Step 2: Region/Destination**
- Options change based on Step 1
- River: Danube, Rhine, Douro, Rhone, Seine, Mekong, Nile
- Ocean: Caribbean, Mediterranean, Alaska, Northern Europe, Asia & Pacific, Australia & NZ

**Step 3: Number of Travelers**
- Solo (1) - affects cabin pricing/single supplements
- Couple (2) - standard per-person pricing
- Family/Group (3+) - larger cabins needed

### 2. Results Display
After wizard completion:
- Full cruise cards appear immediately
- Sorted by price (low to high) by default
- Shows first 20 results
- Filter bar at bottom for refinements

### 3. Filtering
- Text-based filter bar (no multi-select buttons)
- Type natural language: "under £2000", "July", "7 nights"
- Filters apply instantly (no API call)
- Active filters shown with "Clear all" option
- Filters accumulate - each search adds to previous

---

## Card Design

### Layout (matches flightsandpackages.com)
Each card has 3 columns:

**Left: Image Area**
- Gradient background (sky blue)
- Operator name in white box (top-left)
- Cruise type badge (top-right): "Cruise Only" / "River Cruise"
- Ship icon placeholder

**Center: Cruise Details**
- Title: "Region : Cruise Name"
- Details row with SVG icons:
  - Ship icon + ship name
  - Calendar icon + departure date
  - Clock icon + nights
  - Location icon + embarkation port
- Rating badge (Premium/Luxury/Ultra Luxury)

**Right: Pricing Panel**
- "From:" label
- All cabin prices shown:
  - Inside (ocean) / Outside (river) - main price
  - Balcony price (if available)
  - Suite price (if available)
- "View Itinerary" link
- Red "VIEW MORE" button

### Data Rules
- **Only show data we have** - no hardcoded "Cruise Includes" or fake features
- Cabin type label: "Inside" for ocean, "Outside" for river
- Prices formatted with £ and comma separators
- Dates formatted: "01 May 2026"

---

## Sorting Options

Available via dropdown:
- Price: Low to High (default)
- Price: High to Low
- Departure Date
- Duration (nights)

---

## Data Rules

### Date Filtering
- **Only show future cruises** (start date >= today)
- Today's date: 2026-02-17

### Price Filtering
- **Exclude £0 prices** and missing prices
- Show all available cabin types (price, balcony, suite)
- Hide cabin type if price is 0 or missing

### Region Matching
- Case-insensitive
- Uses `includes()` not exact match

---

## Filter Logic

### Wizard Base Filters
Applied after wizard completion:
1. Cruise type (river/ocean)
2. Region
3. Future dates only
4. Valid prices only

### Text Filter Additions
User can type to add filters:
- Budget: "under £2000", "budget", "luxury"
- Duration: "7 nights", "week", "10-14 nights"
- Dates: "July", "summer", "Christmas"
- Operators: "Viking", "NCL", "Royal Caribbean"

### Filter Matching (in `filterCruises()`)
1. Start with future cruises + valid prices
2. Type: river/ocean keywords
3. Region: destination keywords map to region field
4. Duration: specific ranges (3-5, 7, 10-14 nights)
5. Budget: price ranges and keywords
6. Luxury: rating field
7. Dates: month/season keywords
8. Operators: cruise line names

---

## File Structure

```
cruise-consultant/
├── index.html          # Main app (single file)
├── worker.js           # Cloudflare Worker (API proxy)
├── wrangler.toml       # Worker config
├── RULEBOOK.md         # This file
├── cruise-data-*.json  # Local data files
```

Data hosted on Cloudflare R2:
- `cruise-data-full.json` - main cruise inventory

Local data files:
- `cruise-data-ships-all.json` - ship details
- `cruise-data-operators.json` - operator details

---

## Console Debugging

Debug logs at each filter step:
- "Filtering with:" - shows search string
- "After date/price filter:" - count after base filter
- "After ocean/river filter:" - count after type
- "After region filter (X):" - count after region
- "After X nights filter:" - count after duration

Check browser console (F12) when filters seem wrong.

---

## Deployment

### GitHub
```bash
git add .
git commit -m "Description"
git push
```

### Local Testing
File:// protocol blocks fetch requests. Use local server:
```bash
python -m http.server 8080
# Then open http://localhost:8080
```

### Cloudflare
- Worker: `cruise-api` via wrangler
- Static files: R2 bucket `cruise-data`

---

## API

- Worker URL: `https://travel-consultant.satwinder-a59.workers.dev/`
- Model: `claude-sonnet-4-20250514`
- Note: Current UX uses instant local filtering, not API calls for search

---

## Design Decisions Log

| Date | Decision | Reason |
|------|----------|--------|
| 2026-02-17 | Wizard modal instead of chat-first | Cleaner UX, focused choices |
| 2026-02-17 | 3 questions: Type, Region, Travelers | Most impactful for narrowing results |
| 2026-02-17 | Full cards not compact | Match existing website style |
| 2026-02-17 | All cabin prices shown | Users want to compare options |
| 2026-02-17 | Inside/Outside cabin labels | Inside=ocean base, Outside=river base |
| 2026-02-17 | Removed hardcoded "Cruise Includes" | Don't show data we don't have |
| 2026-02-17 | Instant filtering (no API) | Faster, no waiting |
| 2026-02-17 | SVG icons not emojis | Professional look |
