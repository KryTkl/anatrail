# Scaling AnaTrail to 200+ Sites

This describes the architecture change made to support 200+ site cards, each with 3 photos and 1 video, and exactly how to populate it at that scale.

## What changed, and why

Up to now, every site's text (in 8 languages) and every photo/video URL lived hardcoded inside `turkey-archaeology-map.html` itself. That works for 2 sites. At 200 sites × 8 languages × (name + region + period + description) + 4 media items each, the HTML file would balloon to several megabytes of inline text, become unopenable in a normal editor, and turn every small edit into a scroll-and-search exercise through one giant file.

The site is now a **small project, not a single file**:

```
your-project/
  index.html              ← the map/app itself (was turkey-archaeology-map.html)
  data/
    sites.json            ← all site content, one JSON array
  media/
    catalhoyuk/
      1.jpg  2.jpg  3.jpg  video.mp4
    aktopraklik-mound/
      1.jpg  2.jpg  3.jpg  video.mp4
    <next-slug>/
      1.jpg  2.jpg  3.jpg  video.mp4
```

The HTML file now `fetch()`es `data/sites.json` on load instead of containing the data. This is the same pattern any real content site uses — code and content are separate, so either can change without touching the other.

### Important: this now requires a web server

Browsers block `fetch()` from reading local files when a page is opened directly (`file:///...`) — a security restriction, not a bug. This means:

- ❌ Double-clicking `index.html` will show "Couldn't load site data."
- ✅ Serving it via `python3 -m http.server`, VS Code's Live Server, or hosting it (GitHub Pages, Netlify, any web host) works fine.

For local testing, run this in the project folder and open `http://localhost:8000`:

```bash
python3 -m http.server 8000
```

See `github-setup-guide.md` for publishing it properly via GitHub Pages, which serves it correctly with no extra setup.

## The `data/sites.json` schema

One JSON array, one object per site:

```json
{
  "id": 3,
  "slug": "ephesus",
  "name": "Ephesus",
  "region": { "en": "İzmir Province", "tr": "İzmir İli" },
  "era": "classical",
  "period": {
    "name": { "en": "Hellenistic to Roman Ephesus", "tr": "Helenistik'ten Roma'ya Efes" },
    "dates": { "en": "10th century BCE – 15th century CE", "tr": "MÖ 10. yüzyıl – MS 15. yüzyıl" }
  },
  "lat": 37.9395,
  "lng": 27.3417,
  "description": { "en": "...", "tr": "..." },
  "website": "https://en.wikipedia.org/wiki/Ephesus"
}
```

Field notes:

| Field | Notes |
|---|---|
| `id` | A unique number. Also drives the "SITE No. 003" catalog label — keep these sequential if you want tidy numbering, but it isn't required to function. |
| `slug` | Lowercase, hyphenated, URL-safe (e.g. `ephesus`, `aktopraklik-mound`). **This must exactly match the folder name in `media/`.** |
| `name`, `region`, `description` | Either a plain string (same text in every language) or an object keyed by language code (`en`, `tr`, `fr`, `it`, `de`, `es`, `ru`, `zh`). Any language you don't fill in falls back to English automatically — so you can add a site in English only and translate later without breaking anything. |
| `era` | One of `neolithic`, `bronze`, `classical`, `byzantine`, `ottoman` — controls the pin color and filter chip. |
| `period.name` / `period.dates` | Same string-or-object rule as `name`. Shown as two lines on the card. |
| `lat`, `lng` | Decimal degrees. Right-click a spot in Google Maps to copy them. |
| `website` | One URL, used for every language. |

You do **not** need a `media` field for normal entries — see below.

## The media convention (why you don't write 800 URLs by hand)

Every site is assumed to have exactly 3 photos and 1 video, so instead of listing URLs in the JSON, the app just looks for these four fixed files:

```
media/<slug>/1.jpg
media/<slug>/2.jpg
media/<slug>/3.jpg
media/<slug>/video.mp4
```

To add a site's media, you literally just drop four correctly-named files into a folder named after its `slug`. Nothing to type, nothing to link. If a file isn't there yet (e.g. you haven't shot the video for that site), it's silently skipped rather than showing a broken icon — so you can roll out photos and videos gradually across all 200 sites instead of blocking on having everything ready at once.

(The two example sites, Çatalhöyük and Aktopraklık Mound, use an optional `demoMedia` field pointing at placeholder images instead, purely so the map has something to show before you've built your own `media/` folder. Leave `demoMedia` out of your real entries.)

## Adding sites at scale — recommended workflow

For 2–10 sites, editing `data/sites.json` directly in a text editor is fine. For 200, hand-editing JSON is error-prone (missing commas, mismatched brackets). Two better options:

### Option A: Keep a spreadsheet, convert to JSON

Keep your source data in a spreadsheet (Google Sheets/Excel) with one row per site and columns for `id, slug, name_en, name_tr, region_en, region_tr, era, period_name_en, period_dates_en, lat, lng, description_en, website, ...`. Export it as CSV, then run this conversion script whenever you want to regenerate `sites.json`:

```python
#!/usr/bin/env python3
"""csv_to_sites.py — convert a sites spreadsheet export to data/sites.json
Usage: python3 csv_to_sites.py sites.csv > data/sites.json
Expected CSV columns (English-only version — extend per language as needed):
id, slug, name, region, era, period_name, period_dates, lat, lng, description, website
"""
import csv, json, sys

def main(path):
    sites = []
    with open(path, newline='', encoding='utf-8') as f:
        for row in csv.DictReader(f):
            sites.append({
                "id": int(row["id"]),
                "slug": row["slug"].strip(),
                "name": row["name"].strip(),
                "region": row["region"].strip(),
                "era": row["era"].strip(),
                "period": {
                    "name": row["period_name"].strip(),
                    "dates": row["period_dates"].strip()
                },
                "lat": float(row["lat"]),
                "lng": float(row["lng"]),
                "description": row["description"].strip(),
                "website": row["website"].strip()
            })
    json.dump(sites, sys.stdout, ensure_ascii=False, indent=2)

if __name__ == "__main__":
    main(sys.argv[1])
```

This keeps your actual content in a spreadsheet (easy to review, sort, hand off to someone else to fill in) and treats `sites.json` as a build output you regenerate, not something you hand-edit at scale.

### Option B: Validate before you publish

Whichever way you produce `sites.json`, always check it's valid before deploying — a single syntax error stops every site from loading, not just one:

```bash
python3 -c "import json; json.load(open('data/sites.json', encoding='utf-8')); print('valid JSON, OK to publish')"
```

## Performance at 200 sites

The map already uses marker clustering (Leaflet.markercluster), which was built for exactly this — it groups nearby pins into a single numbered badge that splits apart as you zoom in, so 200 pins in a small area like Cappadocia won't overwhelm the screen or the browser. No changes needed there as you scale up.

Photos and videos are only requested when a visitor actually opens that specific card (not upfront for all 200), and images use `loading="lazy"`, so initial page load stays fast regardless of how many sites you add.

## Translating at scale

You don't have to translate all 200 sites into all 8 languages before launching. Since every translatable field falls back to English when a language is missing, a practical rollout is:

1. Add all sites in English first.
2. Publish.
3. Translate high-priority/popular sites first, in whichever order makes sense, committing updates to `sites.json` as you go.

Nothing breaks at any point in that process — visitors just see English for anything not yet translated.
