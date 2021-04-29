# QGIS approach

QGIS enables easy visualization during exploration.
The plugins might offer enough functionality for this proof of concept.

## Requirements

- Install
  - QGIS
  - QGIS plugin Offline-MapMatching (QGIS > Plugins > Manage and Install Plugins)
- Download
  - The latest GTFS dump: <https://infopalvelut.storage.hsldev.com//gtfs/hsl.zip>
  - The latest Digiroad Uusimaa K: <https://aineistot.vayla.fi/digiroad/latest/Maakuntajako_DIGIROAD_K_EUREF-FIN/UUSIMAA.zip>

## Workflow

### Use Digitransit raster maps as background

For visual ease, open Digitransit raster maps as background.

1. Start QGIS.
1. Click from the menu Layer > Data Source Manager > Click XYZ > New.
1. Set Name to "Digitransit raster" or your own preference.
1. Set URL to <https://cdn.digitransit.fi/map/v1/hsl-map/{z}/{x}/{y}.png>
1. Set Tile Resolution to "High (512x512 / 192 DPI)".
1. Click OK.
1. Select "Digitransit raster".
1. Click Add to create layer Digitransit raster.

### Add Digiroad

1. Unzip the Digiroad file.
1. In Data Source Manager click Vector.
1. Set Vector Dataset(s) to `UUSIMAA_2/DR_LINKKI_K.shp`.
1. Click Add to create layer DR_LINKKI_K.
1. In main window, click from the menu Processing > Toolbox > Vector general > Reproject layer.
1. Set Input layer to "DR_LINKKI_K".
1. Set Target CRS to "EPSG:4326 - WGS 84".
1. Click Run to create layer Reprojected.

### Add route geometry points from GTFS

1. Unzip the GTFS file.
1. Click Data Source Manager > Delimited Text.
1. Set file name to `shapes.txt`.
1. Set X field to "stop_lon".
1. Set Y field to "stop_lat".
1. Click Add to create layer shapes.

### Create route geometry linestrings (optional)

It might help to see the route geometries as connected linestrings instead of just individual points.
This is not necessary, though.

1. In main window, click from the menu Processing Toolbox > Vector creation > Points to path.
1. Set Input layer to "shapes".
1. Set Order expression to "shape_pt_sequence".
1. Set Path group expression to "shape_id".
1. Click Run to create layer Paths.

### Order and style layers (optional)

1. In the Layers panel of main window, drag Digitransit raster to bottom.
1. Set the reprojected "DR_LINKKI_K" as the second-lowest layer.
1. Toggle "DR_LINKKI_K" off.
1. Toggle Reprojected on.
1. Toggle shapes off.
1. Toggle Paths on.
1. With focus on Reprojected, press F7 to open Layer Styling panel.
1. Set Width to 2 mm and Opacity to 25 %.
1. Select Paths layer.
1. Set Width to 1 mm.

### Select one route as the trajectory

1. Click Processing Toolbox > Vector Selection > Extract by attribute.
1. Extract from "shapes" with `shape_id = 1088B_20210522_2`.

### Run map matching naively

1. In main window, click Plugins > Manage and Install Plugins...
1. Install Offline-MapMatching.
1. Click the button Match Trajectory on the toolbar OfflineMapMatching.
1. Set Network layer to Reprojected.
1. Set Trajectory layer to the layer with just one route.
1. Set Trajectory ID to shape_pt_sequence.
1. Set Matching type to Based on euklidean distances (fast).
1. Click Run.
1. Wait for `initialise data structur...` to finish until the Sun burns out.

### Map matching debugging

The problem is that the plugin tries to initialize some data structure using the whole Digiroad network layer regardless of the bounds of the trajectory to be matched.

When a reasonable surrounding rectangle around one route is extracted and used as the Network layer in map matching, the initialization finishes quickly.

But backtracking gets stuck at "1 %" for hours.

Here's the log:

```
QGIS version: 3.18.2-Zürich
Qt version: 5.15.2
GDAL version: 3.2.2
GEOS version: 3.9.1-CAPI-1.14.2
PROJ version: Rel. 8.0.0, March 1st, 2021
PDAL version: 2.2.0 (git-version: Release)
Processing algorithm…
Algorithm 'Match Trajectory' starting…
Input parameters:
{ 'MAX_SEARCH_DISTANCE' : 20, 'NETWORK' : 'MultiLineStringZM?crs=EPSG:4326&uid={fbe47fe3-2d2e-4d91-adb5-2f9dd3ad6dcb}', 'OUTPUT' : 'TEMPORARY_OUTPUT', 'TRAJECTORY' : 'Point?crs=EPSG:4326&field=shape_id:string(0,0)&field=shape_pt_lat:double(0,0)&field=shape_pt_lon:double(0,0)&field=shape_pt_sequence:integer(0,0)&field=shape_dist_traveled:double(0,0)&uid={2860cf09-8906-4f53-990f-ad17016de133}', 'TRAJECTORY_ID' : 'shape_pt_sequence', 'TYPE' : 1 }

initialise data structur...
create candidate graph...
calculate starting probabilities...
calculate transition probabilities...
create backtracking...
```