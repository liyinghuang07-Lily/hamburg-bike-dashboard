# Hamburg Bike Count Dashboard (N3 traffic analysis)

An interactive HTML dashboard built from Hamburg's public bike-counter network
(`HH_STA_HamburgerRadzaehlnetz`, 328 stations, hourly, 2025-01 to 2026-02), plus a supporting
notebook pipeline that produces it. This is the **N3 (bike traffic)** part of the wider Bike Safety
Index work; the other indicators are not part of this package yet.

> **Just want to look at it?** Open `bike_dashboard.html` in any modern browser (an internet
> connection is needed the first time, so the map tiles and the Plotly library can load).

The dashboard has five tabs:

- **Home** — the whole cycling network coloured by safety class, with all 328 counting stations on
  top (dot size = total traffic, or average bikes-per-hour via a toggle on the map), plus a
  city-wide hourly-traffic chart for 2025 and shortcut cards to the other tabs.
- **Day / Night** — how the day-vs-night traffic balance varies by station and month.
- **Peak / Off-peak / Weekend** — commuter peaks vs off-peak vs weekend riding.
- **Daily Rhythm (24hr)** — the 24-hour shape of a typical day.
- **Summer vs Winter** — which stations are the most seasonal (summer/winter ratio map), and how
  the shape of the day changes between the two seasons.

The seasonal maps (Day/Night, Peak, Summer vs Winter) show a faint grey bike-network and the
Hamburg city boundary underneath the station dots for spatial context.

---

## 1. Setup

```bash
pip install -r requirements.txt
```

Requires Python 3.10+ and the packages in `requirements.txt` (pandas, numpy, plotly, shapely, ijson,
astral, holidays, nbformat).

## 2. Running the notebooks

The notebooks read/write relative paths (`output/…`, `data/…`), so run them from this folder.

**Execution order is 0-7, then 9, then 10, then 8** (the dashboard, notebook 8, consumes the outputs
of 9 and 10 — see `PIPELINE_STATUS.md` for why 8 keeps its number). Notebooks 0-2 call Hamburg's
live API and are usually not re-run.

### Re-generate the dashboard only (most common)

The committed `output/` folder already contains everything the dashboard needs, so you can just run
**notebook 8** on its own — no large input files required:

```bash
jupyter nbconvert --to notebook --execute --inplace 8_Combined_Dashboard.ipynb
# -> writes output/bike_dashboard.html
```

(The root-level `bike_dashboard.html` is a copy of `output/bike_dashboard.html`.)

### Re-generate the seasonal / network inputs (notebooks 9 and 10)

Only needed if you want to rebuild those inputs from scratch. First place the three large files in
`data/` (see `data/PLACE_BIG_INPUTS_HERE.txt`):

```
data/summer_250701-0815_xy.csv
data/winter_250115-0228_xy.csv
data/Bike_infra_4class_4326.geojson
data/HH_Boundary.geojson
```

Then:

```bash
jupyter nbconvert --to notebook --execute --inplace 9_Seasonal_aggregate.ipynb
jupyter nbconvert --to notebook --execute --inplace 10_Network_simplify.ipynb
jupyter nbconvert --to notebook --execute --inplace 8_Combined_Dashboard.ipynb
```

---

## 3. Publishing the dashboard on the web

`bike_dashboard.html` is a **single self-contained file** (~10.5 MB; it pulls the Plotly library and
the map tiles from the internet at view time). It is safe to host publicly — it contains only
aggregate counts, no personal data. Three easy options:

1. **Netlify Drop** — go to <https://app.netlify.com/drop> and drag the HTML file onto the page.
   You get a public URL in seconds, no account needed to try it. Simplest for sharing a link.
2. **GitHub Pages** — commit the HTML to a repo, enable Pages (Settings → Pages), and it is served
   at `https://<user>.github.io/<repo>/bike_dashboard.html`.
   *Caveat:* on GitHub Free, Pages only works from **public** repos; on Pro, a private repo can
   serve Pages but the **published site is still public**. A private, access-controlled site needs
   GitHub Enterprise Cloud. If the repo must stay private but the dashboard can be public, host the
   HTML in a separate public repo (or use Netlify).
3. **HCU institutional hosting** — likely the best long-term home if the dashboard should live under
   an HCU/SUMO URL. Ask HCU IT for a static-file web space and upload the single HTML file.

The file is ~10.5 MB uncompressed; any of these hosts gzip it over the wire to roughly 3-4 MB.

---

## 4. Pushing this code to a private GitHub repo

`.gitignore` already excludes the large input/intermediate files, so only code + small outputs get
committed. Run these on your own machine, signed in with your own GitHub account:

```bash
# from inside this folder
git init
git add .
git commit -m "Hamburg bike dashboard: add Home network map + Summer vs Winter tabs"
```

Create an empty **private** repository on GitHub first (github.com → New repository, Private, no
README/licence), then connect and push:

```bash
git remote add origin https://github.com/<your-username>/<your-repo>.git
git branch -M main
git push -u origin main
```

If you use SSH instead of HTTPS, the remote is
`git@github.com:<your-username>/<your-repo>.git`.

### What gets committed vs ignored

- **Committed**: all notebooks, `README.md`, `PIPELINE_STATUS.md`, `requirements.txt`, `.gitignore`,
  `bike_dashboard.html`, and the small files in `output/` (including
  `bike_network_simplified.geojson`, ~13 MB, and the seasonal CSVs).
- **Ignored** (not pushed): everything in `data/`, the ~608 MB raw hourly CSV, and one large
  step-3 intermediate (`output/hamburg_bike_counts_hourly_5min_clean_v2.csv.gz`, ~30 MB) that the
  dashboard does not read. The step-4 output `output/processed_hourly_bike_counts_time_categories.csv.gz`
  (~43 MB) **is committed**, because the dashboard's Daily Rhythm tab reads it -- so whoever clones
  the repo can re-run the dashboard (notebook 8) straight away, with no large downloads.

> Note: `bike_network_simplified.geojson` (~13 MB) and `bike_dashboard.html` (~10.5 MB) are within
> GitHub's normal file-size limits, so Git LFS is not required.
