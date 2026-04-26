---
title: "Survey of India Topo Maps on OsmAnd"
date: 2026-04-26
summary: "How to get offline Survey of India 1:50,000 topographic maps working as an overlay on OsmAnd for iPhone."
icon: "🗺️"
tags: ["maps", "osmand", "india", "topo", "offline", "himachal"]
---

Survey of India (SOI) publishes detailed 1:50,000 scale topographic maps covering all of India under their Open Series Maps program. These are some of the most detailed official maps available for the Indian subcontinent — showing contours, trails, villages, rivers, and terrain features that simply don't exist in OpenStreetMap or Google Maps, especially in remote mountain areas like Himachal Pradesh.

For anyone traveling, biking, or trekking in the hills, having these maps offline on OsmAnd is genuinely useful.

## Credits

This workflow would not be possible without two open source projects:

- [**india_topo_maps**](https://github.com/ramSeraph/india_topo_maps) by [ramSeraph](https://github.com/ramSeraph) — an extraordinary effort to georeferece, process, and publicly host the entire SOI 1:50,000 Open Series Map collection. The dataset is updated weekly and freely available.
- [**mbtiles2osmand**](https://github.com/tarwirdur/mbtiles2osmand) by [tarwirdur](https://github.com/tarwirdur) — a simple but essential Python script for converting MBTiles files into the SQLitedb format that OsmAnd understands.

Both projects are open source and worth a star on GitHub if you find this guide useful.

## What You Need

- A Windows PC with QGIS installed (for GDAL tools)
- Python 3
- The [mbtiles2osmand](https://github.com/tarwirdur/mbtiles2osmand) conversion script
- OsmAnd on your phone

## Step 1: Find Your Sheets

The SOI 50k maps are available as individual georeferenced GeoTIFF files at [ramseraph.github.io/india_topo_maps](https://ramseraph.github.io/india_topo_maps/50k/osm/). Each sheet covers a small area and is named by SOI grid code (e.g. `52H_8`, `53E_10`).

To find which sheets cover your area, download the sheet index:

```powershell
Invoke-WebRequest -Uri "https://github.com/ramSeraph/india_topo_maps/releases/download/soi-ancillary/index_50k.geojson" -OutFile "index_50k.geojson" -UseBasicParsing
```

Then run this Python snippet to find sheets within your district's bounding box (adjust coordinates as needed):

```python
import json

with open('index_50k.geojson') as f:
    data = json.load(f)

# Example bbox for Kullu district
lon_min, lat_min, lon_max, lat_max = 76.9, 31.7, 77.8, 32.6

for feat in data['features']:
    props = feat['properties']
    coords = feat['geometry']['coordinates']
    flat = []
    for ring in coords:
        for c in ring:
            flat.append(c) if not isinstance(c[0], list) else flat.extend(c)
    lons = [p[0] for p in flat]
    lats = [p[1] for p in flat]
    if any(lon_min <= lo <= lon_max for lo in lons) and any(lat_min <= la <= lat_max for la in lats):
        print(props['id'])
```

## Step 2: Download the Sheets

Sheets are spread across multiple GitHub release tags (`50k-osm-georef`, `50k-osm-georef-extra1` through `extra5`). This script tries each tag and downloads whatever it finds:

```powershell
$base = "https://github.com/ramSeraph/india_topo_maps/releases/download"
$tags = @("50k-osm-georef","50k-osm-georef-extra1","50k-osm-georef-extra2","50k-osm-georef-extra3","50k-osm-georef-extra4","50k-osm-georef-extra5")
$sheets = @("52H_8","52H_10","53E_9","53E_10") # replace with your sheet list

New-Item -ItemType Directory -Force -Path "district_tifs"

foreach ($sheet in $sheets) {
    $out = "district_tifs\$sheet.tif"
    if (Test-Path $out) { Write-Host "SKIP: $sheet"; continue }
    foreach ($tag in $tags) {
        try {
            Invoke-WebRequest -Uri "$base/$tag/$sheet.tif" -OutFile $out -UseBasicParsing -ErrorAction Stop
            Write-Host "OK: $sheet from $tag"
            break
        } catch {}
    }
}
```

## Step 3: Mosaic and Convert to MBTiles

Using GDAL from your QGIS installation:

```powershell
$gdal = "C:\Program Files\QGIS 3.44.9\bin"

# Build virtual mosaic (instant)
& "$gdal\gdalbuildvrt.exe" district_mosaic.vrt district_tifs\*.tif

# Convert to MBTiles
& "$gdal\gdal_translate.exe" -co "ZLEVEL=9" -of MBTiles district_mosaic.vrt district_50k.mbtiles

# Generate lower zoom levels
& "$gdal\gdaladdo.exe" -r nearest district_50k.mbtiles
```

## Step 4: Convert to SQLitedb for OsmAnd

Install the dependency and download the conversion script:

```powershell
pip install Pillow
Invoke-WebRequest -Uri "https://raw.githubusercontent.com/tarwirdur/mbtiles2osmand/master/mbtiles2osmand.py" -OutFile "mbtiles2osmand.py" -UseBasicParsing
```

Then convert (JPEG quality 60 keeps file sizes reasonable):

```powershell
python3 mbtiles2osmand.py --jpg 60 district_50k.mbtiles district_50k.sqlitedb
```

A district like Kullu with ~25 sheets produces an MBTiles of around 1.1 GB which compresses down to roughly 150 MB as a SQLitedb — very manageable.

## Step 5: Copy to OsmAnd on iPhone

The easiest method is iTunes File Sharing:

1. Connect your iPhone to your PC and open iTunes
2. Select your device → **File Sharing** → find **OsmAnd** in the list
3. Drag your `.sqlitedb` file into the OsmAnd documents area

## Step 6: Enable as Overlay in OsmAnd

1. Open OsmAnd and go to **Configure Map**
2. Scroll to **Overlay map** or **Underlay map**
3. Select your `.sqlitedb` file from the list
4. Adjust opacity to your preference

The SOI topo layer will now appear over OsmAnd's base map, giving you contour lines, trails, and terrain detail for offline use.

## Notes

- Keep one SQLitedb per district — OsmAnd can only display one overlay at a time and large files can cause the app to lag
- The maps are updated weekly on the source website
- Not all sheets are available — some remote high-altitude areas in Lahaul-Spiti may be missing
- The tiles are in WebP format internally; the JPEG conversion step handles this during the mbtiles2osmand conversion
