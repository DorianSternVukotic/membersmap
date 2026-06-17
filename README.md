# Liberland on the map

An interactive world map of registered **Liberlanders, e-residents and citizens** by city.

![map](https://img.shields.io/badge/data-anonymized-3fb950) ![static](https://img.shields.io/badge/hosting-static-58a6ff)

It's a single static page — `index.html` reads `data.json` and renders circle markers with
[Leaflet](https://leafletjs.com/). No backend, no build step, no database.

## What's in here

| File | What it is |
|------|------------|
| `index.html` | The whole app (map, controls, popups). |
| `data.json`  | 5,430 cities: `lat, lon, city, iso2, liberlanders, eresidents, citizens, members`. |

### Privacy
`data.json` contains **anonymized counts only** — no names, emails, or any personal data.
Every city on the map has **at least 5 Liberlanders** (k-anonymity), so no individual can be
identified from a count. Do **not** add per-person data to this repository; it is meant to be
public.

## Run locally

The page `fetch`es `data.json`, so it must be **served over HTTP** (opening `index.html` as a
`file://` will fail with a CORS error). Any static server works:

```bash
python3 -m http.server 8000
# then open http://localhost:8000
```

```bash
npx serve .        # Node alternative
```

(Leaflet and the map tiles load from a CDN, so an internet connection is required.)

## Deploy

It's pure static files — host them anywhere. Pick one:

### GitHub Pages (zero config — included)
This repo ships a workflow at `.github/workflows/pages.yml` that publishes the root on every
push to `main`.

1. Create a repo on GitHub and push (see [First push](#first-push)).
2. On GitHub: **Settings → Pages → Build and deployment → Source: GitHub Actions**.
3. Push to `main`. The site goes live at `https://<user>.github.io/<repo>/`.

### Netlify
```bash
npm i -g netlify-cli
netlify deploy --prod --dir .
```
Or drag the folder onto <https://app.netlify.com/drop>. No build command; publish directory `.`.

### Cloudflare Pages
Connect the repo in the Cloudflare dashboard with **Build command: _(none)_** and
**Build output directory: `/`**. Or one-shot from the CLI:
```bash
npm i -g wrangler
wrangler pages deploy . --project-name liberland-map
```

### Vercel
```bash
npm i -g vercel
vercel --prod
```
Framework preset: **Other**. No build command; output directory `.`.

### Any web server / S3 / nginx
Copy `index.html` and `data.json` to the document root. That's it.

## First push

```bash
git init
git add .
git commit -m "Liberland city map"
git branch -M main
git remote add origin git@github.com:<user>/liberland-map.git
git push -u origin main
```

## Updating the data

`data.json` is generated from the anonymized public export
(`out/city_numbers_public_major.csv`) in the data project. To regenerate:

```bash
python3 - <<'PY'
import csv, json
rows=[]
for r in csv.DictReader(open('city_numbers_public_major.csv')):
    if r['geocoded']!='t' or not r['latitude']: continue
    rows.append([round(float(r['latitude']),4), round(float(r['longitude']),4),
                 r['city'], r['country_iso2'],
                 int(r['liberlanders']), int(r['eresidents']),
                 int(r['citizens']), int(r['members'])])
rows.sort(key=lambda x:-x[4])
json.dump({"fields":["lat","lon","city","iso2","liberlanders","eresidents","citizens","members"],
           "cities":rows}, open('data.json','w'), separators=(',',':'))
PY
```

Only ever feed it the **anonymized** CSV — keep PII out of this repo.
