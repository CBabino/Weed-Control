# Lot Size Lookup — Maricopa County, AZ

Type in an address, get the lot size — and the treatable acreage once you subtract the actual building footprints on the lot. A static, single-file web app (no backend, no build step) that combines two live public data sources.

**[Live demo](#)** — replace with your GitHub Pages URL once deployed (see below).

## How it works

1. You type a street address.
2. The app queries Maricopa County Assessor's public ArcGIS REST service (`MaricopaDynamicQueryService/MapServer/3`) for a parcel whose `PHYSICAL_ADDRESS` matches, requesting the boundary geometry in plain lat/lon (WGS84).
3. It computes the parcel's area itself from that boundary (a local equirectangular projection + shoelace formula — accurate to well under 0.1% at lot scale), so lot size comes from the actual recorded boundary, not a possibly-mislabeled attribute.
4. It feeds that same boundary into the [Overpass API](https://overpass-api.de) — OpenStreetMap's live query service — asking for every building footprint that falls inside the parcel, then measures those footprints the same way.
5. **Treatable acreage** = measured lot area − measured building footprint(s).
6. If OpenStreetMap has nothing mapped on that lot, it falls back to the Assessor's `LIVING_SPACE` field (interior finished square footage) as a rougher stand-in, and says so on the card.

No API key, no server, no cost. It's a single `index.html`.

### About "treatable acreage" specifically

The primary method (OpenStreetMap footprints) measures the actual roofline of whatever's built on the lot — garage, house, shed, whatever OSM has traced — so it's a real footprint measurement, not a proxy. Two caveats:

- **OSM coverage varies.** Dense Phoenix-metro subdivisions are generally well mapped, but rural parcels, brand-new construction, or outbuildings sometimes aren't traced yet. When nothing is found, the card says so explicitly and falls back to the Assessor's `LIVING_SPACE` field — which has its own known gaps (undercounts multi-story footprints, misses garages/sheds/pools) and is labeled accordingly.
- **Footprints aren't clipped to the parcel line.** A building is counted if its center point falls inside the parcel boundary, and its full footprint area is used. For a structure that straddles a property line (rare for houses, more plausible for a shared fence or a barn on a large rural lot), this can be slightly off in either direction.

Every card tells you which method actually produced its number, so you're never looking at a figure without knowing how it was derived.

## Deploying to GitHub Pages

1. Create a new GitHub repo and push these files (`index.html`, this `README.md`) to the `main` branch.
2. In the repo, go to **Settings → Pages**.
3. Under "Build and deployment," set **Source** to "Deploy from a branch," branch `main`, folder `/ (root)`.
4. Save. GitHub will give you a URL like `https://yourusername.github.io/repo-name/` within a minute or two.

That's it — no build tools, no dependencies to install.

## Known limitations (please read before relying on this)

- **Maricopa County only** for the parcel lookup. The address-matching and field names (`PHYSICAL_ADDRESS`, `LAND_SIZE`, etc.) are specific to how Maricopa County's Assessor publishes data. The OpenStreetMap building-footprint step is not county-limited and would work anywhere OSM has coverage — only the parcel-boundary step needs rebuilding for another county.
- **CORS is not guaranteed** on the Assessor's service. The app tries a normal request first, then automatically falls back to a JSONP request (a decades-old but still-supported ArcGIS REST feature) if the direct one is blocked. If both fail, the county server may be down or throttling automated traffic — the app will tell you and point to the county's own map viewer as a fallback. Overpass, by contrast, sends proper CORS headers, so it's called directly; the app also tries a second mirror endpoint if the primary is down or rate-limited.
- **Address matching is a simple text match**, not real geocoding. It looks for your street number and name inside the county's `PHYSICAL_ADDRESS` field. Apartment/unit numbers, abbreviations (e.g. "St" vs "Street"), or typos can cause a miss — if you get no match, try trimming the address down to just the number and street name.
- **Data currency.** Parcel boundaries reflect whatever the county's live GIS service currently has on file; building footprints reflect whatever OpenStreetMap contributors have traced. Both can lag real-world changes — recent subdivisions, re-plats, or new construction may not show up immediately in either source.
- **Not a survey.** Both measurements are only as accurate as the underlying digitized data. For anything that needs to hold up legally (a property line dispute, a permit, a closing) or a precise chemical application rate, get a licensed survey or verify structure footprints on the ground.
- **Overpass is a shared public resource.** It's free and requires no API key, but it's rate-limited and occasionally slow under load. For occasional personal lookups this is a non-issue; for high-volume or commercial use, consider running your own Overpass instance or a paid API.

## Customizing for another county

If you want to point the parcel-lookup half at a different county, you'll need to:

1. Find that county's ArcGIS REST parcel layer (usually discoverable at `<county-gis-domain>/arcgis/rest/services/`).
2. Update `SERVICE_URL` in `index.html`.
3. Update the `OUT_FIELDS` list to match that county's actual field names — they will almost certainly be different from Maricopa's.
4. Make sure the query still requests geometry in WGS84 (`outSR=4326`) — the Overpass building-footprint step depends on getting lat/lon coordinates back.

The Overpass building-footprint step needs no changes at all — it works off the returned parcel boundary regardless of which county it came from.

There's no nationwide standard for parcel data, so each county's lookup is its own small project, but the "measure the footprint against OpenStreetMap" half is portable everywhere.
