# Lot Size Lookup — Maricopa County, AZ

Type in an address, get the lot size. A static, single-file web app (no backend, no build step) that queries Maricopa County's own public parcel records live and measures the lot directly from the official parcel boundary.

**[Live demo](#)** — replace with your GitHub Pages URL once deployed (see below).

## How it works

1. You type a street address.
2. The app queries Maricopa County Assessor's public ArcGIS REST service (`MaricopaDynamicQueryService/MapServer/3`) for a parcel whose `PHYSICAL_ADDRESS` matches.
3. It requests the parcel's boundary geometry reprojected into Arizona State Plane feet (`outSR=2223`), then computes the polygon's area itself (shoelace formula) — so the lot size comes from the actual recorded boundary, not from a possibly-mislabeled attribute.
4. It also shows the Assessor's own `LAND_SIZE` attribute for reference, flagged as unverified-unit, since the county doesn't publicly document what unit that field is in.

No API key, no server, no cost. It's a single `index.html`.

## Deploying to GitHub Pages

1. Create a new GitHub repo and push these files (`index.html`, this `README.md`) to the `main` branch.
2. In the repo, go to **Settings → Pages**.
3. Under "Build and deployment," set **Source** to "Deploy from a branch," branch `main`, folder `/ (root)`.
4. Save. GitHub will give you a URL like `https://yourusername.github.io/repo-name/` within a minute or two.

That's it — no build tools, no dependencies to install.

## Known limitations (please read before relying on this)

- **Maricopa County only.** The address-matching and field names (`PHYSICAL_ADDRESS`, `LAND_SIZE`, etc.) are specific to how Maricopa County's Assessor publishes data. Every other county runs its own GIS server with its own schema — this won't work elsewhere without rebuilding the query for that county's service.
- **CORS is not guaranteed.** Maricopa County's ArcGIS service may or may not send the cross-origin headers a browser needs for a direct request. The app tries a normal request first, then automatically falls back to a JSONP request (a decades-old but still-supported ArcGIS REST feature) if the direct one is blocked. If both fail, the county server may be down or throttling automated traffic — the app will tell you and point to the county's own map viewer as a fallback.
- **Address matching is a simple text match**, not real geocoding. It looks for your street number and name inside the county's `PHYSICAL_ADDRESS` field. Apartment/unit numbers, abbreviations (e.g. "St" vs "Street"), or typos can cause a miss — if you get no match, try trimming the address down to just the number and street name.
- **Data currency.** This reflects whatever the county's live GIS service currently has on file. Very recent subdivisions, boundary adjustments, or re-plats may lag behind official recorded plats.
- **Not a survey.** The measurement is only as accurate as the county's digitized parcel boundary. For anything that needs to hold up legally (a property line dispute, a permit, a closing), get a licensed survey or confirm with the Assessor's Office directly.

## Customizing for another county

If you want to point this at a different county, you'll need to:

1. Find that county's ArcGIS REST parcel layer (usually discoverable at `<county-gis-domain>/arcgis/rest/services/`).
2. Update `SERVICE_URL` in `index.html`.
3. Update the `OUT_FIELDS` list to match that county's actual field names — they will almost certainly be different from Maricopa's.
4. Update `OUT_SR` to an appropriate local projected coordinate system in feet or meters (so the area math stays accurate); your county's GIS department or a spatial reference lookup like epsg.io can help identify the right one.

There's no nationwide standard for this data, so each county is its own small project.
