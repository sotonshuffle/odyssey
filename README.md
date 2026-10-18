# Odyssey: walking-route comparison for Southampton

Odyssey helps Southampton walkers compare mapped routes using travel time, a
crime-risk model and their own street preferences. It shows where data is
missing so people can make an informed choice. No route is guaranteed safe.

This is a participant submission for the SotonShip Coastal City track. No
proprietary data or restricted systems were used.

**Live demo:** <https://sotonshuffle.github.io/odyssey/> (auto-published from `odyssey-app/` via the `gh-pages` branch)

## Repo map

| Path | What it is |
|---|---|
| `odyssey-app/` | The web app. Static, no backend, no keys. Run it with any static server. |
| `odyssey-city-data/` | The data pipeline: `build_odyssey_data.py` fetches every source, trains the risk model and emits `entities/*.json` |
| `GUIDE.md` | The plain-English guide: what it is, how to run it, how to publish it |
| `LICENSE` | MIT (code). Data licences are stated per source below. |

## Quickstart

```bash
cd odyssey-app
python3 -m http.server 8000      # open http://localhost:8000
```

Refresh every data source and retrain the model (about 4 minutes):

```bash
cd odyssey-city-data
python3 build_odyssey_data.py
cp entities/*.json ../odyssey-app/data/
```

Run the routing tests on the real data: `node odyssey-app/test_core.js`.

## DATA

| Source | Use | Licence |
|---|---|---|
| Police UK street-level crime API | 15,258 recorded street crimes, Feb to Aug 2026, snapped to segments | OGL v3 |
| OpenStreetMap / Overpass | 23,144 walking ways, geometry and `lit` tags | ODbL 1.0 |
| Environment Agency flood-monitoring API | live Woolston tide gauge and flood warnings, fetched live in the browser | OGL v3 |
| OSM nightlife venues | model features | ODbL 1.0 |
| Curated safe havens | 7 listed staffed venues with source notes; confirm current access and hours | this repo |

The app renders this provenance itself: the **Data sources & freshness**
panel is generated from `entities/data_manifest.json`, and each segment
popup cites Police UK with the month of the data.

## Why it matters

Night-time walking safety is a widespread concern. In the ONS Opinions and
Lifestyle Survey (February to March 2022), 82% of women in Britain said they
feel very or fairly unsafe walking alone after dark in a park or other open
space, compared with 42% of men. From Southampton City Council (2025), 70% of women reported feeling unsafe walking at night.
Odyssey exists to give people better information for these
journeys while being honest about what the data can and cannot show.

## The model

The risk model (ML) is a L2 logistic regression (13 standardised features) trained
by the pipeline on the real crime data from August 2025 to July 2026 with evaluation on August data.
AUC 0.971 on the held-out month, against 0.866 for a one-month crime baseline and
0.962 for a stronger two-month crime-and-length rule. The router uses the
model score per segment with a 12-minute detour bound. This is a ML AI model calculated
comparison, not a guaranteed estimate of an individual's safety. Full model
card, coefficients and honesty notes: `odyssey-city-data/entities/model.json`.

## Honesty by design

- <45% of mapped walking segments have no lighting data: rendered
  grey-dashed and labelled as a data gap, never given a reassuring band.
- The crime data has no time of day. Lighting and terrain preferences affect
  the route choice, but the model does not prove a route is safer at night.
