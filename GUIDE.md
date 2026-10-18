# Odyssey: a plain-English guide

Odyssey is a prototype for comparing walking routes in Southampton. It shows
Shortest, Lower risk and Balanced options, explains why they differ and
marks gaps in the street data. Athena's model scores mapped street segments
using historical reported crime. **A lower score does not prove a route is safe
for a particular person or journey.**

## Open the app

From the `odyssey-app` folder, run a local static server:

```text
python -m http.server 8000
```

Then open `http://localhost:8000`. On Mac or Linux, use `python3` if needed.
Opening `index.html` directly from the file system may prevent its JSON data
from loading. A fork can also serve the app through GitHub Pages if Pages is
enabled, at `https://YOURNAME.github.io/odyssey/odyssey-app/`.

## Plan a journey

Enter a destination and allow GPS to use your current location. You can also
click a start point and then a destination on the map. The route cards mean:

- **Shortest:** prioritises distance on mapped paths. Service roads and any
  saved route preferences can affect the exact path.
- **Lower risk:** gives Athena's street scores more weight, with an
  estimated detour limit of 12 minutes over Shortest.
- **Balanced:** gives distance and the scores an intermediate weight.

When two profiles produce the same mapped path, both cards say so. Selecting
**View on map** highlights a route inside Odyssey. **Google Maps** receives
the start, destination and up to eight intermediate points for non-shortest
routes. Google may still calculate a different path.

The route preferences for mapped lighting, stairs, steep gradients, shelter
and terrain change route selection. They do not change the historical crime
model. A missing street attribute is not treated as a known good attribute.

## Read the map and its limits

Green, amber and red show Athena's lower, medium and higher **model bands**.
These are relative score bands, not personal safety ratings. Grey dashed
segments have no mapped lighting data and are labelled as a data gap. A route
card shows the highest known band along that route and how many segments
lack lighting data. If every segment lacks the data, the card says the band
is unknown.

Police UK publishes approximate street-level crime locations. Odyssey maps
them to nearby walking segments, so a count is not an exact count for a
particular pavement. The records have no time of day and the current bundled
data ends in August 2026; they cannot establish whether a route is safe now
or at night. The app shows the source and date of the data. Some lighting
values are documented road-type assumptions, while many paths have no value.

The Environment Agency tide reading and flood notices are live when the
source is available, but they are context only: the live reading does not
currently change the walking route. Venue opening hours should be checked
before relying on them. Odyssey has no server for submitting route reports.

## How Athena works

`odyssey-city-data/build_odyssey_data.py` builds a walking network from
OpenStreetMap and trains a logistic-regression model on historical Police UK
records. The current model card in `odyssey-app/data/model.json` lists 13
features, including road type, nightlife density, distance to the centre and
prior crime counts. Lighting and terrain preferences are applied separately
when the router compares paths. The reported held-out August 2026 AUC is
0.971 versus 0.866 for a one-month crime baseline and 0.962 for a
stronger two-month crime-and-length rule. This evaluates prediction of
reported crime in the dataset; it does not validate personal safety.

The browser runs Dijkstra's route search over the mapped network. It compares
walking distance with the model score and the user's preferences. Walking
times assume 80 metres per minute. Out-of-coverage points and unavailable
data need an explicit warning rather than an invented result.

## Refresh data and check the build

The app loads the JSON files in `odyssey-app/data/`. To rebuild them, run the
pipeline from `odyssey-city-data/` and copy the generated `entities/*.json`
files into `odyssey-app/data/`. This needs network access and may take several
minutes. Keep the source dates and licences in `data_manifest.json` visible.

From the repository root, run `node odyssey-app/test_core.js` to check route
sanity, preference handling and the Google Maps URL. Review the actual
journeys in the browser as well; a passing test does not prove a route safe.

## Origin of this repository

The original repository states that it was built as a SotonShip organiser's
exemplar before the event. A GitHub fork retains that code and history. Check
the event rules and obtain an organiser ruling before submitting a fork as
an event-built entry. Keep the original disclosure in the README.
