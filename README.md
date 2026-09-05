# GigaFish 🎣

A mobile-first, single-page South Florida fishing conditions predictor. It
combines weather models, marine/wave forecasts, NOAA tide predictions, and an
astronomically-computed moon/solunar calendar into a single 0–100 "Fishing
Score" for each hour of the next week — with a full breakdown, not a
black-box number.

Zero API keys, zero build step, one HTML file. Deploy the repo as-is to
Netlify (or any static host).

## Deploying

**Netlify (drag-and-drop):** zip or drag this folder onto
[app.netlify.com/drop](https://app.netlify.com/drop) — nothing to configure.

**Netlify (git-connected):** point a new Netlify site at this repo. The
included `netlify.toml` sets `publish = "."` and no build command, since
`index.html` is fully self-contained. Every push auto-deploys.

**Anywhere else:** it's one static HTML file. Any static host, or even
opening `index.html` directly from disk, works (GPS/fetch still need `https://`
or `file://` — all the APIs used here allow both).

## How it works

- **Fishing modes** (Inshore / Offshore / Freshwater) and, for the boat modes,
  a **hull type** (Flat-bottom, Modified-V, Deep-V) plus boat length — set at
  the top of the page and drive everything below.
- **Location**: GPS auto-detect on load, with quick-pick South Florida spots
  and manual lat/lon entry as a fallback. Reverse geocoding (for the label
  only) is via BigDataCloud's free client endpoint.
- **Data sources**, all free/keyless:
  - Open-Meteo Forecast API, multi-model (GFS, ECMWF, ICON, GEM, JMA) for
    wind, pressure, cloud cover, precipitation, air temp.
  - Open-Meteo Marine API for wave height/period, swell direction, sea
    surface temperature.
  - NOAA CO-OPS Tides & Currents for tide predictions and (where available)
    observed water temperature, from the nearest of several hardcoded South
    Florida stations (user-overridable via a dropdown).
  - USGS Water Services, queried dynamically by bounding box (no hardcoded
    site IDs), for Freshwater mode's nearby gauge temp/flow/level.
  - Moon phase and solunar major/minor periods: computed client-side, no API.
- **Scoring**: each hour gets a 0–100 score from a weighted blend of solunar
  timing, pressure trend, tide movement, wind, wave-vs-hull, water temp, and
  (Freshwater) rainfall/flow — tap any hour to see the full point-by-point
  breakdown. A separate "model agreement" percentage is shown alongside the
  score rather than folded into it, so low-confidence hours read as
  low-confidence, not as a falsely precise number.

## Tuning the scoring

Everything is in the `CONFIG` section at the top of the `<script>` block in
`index.html`:

- `WEIGHTS` — per-mode point allocation (each mode must sum to 100). Missing
  data for a factor is handled by redistributing its weight across whatever
  *does* have data that hour, not by zeroing the score.
- `WIND_ANCHORS`, `PRESSURE_TREND_ANCHORS` — piecewise-linear lookup tables;
  add/edit anchor points to reshape the curve.
- `HULL_BASE_CEILING_FT` — the wave height (ft) each hull type is comfortable
  up to before the app hard-caps the score and shows a "rough for your boat"
  warning. Boat length nudges this via `hullCeilingFt()`.
- `WATERTEMP_BANDS` — favorable/marginal temperature ranges per water type.
- `TIDE_STATIONS` — NOAA CO-OPS station list. IDs/coordinates are
  best-effort; if a station ever returns no data the app automatically
  falls back to the next-nearest one, and a dropdown lets the user override
  the pick manually. Worth spot-checking against
  [tidesandcurrents.noaa.gov/stations.html](https://tidesandcurrents.noaa.gov/stations.html)
  if you add new stations.

## Known approximations

- **Solunar major/minor periods** use a real but deliberately simplified
  astronomical relationship (moon age → meridian transit time), rather than
  a full lunar ephemeris — see the long comment above `computeSolunarWindows`
  in `index.html` for the exact error budget (roughly ±15–60 minutes
  depending on the factor). That's why windows are shown as ranges (±1h
  major, ±30min minor) rather than exact instants, the same way printed
  solunar tables do.
- **Onshore/offshore wind** is a coarse heuristic based on which side of the
  peninsula the coordinates fall on — informational only, not scored.
- **Freshwater water temp** persists the single latest USGS/NOAA reading
  across the whole 7-day outlook (there's no freshwater temp *forecast*
  source), which is a reasonable approximation for water temp's slow
  day-to-day drift but won't catch a sudden cold front.

## Local development

No build step — just open `index.html` in a browser, or serve the folder
with anything static (`npx serve`, `python3 -m http.server`, etc.) if you'd
rather test over `http://` than `file://`.
