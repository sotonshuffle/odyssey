# Odyssey — walking-route comparison for Southampton

> Odyssey helps Southampton walkers compare mapped routes using travel time,
> Athena's historical-crime model and their own street preferences. It shows
> missing data so people can make an informed choice; no route is guaranteed safe.

Odyssey is the exemplar project for the **SotonShip** Coastal City track: a
walking-route prototype whose navigation engine, **Athena**, scores mapped
segments and compares Shortest / Lower risk / Balanced routes. Preferences
for mapped lighting, stairs, steep gradients, shelter and terrain affect route
selection separately from the historical-crime model.

**Live demo:** <https://sotonshuffle.github.io/odyssey/> (auto-published from `odyssey-app/` via the `gh-pages` branch)

## Repo map

| Path | What it is |
|---|---|
| `odyssey-app/` | The web app. Static, no backend, no keys. Run it with any static server. Its settings include three saved colour themes. |
| `odyssey-city-data/` | The live data pipeline: `build_odyssey_data.py` fetches every source, trains the Athena model, and emits `entities/*.json` |
| `odyssey-city-data/entities/` | Generated data (real crimes, scored segments, model card, source manifest) |
| `GUIDE.md` | The complete plain-English walkthrough: what it is, how it works, how to run it, how to publish it |
| `LICENSE` | MIT (code). Data licences stated per source in the table below. |

## Quickstart

```bash
cd odyssey-app
python3 -m http.server 8000      # open http://localhost:8000
```

Refresh every data source and retrain the model (~4 minutes):

```bash
cd ../odyssey-city-data
python3 build_odyssey_data.py
cp entities/*.json ../odyssey-app/data/
```

Run the routing tests on the real data: `node odyssey-app/test_core.js`.

## What's real (everything)

| Source | Use | Licence |
|---|---|---|
| Police UK street-level crime API | 15,258 recorded street crimes, Feb–Aug 2026, snapped to segments | OGL v3 |
| OpenStreetMap / Overpass | 23,144 walking ways, geometry + `lit` tags | ODbL 1.0 |
| Environment Agency flood-monitoring API | live Woolston tide gauge + flood warnings, fetched live in the browser | OGL v3 |
| OSM nightlife venues | model features | ODbL 1.0 |
| Curated safe havens | 7 listed staffed venues with source notes; confirm current access and hours | this repo |

The app renders this provenance itself: the **Data sources & freshness**
panel is generated from `entities/data_manifest.json`, and each segment
popup cites Police UK with the month of the data.

## AI at the core

Athena's model is a logistic regression (13 standardised features)
trained by the pipeline on the real crime data, with a temporal split:
trained on months to Jul 2026, evaluated on the held-out month Aug 2026.
**AUC 0.971** on the held-out month, against **0.866** for a one-month
crime baseline and **0.962** for a stronger two-month crime-and-length rule.
The router uses the model's score per segment with a 12-minute detour bound.
This is a model comparison, not a validated estimate
of an individual's safety. Full model card, coefficients
and honesty notes: `odyssey-city-data/entities/model.json`.

## Honesty by design

- About 45% of mapped walking segments have no lighting data: rendered
  grey-dashed and labelled as a data gap, never given a reassuring band.
- Police UK blurs crime locations to a street point; snapping distances are
  recorded and the blurring is stated in the UI.
- The crime data has no time-of-day. Lighting and terrain preferences affect
  the route choice, but the model does not prove a route is safer at night.
- Google Maps receives the start, destination and up to eight intermediate
  points for non-shortest routes. Google can still recalculate the walk.
- Offline or out of coverage, the app says what is stale or unavailable
  instead of faking liveness.

## Business case (short version)

Potential pilot funders include universities, the council's community-safety
team and the city-centre BID. A pilot would need user testing, validated route
claims and measured hosting and update costs before a licence could be offered.
A second city needs local data, licence checks and fresh validation; changing
the geographic configuration alone is not enough. Full version in
`odyssey-city-data/README.md`.

## Demo script

Show a real Southampton journey, compare all three route profiles and point
out any missing lighting data. The live Environment Agency reading is context
only; it does not currently change the chosen route.

## Disclosure

Built as an organiser's exemplar ahead of the event with AI assistance
(Base44 Superagent). No proprietary data or restricted systems were used.
Entering teams should build their own during the event; this repo exists to
show the bar: real data, load-bearing AI, an honest model card, a working
product, and a live demo.
