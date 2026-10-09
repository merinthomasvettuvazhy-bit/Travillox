# Travillox Urban Sense (web app / PWA)

Static, installable web app. No build step, no dependencies. Uses simulated fleet data.

## Run locally
    python3 -m http.server 8080      # then open http://localhost:8080
(Service workers need http://localhost or HTTPS; opening index.html via file:// still works but won't install.)

## Deploy (any static host)
- **GitHub Pages:** push this folder to a repo > Settings > Pages > deploy from branch.
- **Netlify / Vercel / Cloudflare Pages:** drag-and-drop the folder, or connect the repo. Build command: none. Output dir: `/`.

## Install
Open the deployed HTTPS URL in Chrome/Edge/Safari > "Install app" / "Add to Home Screen". Works offline after first load.

## Files
index.html (app) · manifest.json · sw.js (offline cache) · icons/

## Next step: make it real
Replace the simulated `EV` array and the edge feed in index.html with calls to your backend
(FastAPI + PostGIS) and your YOLO inference service; POST events as {type, lat, lon, conf, bus, ts, plate?}.

## Live map & heat maps (v2)
- Key-free Esri basemaps (light/dark gray, streets) or OpenStreetMap via Leaflet, with automatic fallback; routes snap to real roads via the public OSRM server (falls back to straight waypoints offline). Needs internet for tiles/OSRM on first load.
- Heat map modes: **Congestion** (route jam profile + live pings from slow-moving buses), **Road defects & infrastructure**, **Safety incidents**. Defect/safety heat is weighted by severity x repeated sightings.
- **Export CSV** downloads `lat,lng,weight` points for the current heat map (usable in QGIS / Kepler.gl).
- Real data: set `lat`,`lng` on each event in `EV`; push bus GPS speed samples into `pings` as `[lat,lng,weight]`, weight = 1 - speed/free-flow speed.
- For production, self-host tiles/OSRM (public servers are for light demo use only).

## Troubleshooting: "Access blocked" on the map
tile.openstreetmap.org refuses requests that carry no Referer (e.g. opening index.html as file://). v3 uses CARTO tiles and sends a Referer,
but still serve the folder over http (`python3 -m http.server 8080`) or HTTPS. For production use your own tile server or a keyed provider (MapTiler, Stadia, Mapbox).

## Basemap "API key required" (v4)
CARTO tiles asked for a key, so v4 uses key-free Esri tiles by default (Basemap dropdown above the map; auto-falls back Esri gray > Esri streets > OSM).
These public tiles are for demos. For production use a keyed provider (MapTiler, Stadia, Mapbox) or self-host tiles: change the URL in `PROV` inside index.html.

## v5
Default basemap is OpenStreetMap (dark mode uses an inverted OSM style); falls back to Esri gray, then Esri streets, if OSM tiles are blocked.
OSM tiles need the app to be served over http(s) so the browser sends a Referer. Fixed a bug where the basemap code could not read the dark-mode helper.

## v6: "Access blocked" tiles
OSM returns its block notice as a normal image, so the browser cannot detect it. v6 probes each basemap with fetch() and uses the first one that answers 200 (OSM > Esri gray > Esri streets).
The service worker now caches only same-origin files (so a blocked tile can never be cached). If sw.js ever becomes empty after uploading, restore it from this zip.

## v7: MapTiler
Default basemap is your MapTiler custom map (`MT_MAP` / `MT_KEY` near the top of the script). Falls back to MapTiler Streets/Dataviz-dark, OSM, then Esri.
The key is visible in client-side code: in MapTiler Cloud > Keys, restrict "Allowed HTTP origins" to your domains (e.g. http://localhost:8080 and your deployed URL), and regenerate the key if it was shared publicly.

## v8: BMTC Bengaluru
Six real BMTC routes (335E, 333, K1, 342F, 500CF, KIAS-9) with named terminals, drawn on real roads (OSRM-snapped, so paths approximate the actual routes).
Events, delays, plates and counts are simulated. To go live, replace `ROUTES` waypoints with BMTC GTFS shapes/stops and feed bus GPS (e.g. BMTC ITMS/VTS) into `pings`.
Problem statement SIH26124 is hosted by Bharat Electronics Limited (BEL).
