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

### Add route geometry points from GTFS

1. Unzip the GTFS file.
1. Click Data Source Manager > Delimited Text.
1. Set file name to `shapes.txt`.
1. Set X field to "stop_lon".
1. Set Y field to "stop_lat".
1. Click Add to create layer shapes.

@jarkkoka realized that algorithm does not work with spherical coordinates so we will reproject shapes into the same projection as Digiroad.

1. In main window, click from the menu Processing > Toolbox > Vector general > Reproject layer.
1. Set Input layer to "shapes".
1. Set Target CRS to "EPSG:3067".
1. Click Run to create layer Reprojected.

### Create route geometry linestrings (optional)

It might help to see the route geometries as connected linestrings instead of just individual points.
This is not necessary, though.

1. In main window, click from the menu Processing Toolbox > Vector creation > Points to path.
1. Set Input layer to "Reprojected".
1. Set Order expression to "shape_pt_sequence".
1. Set Path group expression to "shape_id".
1. Click Run to create layer Paths.

### Order and style layers (optional)

1. In the Layers panel of main window, drag Digitransit raster to bottom.
1. Set the "DR_LINKKI_K" as the second-lowest layer.
1. Toggle shapes off.
1. Toggle Reprojected off.
1. Toggle Paths on.
1. With focus on "DR_LINKKI_K", press F7 to open Layer Styling panel.
1. Set Width to 2 mm and Opacity to 25 %.
1. Select Paths layer.
1. Set Width to 1 mm.

### Select one route as the trajectory

1. Click Processing Toolbox > Vector Selection > Extract by attribute.
1. Extract from "Reprojected" with `shape_id = 1088B_20210522_2`.

### Crop and filter the street network

To speed up the algorithm, only include an area around the selected route.
Also remove ways for pedestrians and cyclists.

1. Select layer "DR_LINKKI\K".
1. Choose tool Select Features by Area or Single Click from the toolbar.
1. Drag a rectangle around the chosen route, leaving maybe 100 meters of buffer all around the route.
1. Click menu Edit > Copy Features.
1. Press `Ctrl+Alt+V` to Paste Features As Temporary Scratch Layer...
1. Give a name like "Around route".

Another way is to create a buffer automatically and clip Digiroad within that buffer.
I did not try that.

1. Click Processing Toolbox > Vector geometry > Buffer.
1. Create a buffer layer around the route.
1. Click Processing Toolbox > Vector overlay > Extract/clip by extent.

### Run map matching

1. In main window, click Plugins > Manage and Install Plugins...
1. Install Offline-MapMatching.
1. Click the button Match Trajectory on the toolbar OfflineMapMatching.
1. Set Network layer to "Around route".
1. Set Trajectory layer to the layer with just one route.
1. Set Trajectory ID to shape_pt_sequence.
1. Set Matching type to Based on network routing (slow).
1. Click Run.
