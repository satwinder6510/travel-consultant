# Cruise Consultant

AI-powered cruise search application for river and ocean cruises.

## Tech Stack

- **Frontend**: Single-file HTML app with vanilla JavaScript
- **Styling**: Tailwind CSS (via CDN)
- **Hosting**: Cloudflare Workers + R2 storage
- **Data**: JSON files from Widgety cruise API

## File Structure

```
cruise-consultant/
├── index.html              # Main app (all HTML/CSS/JS in one file)
├── worker.js               # Cloudflare Worker (API proxy)
├── wrangler.toml           # Worker config
├── RULEBOOK.md             # UX decisions and design docs
├── CLAUDE.md               # This file
├── cruise-data-ships-all.json    # Ship data with images
├── cruise-data-operators.json    # Operator/cruise line data
```

Data on R2:
- `cruise-data-full.json` - Main cruise inventory with cabin prices

## Key Patterns

### Data Structure
Cruises have this structure:
```javascript
{
  ref: "AROSADONCLPPRE21052026",
  name: "Danube Classics",
  operator: "A-ROSA",
  ship: "A-ROSA DONNA",
  region: "Danube",
  nights: 7,
  start: "2026-05-21",
  from: "Engelhartszell",
  to: "Engelhartszell",
  rating: "Premium",
  cabins: [
    { code: "S", name: "2-bed Outside Cabin", price: "1141.0" },
    { code: "B", name: "2-bed Outside Cabin with Panorama Window", price: "1474.0" }
  ]
}
```

### Cruise Type Inference
No `type` field in data - inferred from region:
```javascript
const RIVER_REGIONS = ['danube', 'rhine', 'douro', 'rhone', 'seine', 'mekong', 'nile', ...];
function getCruiseType(cruise) {
  const region = (cruise.region || '').toLowerCase();
  return RIVER_REGIONS.some(r => region.includes(r)) ? 'river' : 'ocean';
}
```

### Itinerary Grouping
Same cruise with multiple sailing dates grouped by `operator|ship|name` key:
- `sailingDates` = count of available dates
- `allDates` = array of all departure dates
- Display shows cheapest date, hover tooltip shows all dates

### Price Handling
```javascript
function getBasePrice(cruise) {
  if (cruise.cabins && cruise.cabins.length > 0) {
    return parseFloat(cruise.cabins[0].price) || 0;
  }
  return parseFloat(cruise.price) || 0;
}
```

## UX Flow

1. **Wizard Modal** (3 steps on load):
   - Step 1: River or Ocean cruise
   - Step 2: Region/destination
   - Step 3: Number of travelers

2. **Results Display**:
   - Full cruise cards with ship images
   - Filter dropdowns: Cruise Line, Duration, Sort
   - "Ask Inder" AI search bar at bottom

3. **Card Layout** (3 columns):
   - Left: Ship image + operator badge + cruise type badge
   - Center: Title, details (ship, date, nights, port), rating
   - Right: Cabin prices + View More button

## Development

### Local Testing
```bash
python -m http.server 8080
# Open http://localhost:8080
```

### Deploy to Cloudflare
```bash
# Upload data to R2
wrangler r2 object put cruise-data/cruise-data-full.json --file cruise-data-full.json --remote

# Deploy worker
wrangler deploy
```

### Git Workflow
```bash
git add .
git commit -m "Description"
git push
```

## Important Rules

- **Only show future cruises** (start date >= today)
- **Exclude £0 prices** and missing prices
- **Don't hardcode features** - only show data we have
- **Ship images** from Widgety CDN, with gradient fallback on error
- **Cabin names** come from actual API data, not generic labels

## URLs

- Worker API: `https://travel-consultant.satwinder-a59.workers.dev/`
- R2 Data: `https://pub-484ab07ee6e74f0197c2acd5c8016f86.r2.dev/cruise-data-full.json`
- GitHub: `https://github.com/satwinder6510/travel-consultant`
