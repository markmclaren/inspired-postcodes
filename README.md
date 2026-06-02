# Inspired Postcodes

An interactive web map for exploring GB postcode boundaries derived from open Land Registry and ONS data.

**Live map:** [markmclaren.github.io/inspired-postcodes](https://markmclaren.github.io/inspired-postcodes/)

---

## What is this?

This project uses INSPIRE Index Polygon data (HM Land Registry) combined with NSUL point data (ONS) to produce polygon boundaries for GB postcodes at multiple levels of aggregation. The resulting boundaries are served as [PMTiles](https://protomaps.com/docs/pmtiles) vector tile files and rendered in the browser using [MapLibre GL JS](https://maplibre.org/).

The approach was inspired by Mark Longair's blog post:
[Open Data GB Postcode Unit Boundaries](https://longair.net/blog/2021/08/23/open-data-gb-postcode-unit-boundaries/)

Mark's Voronoi-processed postcode data, derived from Mapit, is also available as PMTiles:
[github.com/markmclaren/mapit-postcode-pmtiles](https://github.com/markmclaren/mapit-postcode-pmtiles)

---

## Map Layers

| Layer | Description |
|---|---|
| **Original INSPIRE** | All INSPIRE property boundary polygons. Where a parcel spans multiple postcodes, it is split into segments using Voronoi cells derived from UPRN coordinates. |
| **Single Postcodes** | Property parcels that map cleanly to exactly one postcode. |
| **Dissolved Postcodes** | Postcode unit boundaries formed by dissolving all single-postcode parcel geometries by postcode (e.g. `AB10 1AB`). |
| **Sector Dissolved** | Postcode sector boundaries (e.g. `AB10 1`), formed by dissolving the unit boundaries. |
| **District Dissolved** | Postcode district boundaries (e.g. `AB10`), formed by dissolving the sector boundaries. |
| **Area Dissolved** | Top-level postcode area boundaries (e.g. `AB`), formed by dissolving the district boundaries. |

All layers include a `colour_index` property (0–5) for neighbour-contrast colouring.

---

## Data Sources

- **INSPIRE Index Polygons** — HM Land Registry Cadastral Parcel data for England and Wales.
  Download: [use-land-property-data.service.gov.uk/datasets/inspire](https://use-land-property-data.service.gov.uk/datasets/inspire/download)

- **National Statistics UPRN Lookup (NSUL)** — ONS dataset mapping Unique Property Reference Numbers (UPRNs) to postcodes and grid coordinates (December 2025, Epoch 123).
  Download: [geoportal.statistics.gov.uk](https://geoportal.statistics.gov.uk/datasets/4e0b4b3fbc2540caae27e7be532e61be/about)

The PMTiles files are hosted on Hugging Face:
[huggingface.co/datasets/markmclaren/inspired-postcodes](https://huggingface.co/datasets/markmclaren/inspired-postcodes)

---

## How the Data Was Produced

The pipeline was run on the [Isambard 3](https://docs.isambard.ac.uk/) HPC system. Processing steps:

1. Download INSPIRE GML zip files (one per local authority, ~318 councils).
2. Load each GML into DuckDB via GDAL/`ST_Read`.
3. Spatially join NSUL UPRN points onto polygons to assign postcodes.
4. For parcels containing multiple postcodes, split geometry using Voronoi cells seeded by UPRN coordinates.
5. Export to newline-delimited GeoJSON per council.
6. Convert each GeoJSON to MBTiles with [Tippecanoe](https://github.com/felt/tippecanoe).
7. Merge all per-council MBTiles with `tile-join` and convert to PMTiles.
8. Dissolve the single-postcode layer progressively by postcode unit, sector, district, and area to produce the four dissolved layers.

Source code and Slurm job scripts: [github.com/markmclaren/isambard-postcodes](https://github.com/markmclaren/isambard-postcodes)

---

## Licence

Contains HM Land Registry data © Crown copyright and database right 2025.
Contains National Statistics data © Crown copyright and database right 2025.
Licensed under the [Open Government Licence v3.0](https://www.nationalarchives.gov.uk/doc/open-government-licence/version/3/).

