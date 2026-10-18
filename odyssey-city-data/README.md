# Odyssey: live data for SotonShip

**One sentence:** *Odyssey helps Southampton walkers compare mapped routes
using travel time, a historical-crime model and their own street preferences,
while showing where data is missing. No route is guaranteed safe.*

Odyssey is a walking-route comparison prototype for Southampton. Its route
model scores mapped segments with a trained model, offers
Shortest / Lower risk / Balanced routes with plain-English explanations,
hands off to Google Maps, and displays live flood information as context.
It is a static
web app: no backend, no accounts, no API keys; every data source is public
and credited.

## Repository layout

| Folder | What |
|---|---|
| `odyssey-app/` | The web app (Leaflet vendored, runs from any folder) |
| `odyssey-city-data/` | `build_odyssey_data.py`: fetches all live data, trains the risk model, emits everything the app loads |
| `odyssey-city-data/entities/` | The generated data files (also copied into `odyssey-app/data/`) |

Run the app:
```
cd odyssey-app
python3 -m http.server 8000     # then open http://localhost:8000
```

Refresh all data + retrain (about 4 minutes, internet needed):
```
cd odyssey-city-data
python3 build_odyssey_data.py
cp entities/*.json ../odyssey-app/data/
```

## Data sources: every number's provenance

| Source | What we use | Licence | Checked |
|---|---|---|---|
| OpenStreetMap via Overpass API | 23,144 walking ways (geometry, names, `lit` tags, types) | ODbL 1.0, © OSM contributors | build date (see `data_manifest.json`) |
| Police UK street-level crime API | 15,258 real recorded street crimes, Feb to Aug 2026, snapped to segments | OGL v3 (data.police.uk) | build date |
| Environment Agency flood-monitoring API | **Live** tide gauge (Woolston E12660) + active flood warnings, fetched live in the browser during use | OGL v3 | every app load |
| OSM nightlife venues | pubs/bars/nightclubs as model features | ODbL | build date |
| Curated safe havens | 7 listed staffed venues with source notes; confirm current access and hours | our curation | Oct 2026 |
| GeoJSON basemap (water/green/rail) | map background, self-contained | © OSM contributors (ODbL) | built by pipeline (step 2/8) |
| (optional) UKHO Admiralty Tidal API | tide predictions; set `TIDAL_API_KEY` to enable | UKHO free tier | build date |

The app itself shows this table live: **Data sources & freshness** panel,
generated from `data_manifest.json`.

## The risk model

The trained model produces the street score behind the lower-risk route.
The app still has a distance-focused route without it. The model is
trained by the pipeline on historical records:

- **Type:** logistic regression (L2-regularised, 13 standardised features)
- **Labels:** real Police UK street crimes snapped to walking segments
- **Features:** segment length, highway type, nightlife density, distance to
  the city centre and lagged crime counts. Lighting, steepness, shelter and
  terrain are separate route preferences, not features in this model.
- **Honest split:** trained on months up to 2026-07, evaluated on the
  held-out month **2026-08**: **AUC 0.971** vs **0.866** for a one-month
  crime baseline and **0.962** for a stronger two-month crime-and-length rule
- **Output:** a score for a reported street crime in the next month of the
  evaluation data, not personal safety odds; bands are documented quantiles
  (top 10% higher, 75th to 90th percentile medium)
- **In the router:** the Dijkstra search consumes the continuous probability
  (not just the band) as extra cost per metre, with a 12-minute detour bound

Model card with coefficients, train/test months and honesty notes:
`entities/model.json` (also rendered in the app's coverage panel).

## What's honestly real, and the failure cases we show

- **About 45% of mapped walking segments have no lighting data.** We render
  those grey-dashed and label the model band unknown rather than guessing.
- **Police UK blurs crime locations** to a nearby street point. We snap each
  crime to the nearest walking segment (≤120 m, snap distance recorded) and
  say so in the popup.
- **No time-of-day in Police UK data.** The model predicts reported street
  crime generally. Route preferences can favour mapped lighting but cannot
  turn historical data into evidence about safety at night.
- **When the lower-risk route has no lower nearby report count**, the
  app says so; the model may have considered other signals.
- **Google Maps handoff:** the start, destination and up to eight intermediate
  points are passed for non-shortest routes. Google may still redraw the walk.
- **Out of coverage / offline:** outside the Southampton bbox the router says
  so; offline the live EA panel falls back to the dated build snapshot,
  labelled "offline, not live".

## Business case

- **User:** people walking home late at night. Southampton has ~40,000
  students and a night-time economy along Bevois Valley and the waterfront.
- **Potential funders:** a university, accommodation provider, council
  community-safety team or Business Improvement District. A paid pilot needs
  user validation and evidence of value before a licence claim is credible.
- **Running cost:** static hosting, data updates, retraining and support;
  measure these before quoting a price.
- **Second city:** the pipeline is already city-parameterised. Swap the OSM
  area id, the bbox, and the Police UK sample points are starting changes.
  Every new city also needs data licensing checks, calibration and testing.

## Disclosure (track rules)

Built as a participant submission for the SotonShip Coastal City track.
Everything above was generated by the scripts in this repo; no proprietary
data or restricted systems were used.

## Demo script (3 minutes)

1. **Open on the honesty stat.** Grey dashed segments everywhere:
   "45% of Southampton's paths have no lighting data. We show what we don't
   know: every grey line is unscorable, by design."
2. **Shirley → city centre.** Compare the three route profiles using their
   actual displayed times, bands and missing-data counts. The model card
   reports AUC 0.971 versus a 0.866 one-month baseline and a 0.962
   two-month crime-and-length rule on held-out data.
3. **Click a red segment** near the centre: real crime categories with the
   Police UK credit and month. Click a grey one: lighting data is missing.
4. **Live data moment:** the panel shows the real-time Woolston tide gauge
   reading fetched from the EA seconds ago, and active flood warnings.
5. **St Denys honesty case:** compare the nearby report counts and show that
   model preference does not always mean fewer reports.
6. Hand off to Google Maps, noting it may redraw the path; show SOS
   (999/101 + official guidance links).

## Licences & attribution

OpenStreetMap data © OSM contributors, ODbL 1.0. Police UK and Environment
Agency data under OGL v3. Basemap GeoJSON © OSM contributors (ODbL). Leaflet BSD-2-Clause. App code:
MIT. Crime data contains public-sector information licensed under OGL v3.
