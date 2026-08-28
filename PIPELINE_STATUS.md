# Pipeline Status — Hamburg Bike Safety Index (cleaned-up version)

Last updated: 2026-08-27
This file explains what's in this folder, how to run it, and how it maps to the earlier numbering
used during development.

---

## 1. Run order

Notebooks 0-8 are the original sequence. Notebooks **9 and 10 were added later** to feed two new
dashboard tabs (Home network map + Summer vs Winter). Because the dashboard (notebook 8) now also
consumes the outputs of 9 and 10, the **execution order is 0-7, then 9, then 10, then 8**:

| # | Notebook | Input | Output |
|---|---|---|---|
| 0 | `0_Hamburg_bike_stations.ipynb` | Live API | `hamburg_bike_stations.csv`, `hamburg_bike_map.html` |
| 1 | `1_Hamburg_bike_timeseries.ipynb` | Live API | `hamburg_bike_timeseries_XX_YY.csv` (batched) |
| 2 | `2_Bike_hourly_aggregation.ipynb` | Step 1 output | `hamburg_bike_counts_hourly.csv` |
| 3 | `3_Clean_5min_hourly.ipynb` | Step 2 output | `output/hamburg_bike_counts_hourly_5min_clean_v2.csv.gz` |
| 4 | `4_Time_categories_daylight.ipynb` | Step 3 output | `output/processed_hourly_bike_counts_time_categories.csv.gz`, `output/station_metadata.csv` |
| 5 | `5_Monthly_time_series_summary.ipynb` | Step 4 output | `output/monthly_day_night_summary.csv`, `output/monthly_time_period_summary.csv`, `output/monthly_temporal_ratios.csv` |
| 6 | `6_Station_Month_Day_Night.ipynb` | Step 4 output | `output/station_month_day_night.csv` |
| 7 | `7_Station_Month_Time_Period_Summary.ipynb` | Step 4 output | `output/station_month_time_period_summary.csv`, `output/station_month_temporal_ratios.csv` |
| 9 | `9_Seasonal_aggregate.ipynb` | `data/summer_*.csv`, `data/winter_*.csv` | `output/station_seasonal_summer_winter.csv`, `output/seasonal_hourly_profile.csv` |
| 10 | `10_Network_simplify.ipynb` | `data/Bike_infra_4class_4326.geojson`, `data/HH_Boundary.geojson` | `output/bike_network_simplified.geojson`, `output/bike_network_context.geojson`, `output/hamburg_boundary_simplified.geojson` |
| 8 | `8_Combined_Dashboard.ipynb` | Outputs of steps 4, 5, 6, 7, **9, 10** | `output/bike_dashboard.html` (the final interactive dashboard -- the copy at the root of this folder is the same file) |

**Why 8 keeps its number**: the dashboard has been "notebook 8" throughout the project's history
(project knowledge base, notes, cross-references). Notebooks 9 and 10 were appended rather than
renumbering everything, so the only thing to remember is the run order above.

**Setup**: install the required packages first with `pip install -r requirements.txt`.

**Path convention**: every notebook reads/writes relative paths like `output/xxx.csv`. This
package already includes an `output/` folder with the results of steps 3-7, 9 and 10 pre-computed,
so you can **re-run just the dashboard (step 8) on its own** without re-running anything before it
and without needing the large input files.

**Large inputs are not bundled** (see `.gitignore` and `data/PLACE_BIG_INPUTS_HERE.txt`). Notebooks
9 and 10 read three large files from `data/`; drop them in that folder only if you need to
regenerate their outputs. To just rebuild the dashboard, you don't need them.

**Verified reproducibility**: notebooks 3-8 were previously re-executed end-to-end in a fresh
environment with every output matching byte-for-byte. Notebooks **9 and 10, and the updated
notebook 8, were then also executed end-to-end in a fresh environment** (fresh `pip install -r
requirements.txt`), producing the committed `output/` files and the 10.5 MB `bike_dashboard.html`.
**Notebooks 0-2 were not executed by the assistant** -- they call Hamburg's live sensor API, so
running them again fetches a different point in time. If you only need steps 3-10, the `output/`
folder already has everything needed.

---

## 2. Mapping to the old numbering (for tracing back through project history)

| New # | Old filename | Notes |
|---|---|---|
| 0 | `0_hamburg_bike_stations_Colab.ipynb` | Filename shortened, content unchanged |
| 1 | `1_Hamburg_bike_timeseries_3.ipynb` | Filename shortened, content unchanged |
| 2 | `2_Bike_hourly_aggregation.ipynb` | Content unchanged |
| 3 | `5_Clean_5min_hourly.ipynb` | Content unchanged, renumbered only, translated to English |
| 4 | `6_Time_categories_daylight.ipynb` | Content unchanged, renumbered only, translated to English |
| 5 | `7_Monthly_time_series_summary.ipynb` | Content unchanged, renumbered only, translated to English |
| 6 | `9_Station_Month_Day_Night.ipynb` | Content unchanged, renumbered only, translated to English |
| 7 | `11_Station_Month_Time_Period_Summary.ipynb` | Content unchanged, renumbered only, translated to English; internal cross-reference updated |
| 8 | `12_Combined_Dashboard.ipynb` | Renumbered + translated; **now extended with a Home tab and a Summer vs Winter tab** |
| 9 | (new) | Added 2026-08: seasonal summer/winter aggregation |
| 10 | (new) | Added 2026-08: bike-network simplification for the Home map |

---

## 3. Files retired and NOT included in this cleanup

| File | Why it was retired |
|---|---|
| `3_Split_categories.ipynb` (old) | Earlier version. Used unfiltered daily+5-min mixed data, applied a `tz_convert` timezone conversion (conflicts with the later "do not convert" decision), and used a peak-hour definition of `[6,7,8,16,17]` -- none of this matches the current pipeline's assumptions |
| `4_timeseries_check.ipynb` (old) | Earlier version. Determined peak hours directly from `hour_utc.hour` without a timezone conversion (effectively judging local peak hours using UTC -- a bug); its holiday list only went up to 2026-01-01, incomplete |
| `8_Time_series_visualizations.ipynb` (old) | The 20 static PNGs it produced have been superseded by the interactive dashboard in `8_Combined_Dashboard.ipynb` |
| `10_Interactive_Dashboard.ipynb` (old) | Covered only the Day/Night topic; merged into `8_Combined_Dashboard.ipynb` (which has all tabs) |
| `hamburg_bike_counts_hourly_5min_clean.csv.gz` (v1, unversioned) | Used the Tukey IQR method for extreme-value detection, which fails on 21 low-traffic stations; superseded by the `_v2` file (switched to a percentile-based method) |
| `station_level_metrics.csv` | An intermediate product of the old notebook 8 (Panel E maps); not used by the final dashboard |
| `charts_all_20.zip`, `charts/`, `charts_v2/` (from the old notebooks 8/10) | Static PNGs superseded by the interactive dashboard |
| `summary_interpretation.md` | Companion notes for the old 20-static-chart set; content is retired along with those charts |
| `bike_day_night_dashboard.html` | An earlier Day/Night-only dashboard, superseded by `bike_dashboard.html` |
| `README_hamburg_bike_clean_v2.txt` | Its content is already fully covered in `3_Clean_5min_hourly.ipynb`'s markdown notes; not duplicated here |

---

## 4. Known limitations / things not yet done (listed honestly, so this isn't mistaken for a finished product)

1. **Step 2's aggregation logic itself has a known gap**: found very early. The notebook only
   completes "load + dedupe"; the resampling-to-hourly and coverage_pct calculation that the
   markdown promises has no corresponding code, yet the output file already has those columns.
   Pre-existing, independent of this cleanup.
2. **Timezone**: the whole pipeline treats the numbers in `hour_utc` as Hamburg local time
   directly, without a `tz_convert('Europe/Berlin')`. Empirical evidence suggests this column is
   very likely true UTC. This was an explicit "do not convert" decision by the data owner --
   details in the markdown at the top of `3_Clean_5min_hourly.ipynb`. Notebooks 9 and 10 follow
   the same convention.
3. **`time_period` stays as three categories** (`weekday_peak` / `weekday_offpeak` /
   `weekend_public_holiday`); public holidays are merged with regular weekends.
4. **Bike lane network layer: DONE (2026-08).** The lab-classified network
   (`Bike_infra_4class_4326.geojson`, 94k segments, 4 safety classes) is simplified by notebook 10
   and shown on the Home tab, coloured by safety rank (Class I safest > IV > II > III least safe --
   deliberately different from the numeric order). A coarse grey version of the same network plus
   the Hamburg city boundary (`HH_Boundary.geojson`) are drawn as faint context underlays on the
   seasonal maps (Day/Night, Peak, Summer vs Winter). The grey underlay is injected once from a
   single shared copy (`bike_network_context.geojson`) via a small script, so the file stays small
   instead of embedding the network into every map. Still open: the network is a visual overlay
   only; it is not yet joined to the counting-station metrics (e.g. traffic-per-class summaries).
5. **Seasonal windows are a fixed 6-week sample**: the Summer vs Winter tab compares a summer
   window (2025-07-01..08-15) against a winter window (2025-01-15..02-28), not full seasons. The
   two windows are matched in length (~45-46 days). ~96% of reliable stations are busier in summer
   (median ~1.7x), so the ratio map is read by *degree*, not just red/blue.
6. **The Bike Safety Index's other N1/N2/N4/N5/N6 indicators, and H3 grid interpolation**: not
   started. This pipeline still covers only the N3 (bike traffic) analysis, plus the new static
   network overlay.
7. **Dashboard "numeric marker" issue**: a defensive fix (`showlegend=False` on the seasonal maps,
   including the new Summer vs Winter map) is applied; root cause never 100% confirmed -- verify in
   actual use.
8. **A filterable/sortable traffic-hotspot table**: discussed but not implemented.
9. **Mobile layout**: confirmed not to break at common desktop widths (1280-1920px); the tab bar
   wraps and cards reflow, but no dedicated phone-screen layout work has been done.
10. **Home City-total panel (2026-08)**: the Home tab now shows a city-wide hourly total for 2025
    (all stations summed per hour) with 7-day (168-hour) rolling average and median. Only 2025 is
    shown; the 2026-01/02 tail is intentionally excluded.
11. **Home dot-size toggle (2026-08)**: the hero map can size station dots by total volume (default)
    or by average bikes-per-hour intensity, via buttons on the map. The hover shows both numbers.
12. **Display names (2026-08)**: the legacy "(veraltet)" suffix that appears in every raw station
    name is stripped for display only; the underlying data and station_ids are unchanged.
13. **Class colours (2026-08)**: safety-class colours are Class I `#1a9850`, IV `#a6d96a`,
    II `#f46d43`, III `#a50026` (chosen so orange II and red III separate clearly on the map).

---

## 5. Folder structure

```
.
├── README.md                     <- start here: overview, how to run, how to publish, how to push to git
├── PIPELINE_STATUS.md            <- this file
├── requirements.txt              <- Python packages needed for steps 3-10
├── .gitignore                    <- keeps the large input/intermediate files out of git
├── 0_Hamburg_bike_stations.ipynb
├── 1_Hamburg_bike_timeseries.ipynb
├── 2_Bike_hourly_aggregation.ipynb
├── 3_Clean_5min_hourly.ipynb
├── 4_Time_categories_daylight.ipynb
├── 5_Monthly_time_series_summary.ipynb
├── 6_Station_Month_Day_Night.ipynb
├── 7_Station_Month_Time_Period_Summary.ipynb
├── 9_Seasonal_aggregate.ipynb     <- NEW (run before 8)
├── 10_Network_simplify.ipynb      <- NEW (run before 8)
├── 8_Combined_Dashboard.ipynb     <- run LAST; builds the dashboard
├── bike_dashboard.html            <- the final interactive dashboard, double-click to open
├── data/
│   └── PLACE_BIG_INPUTS_HERE.txt  <- where to drop the 4 large input files (not committed)
└── output/                        <- CSV/GeoJSON outputs (input for step 8)
    ├── hamburg_bike_counts_hourly_5min_clean_v2.csv.gz   (git-ignored; step-3 intermediate, not used by dashboard)
    ├── processed_hourly_bike_counts_time_categories.csv.gz (committed; read by the dashboard's Daily Rhythm tab)
    ├── station_metadata.csv
    ├── monthly_day_night_summary.csv
    ├── monthly_time_period_summary.csv
    ├── monthly_temporal_ratios.csv
    ├── station_month_day_night.csv
    ├── station_month_time_period_summary.csv
    ├── station_month_temporal_ratios.csv
    ├── station_seasonal_summer_winter.csv   <- NEW (from notebook 9)
    ├── seasonal_hourly_profile.csv          <- NEW (from notebook 9)
    ├── bike_network_simplified.geojson      <- from notebook 10 (Home colour map)
    ├── bike_network_context.geojson         <- NEW (from notebook 10; grey underlay)
    ├── hamburg_boundary_simplified.geojson  <- NEW (from notebook 10; city outline)
    └── charts_v2/                 <- 4 static PNGs from step 6 (superseded by the dashboard)
```

Note: everything under `data/` and the step-3 intermediate `hamburg_bike_counts_hourly_5min_clean_v2.csv.gz`
are git-ignored. The `processed_hourly_bike_counts_time_categories.csv.gz` file is kept (the dashboard
reads it), so the committed `output/` folder is enough to re-run the dashboard (step 8) on its own.
