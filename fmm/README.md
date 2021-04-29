# FMM approach

## Build Docker image

Build the Docker image from <https://github.com/HSLdevcom/fmm/tree/fix/improve-dockerization>.

In its project root, run:

```sh
docker build -f docker/Dockerfile -t cyang-kth/fmm:latest .
```

## Prepare Digiroad network

Digiroad `DR_LINKKI_K` shapefile should be transformed into the same format as [this script](https://github.com/cyang-kth/osm_mapmatching#1-download-routable-network-from-osm) produces.

This might get tricky.

More information: <https://github.com/cyang-kth/fmm/issues/138>

## Prepare GTFS routes

Perhaps use `shapes.txt` from the GTFS dump to create shapefiles like in the QGIS approach.

## Run FMM

Follow the [CLI advice](https://github.com/cyang-kth/fmm/tree/master/example/command_line_example).

Run something like:

```sh
docker run \
  --rm \
  --volume "${PWD}/input:/input:ro" \
  --volume "${PWD}/output:/output" \
  'cyang-kth/fmm:latest' \
  ubodt_gen \
    --network=/input/digiroad_2021_02_k_uusimaa_2_dr_linkki_k_wgs84.shp \
    --output=/output/ubodt.txt \
    --delta 3 \
    --use_omp \
  && fmm \
    --ubodt=/output/ubodt.txt \
    --network=/input/digiroad_2021_02_k_uusimaa_2_dr_linkki_k_wgs84.shp \
    --gps=/input/hsl_gtfs_2021-04-29_shapes.shp \
    -k 4 \
    -r 0.4 \
    -e 0.5 \
    --output=/output/mr.txt
```

Then `output/mr.txt` should hold something interesting.
