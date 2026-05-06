# Travel Guide Generator — Claude Code Skill

A reusable methodology for building complete travel guide websites using Claude Code + Google Maps API.

## What it does

Takes a travel destination + basic trip info → outputs a fully deployable static website with:
- Destination intro with hand-drawn SVG map (Chinese-labeled)
- Flight info cards
- Accommodation listings with inline maps (auto-switches AMap/Google Maps by IP)
- Top 10 attractions (cross-referenced: blogger reviews × Google Maps ratings)
- Top 10 restaurants (with ratings, must-try dishes, price in local + CNY, dietary notes)
- Activity/experience recommendations (cooking class, Muay Thai, pottery, etc.)

## Key Techniques

### Research Pipeline
1. **User input**: screenshots from Airbnb/booking platforms, blogger recommendations (Xiaohongshu, etc.)
2. **Cross-validation**: Google Maps `search_places` + `place_details` for ratings, review counts, coordinates, photos
3. **Ranking**: `blogger_mention_frequency × google_rating` → Top N
4. **Image sourcing**: Google Maps Place Photos (most accurate) → Pexels (fallback) → always verify visually before deploying

### Map Embedding (China + Overseas)
```javascript
// IP detection → provider selection
fetch("https://ipapi.co/json/", { signal: AbortSignal.timeout(3000) })
  .then(r => r.json())
  .then(d => d.country_code === "CN" ? loadAMap() : loadGMap())
  .catch(() => loadAMap()); // ipapi blocked = likely China
```
- **AMap** for China (needs Web JS API key + securityJsCode)
- **Google Maps** for overseas (needs JS Maps key + `v=weekly` + `callback` pattern)
- External JS file, NOT inline script (React hydration destroys inline scripts)

### SVG Illustrated Map
Hand-drawn style overview map in pure SVG:
- All labels in Chinese
- Geographic features: mountains, rivers, old city walls
- Numbered accommodation markers with date labels
- Landmark indicators
- Compass + scale bar
- Vector rendering = crisp at any screen size, no external image dependency

### Tech Stack
- **Next.js** (App Router, `output: "export"` for static site)
- **Tailwind CSS v4** (`@import "tailwindcss"` + `@theme inline`)
- **Deploy**: `rsync -avz --delete out/ server:/path/`

## Known Gotchas

| Problem | Solution |
|---------|----------|
| Turbopack + CJK file paths | Use `next build` directly, avoid dev server |
| React hydration kills inline `<script>` | Use external `.js` file with `defer` |
| Google Maps `Marker` unreliable with `onload` | Use `callback` param + `v=weekly` |
| `ipapi.co` blocked in China → fallback loads Google (also blocked) | Fallback to AMap first (blocked = likely in China) |
| Airbnb pages return 403 | Use Google Maps reviews (guests often mention Airbnb location) |
| Stock photo mismatch | **Always** `Read` each downloaded image to visually verify before deploying |

## Usage

Install as a Claude Code skill:
```
~/.claude/commands/travel-guide.md
```

Then invoke with `/travel-guide` in Claude Code, providing your destination and trip details.

## License

MIT
