# Railway Heat Exposure — Berlin (pilot for Germany)

## Repository structure

```
003_python_railway-heat-exposure-germany/
├── 00_data/
│   └── berlin-260706-free.gpkg.zip      # Geofabrik OSM extract (Berlin), incl. railways layer
├── 01_scripts/
│   └── railway_heat_exposure_berlin.ipynb   # rail network -> Landsat 30 m peak temperature
├── 02_results/
│   ├── plots/                           # maps and histogram (created by the notebook)
│   ├── rail_network.gpkg                # filtered rail+light_rail+tram lines
│   ├── rail_lst_points.gpkg              # ~1 km point samples, nearest-neighbour Landsat LST
│   ├── rail_lst_segments.gpkg            # ~1 km buffered segments, zonal Landsat LST (max/mean/count)
│   ├── summary_stats.csv                # track-km-weighted network stats (zonal estimator)
│   ├── top_corridors.csv                # corridor ranking by mean zonal max
│   └── temperature_comparison.csv       # Landsat zonal max vs. air-temperature anchor vs. rail envelope
└── README.md
```

## Project summary

**Data.** (i) Landsat 8/9 Collection 2 Level-2 surface temperature (`lwir11`, 30 m) via the Microsoft
Planetary Computer STAC API; (ii) OpenStreetMap railway network from the Geofabrik Berlin extract
(GeoPackage, `gis_osm_railways_free` layer).

**Objective.** Measure and map how hot Berlin's surface rail network got during the late-June 2026
heatwave (national record of 41.8 °C on Jun 27), using the estimator that actually characterises an
extreme: the **maximum** surface temperature observed over the heatwave window (Jun 18 – Jul 3, 2026),
read from the 30 m Landsat grid with a **buffer + zonal max/mean per ~1 km segment** — not a nearest-
neighbour point sample. All statistics are weighted by track-kilometres. Pilot for a Germany-wide
version (swap the Geofabrik URL; nothing else changes).

**Network scope.** OSM's `gis_osm_railways_free` layer covers `rail` (mainline/regional heavy rail —
also where most of the S-Bahn ends up in Geofabrik's simplified schema), `light_rail` (the rest of the
S-Bahn/VLT-type lines), `tram` (Straßenbahn), and `subway` (U-Bahn), plus a handful of
`narrow_gauge`/`miniature_railway` features. All surface temperature analysis is about exposure to the
sun, so the filter keeps `rail` + `light_rail` + `tram` (2,026 km in the Berlin extract, tunnels
dropped) and excludes `subway` outright — even the ~94 km of U-Bahn tagged as non-tunnel (elevated
viaduct sections), since "metro" is out of scope regardless of whether a given stretch is underground.

**Method.** Two representations of the same network are built from one set of ~1 km cuts: sample
points (nearest-neighbour lookup — the naive baseline) and buffered line segments (30 m-wide corridor
strips, matching the Landsat pixel footprint). Reading the 30 m grid with 1 km-spaced points would use
only 1 of every ~33 available pixels along the corridor; the buffer + zonal approach instead takes the
max/mean of every pixel touching each segment. A histogram and paired maps (point sample vs. zonal max)
make the effect of that choice visible. Track-km-weighted summary statistics and a corridor ranking are
computed on the zonal max, and a rail-steel engineering envelope (T_rail ≈ T_air + 10–20 °C for exposed
track) anchors the magnitude against DWD air temperature.

**Execution note.** The Planetary Computer is not reachable from the Cowork sandbox, so the Landsat
retrieval step must run on a local machine. Requirements: `geopandas pyogrio shapely rioxarray odc-stac
pystac-client planetary-computer rasterio matplotlib` (Python ≥ 3.10). Expected runtime: a few minutes
for the STAC queries and rasterisation over the Berlin bbox.
