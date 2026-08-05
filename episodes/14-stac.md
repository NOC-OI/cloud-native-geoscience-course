---
title: Organizing Cloud Zarr Data with STAC
teaching: 30
exercises: 25
---

:::::::::::::::::::::::::::::::::::::::::: objectives

- "Explain what STAC is and why it matters for cloud‑native geospatial workflows."
- "Describe the roles of STAC Catalogs, Collections, and Items."
- "Create simple STAC objects (catalog, collection, item) using the PySTAC API."
- "Link a Zarr dataset as a STAC Item asset and understand options for JSON vs database-backed STAC (e.g. pgSTAC/PgPystac)."

::::::::::::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::::::::: questions

- "How does STAC help us organise and discover Zarr data cubes in the cloud?"
- "What is the difference between a STAC Catalog, Collection, and Item?"
- "How can we programmatically build STAC metadata for our Zarr datasets?"
- "Why might we store STAC metadata in a database instead of a set of JSON files?"

::::::::::::::::::::::::::::::::::::::::::::::::::


## What is STAC and why is it important?

**STAC (SpatioTemporal Asset Catalog)** is a family of JSON-based specifications for describing geospatial assets (data files) along with their space‑time metadata and making them easy to browse and search. See the spec overview at:

- STAC spec site: <https://stacspec.org/en/about/stac-spec/>
- GitHub repository: <https://github.com/radiantearth/stac-spec>

The core STAC objects are:

- **Catalog**: provides a simple, hierarchical structure of links that group Items (and other Catalogs/Collections) for browsing.
- **Collection**: extends Catalog with additional metadata (spatial/temporal extent, license, keywords, providers) for a coherent group of Items.
- **Item**: describes a single spatiotemporal asset (e.g. one satellite scene, one product, or one Zarr datacube at a given extent/time).

The actual data files (e.g. Zarr stores, NetCDF files, Cloud Optimized GeoTIFFs) are referenced as `assets` in the Item. Each asset has a URL (or Object Store path), media type, and optional roles (e.g. "data", "thumbnail", "metadata").

![[Source](https://sparkgeo.com/blog/introducing-stac-fastapi-indexed-low-overhead-stac-metadata-support/)](episodes/fig/stac_organisation.png){alt="STAC organisation diagram showing Catalog, Collection, Item, and Asset relationships."}

For cloud‑native workflows, STAC is important because:

- It standardises how we describe and organise datasets (including Zarr) so that tools and portals can discover them without custom code for each provider.
- It works equally well as static JSON files in object storage or as dynamic APIs backed by databases, making it flexible for different scales and infrastructures.
- Many ecosystem tools ([STAC Browser](https://github.com/radiantearth/stac-browser), [PySTAC](https://pystac.readthedocs.io/en/latest/), STAC API servers, [pgSTAC](https://github.com/stac-utils/pgstac)) already understand STAC, so using STAC makes our Zarr data "plug‑and‑play" in that ecosystem.

### STAC Catalog

A STAC Catalog is a simple JSON object that links to child Catalogs, Collections, and Items:

```json
{
  "stac_version": "1.1.0",
  "type": "Catalog",
  "id": "root-catalog",
  "title": "ERA5 Zarr Data Catalog",
  "description": "Root catalog for our cloud-native Zarr data",
  "links": [
    {
      "rel": "child",
      "href": "./era5-zarr/collection.json",
      "type": "application/json"
    },
    /* ... */
  ]
}
```

The Catalog acts as a top-level entry point for browsing and searching the dataset hierarchy. It can link to other Catalogs, Collections, or Items via `links` with `rel` values like `child`, `item`, or `collection`. More details on the [Catalog spec](https://github.com/radiantearth/stac-spec/tree/master/catalog-spec).

### STAC Collection

A STAC Collection extends Catalog with extra fields describing a coherent group of Items:

```json
{
  "stac_version": "1.1.0",
  "type": "Collection",
  "id": "era5-zarr",
  "description": "ERA5 surface Zarr datacubes",
  "license": "CC-BY-4.0",
  "extent": {
    "spatial": { "bbox": [[-180, -90, 180, 90]] },
    "temporal": { "interval": [["1950-01-01T00:00:00Z", null]] }
  },
  "keywords": ["reanalysis", "atmosphere", "zarr"],
  "links": [
    {
      "rel": "parent",
      "href": "../catalog.json",
      "type": "application/json",
      "title": "ERA5 Zarr Data Catalog"
    },
    {
      "rel": "item",
      "href": "../items/era5-zarr-2020.json",
      "type": "application/geo+json"
    }
  ]
}
```

Here, `extent` describes the spatial and temporal coverage of the collection, `license` indicates usage rights, and `keywords` help with discovery. The `links` section connects to parent Catalogs and child Items. More details and the full list of fields are available on the [Collection spec](https://github.com/radiantearth/stac-spec/tree/master/collection-spec).

### STAC Item

A STAC Item is the smallest unit in STAC, an atomic bundle of inseparable metadata and assets. It represents the data at a specific place and time. It is a GeoJSON Feature with additional STAC fields:

```json
{
  "stac_version": "1.1.0",
  "type": "Feature",
  "id": "era5-zarr-2020",
  "bbox": [
    0,
    -90,
    360,
    90
  ],
  "geometry": { "type": "Polygon", "coordinates": [/* ... */] },
  "properties": {
    "start_datetime": "2020-01-01T00:00:00Z",
    "end_datetime": "2020-12-31T23:59:59Z",
  },
  "collection": "era5-zarr",
  "links": [],
  "assets": {
    "data": {
      "href": "s3://my-bucket/path/to/data.zarr",
      "media_type": "application/vnd+zarr",
    }
  }
}
```

An Item has a `geometry` and `bbox` for spatial coverage, `properties` for temporal coverage and other metadata, and an `assets` dictionary that points to the actual data files (e.g. Zarr or Icechunk store in object storage). See the [Item spec](https://github.com/radiantearth/stac-spec/tree/master/item-spec) for more details.

## STAC and cloud-native Zarr workflows

For Zarr datasets in object storage:

- Each Zarr store is typically referenced as an **asset** in a STAC Item pointing to the store's root (e.g. `s3://bucket/path.zarr`).
- Collections can group multiple Zarr stores that share a product (e.g. "ERA5 surface daily Zarr cubes" or "CMIP6 model X Zarr outputs").
- Catalogs provide top-level organisation (e.g. per provider, per project).

Benefits for cloud-native workflows:

- STAC makes Zarr datasets "discoverable" by spatial/temporal criteria and keywords, rather than "hidden" as raw paths in buckets.
- Tools like [STAC Browser](https://github.com/radiantearth/stac-browser) can display and search your STAC catalog or API, making Zarr assets visible in a generic UI.
- STAC API servers backed by databases (e.g. pgSTAC/PgPystac) can support fast, scalable search over many Zarr datasets. More information on [STAC API](https://github.com/radiantearth/stac-api-spec) and [pgSTAC](https://github.com/stac-utils/pgstac) documentation.

### Examples of STAC + Zarr in the wild

STAC is already being used to organise and publish Zarr datasets in operational and research settings. A few good examples are:

- [**NOC STAC Catalog**](https://noc-msm.github.io/OceanDataStore/catalog/): The NOC Ocean Data Store publishes oceanographic Zarr datasets as STAC Collections and Items, making them discoverable and accessible via a web interface.
- [**EOPF Sentinel Explorer**](https://api.explorer.eopf.copernicus.eu/browser): The catalog exposes Sentinel Zarr samples as STAC Collections and Items, and you can inspect individual datasets directly in the browser. This is the link for the [STAC API](https://api.explorer.eopf.copernicus.eu/stac/api.html).
- [**Microsoft Planetary Computer**](https://browser.moregeo.it/external/planetarycomputer.microsoft.com/api/stac/v1/): Searchable spatiotemporal metadata describing Earth science datasets hosted by the Microsoft Planetary Computer. This is the link for the [STAC API](https://planetarycomputer.microsoft.com/api/stac/v1/docs).

These examples all follow the same idea: STAC describes the dataset, while the Zarr store remains the actual data asset. That makes the data easier to search, browse, and share without inventing a custom metadata format for each project

## PySTAC basics: creating Catalogs, Collections, and Items

[PySTAC](https://pystac.readthedocs.io/en/latest/) is a Python library for working with STAC objects in code, mirroring the JSON structure. You can create Catalogs, Collections, and Items programmatically, add metadata and assets, and save them as JSON files or feed them into a STAC API.

### Create a catalog

In this example, we will use the Zarr dataset created in the [NetCDF to Zarr conversion lesson](./10-conversion-workflow.html). The original data is a subset of the ERA5 reanalysis (`data/daily_swh/*.nc`) containing the significant wave height (`swh`) variable. Instead of converting the NetCDF files again, we will use the Zarr dataset that has already been generated and uploaded to the object store.

The first step is to create a root `Catalog`. A Catalog is the top-level entry point of a STAC hierarchy and can contain one or more Collections.

```python
import pystac

catalog = pystac.Catalog(
    id="Era5-Catalog",
    description="Root catalog for ERA5 datasets"
)
```

In this example, the Catalog represents all variables available from the ERA5 reanalysis. You can add Collections (or other Catalogs) to it using `add_child()`.

### Create a collection

A STAC Collection groups related Items and describes their shared metadata, such as their spatial and temporal extent, license, and description.

Here, we will create a Collection for the significant wave height (`swh`) dataset. To define its spatial and temporal extent, we first open the Zarr dataset and extract its bounding box and time range.

```python
import xarray as xr

url = "https://atlantis-vis-o.s3-ext.jc.rl.ac.uk/cloud-native-geoscience-course/daily_swh"
ds = xr.open_zarr(url, consolidated=True)
# Get spatial extent (bounding box)
lon_min, lon_max = float(ds.longitude.min()), float(ds.longitude.max())
lat_min, lat_max = float(ds.latitude.min()), float(ds.latitude.max())
# Get temporal extent (start and end time)
time_min, time_max = ds.time.min().values, ds.time.max().values

print(f"Spatial extent: lon [{lon_min}, {lon_max}], lat [{lat_min}, {lat_max}]")
print(f"Temporal extent: time [{time_min}, {time_max}]")
```

For this tutorial, we will define the Collection as covering the years 2000–2026, even though the example dataset represents only a subset of that period.

```python
from datetime import datetime

extent = pystac.Extent(
    spatial=pystac.SpatialExtent(bboxes=[[lon_min, lat_min, lon_max, lat_max]]),
    temporal=pystac.TemporalExtent(intervals=[[datetime(2000, 1, 1), datetime(2026, 12, 31)]])
)


collection = pystac.Collection(
    id="era5-zarr-swh",
    description="ERA5 surface Zarr datacubes for significant wave height (swh)",
    extent=extent,
    license="CC-BY-4.0"
)
```

Add the Collection to the Catalog:

```python
catalog.add_child(collection)
```

### Create an item pointing to a Zarr store


An Item represents a single dataset within a Collection. It contains metadata describing the dataset, including its spatial footprint, temporal coverage, and one or more Assets that point to the data itself.

We will create an Item representing the Zarr dataset. Its geometry and bounding box come from the dataset coordinates, while its temporal coverage is taken from the first and last timestamps.

```python
from shapely.geometry import box
import pandas as pd

bbox = [lon_min, lat_min, lon_max, lat_max]
geom = box(*bbox).__geo_interface__

# Convert time to datetime objects
start_datetime = pd.Timestamp(time_min).to_pydatetime()
end_datetime = pd.Timestamp(time_max).to_pydatetime()

item = pystac.Item(
    id="era5-zarr-swh-2025",
    geometry=geom,
    bbox=bbox,
    datetime=None,
    start_datetime=start_datetime,
    end_datetime=end_datetime,
    properties={
        "description": "ERA5 surface Zarr datacube for swh for 2025"
    }
)
```

Next, create an Asset that points to the Zarr store. The Asset contains the location of the data and describes its format.

```python
url = "https://atlantis-vis-o.s3-ext.jc.rl.ac.uk/cloud-native-geoscience-course/daily_swh"
asset = pystac.Asset(
    href=url,
    media_type="application/vnd+zarr", # Media type for Zarr stores. For icechunk stores, use "application/vnd.zarr+icechunk".
    roles=["data"]
)
```

Attach the Asset to the Item:


```python
# Add a Zarr asset
item.add_asset("data", asset)
```

Finally, add the Item to the Collection:

```python
collection.add_item(item)
```

This completes a minimal STAC hierarchy consisting of a **Catalog**, which contains a **Collection**, which in turn contains an **Item** pointing to a Zarr dataset stored in the object store.

### Save as a static catalog (JSON files)

To save the Catalog, Collection, and Item as static JSON files in a directory (e.g. `stac/`), use the `normalize_and_save()` method:

```python
catalog.normalize_and_save(
    "stac",
    catalog_type=pystac.CatalogType.SELF_CONTAINED,
)
```

This will create `catalog.json` (catalog), `era5-zarr-swh/collection.json` (collection), and `era5-zarr-swh/era5-zarr-swh-2025/era5-zarr-swh-2025.json` (item) in the `stac/` directory. It will normalize the links so that they are relative to the catalog root, making it easy to upload the entire directory to an object store or web server.

::::::::::::::::::::::::::::::::::::::: challenge

## Exercise 1 - Build a STAC catalog for your Zarr dataset

Using the Zarr dataset you generated in the NetCDF to Zarr conversion lesson:

1. Create a root `Catalog` with a clear description.
2. Create a `Collection` describing your dataset group (e.g. "Example Zarr conversions").
3. Create an `Item` whose `assets` include your Zarr store (Object Store URL).
4. Use `catalog.normalize_and_save()` to write a static STAC catalog to a `stac/` directory.

Questions:

- How did you choose the `id`, `description`, and `extent` for your Collection?
- What information did you include in the Item's `properties` and `assets` to make the Zarr dataset discoverable?

::::::::::::::: solution

Create a STAC catalog with a clear description, a Collection with an appropriate extent and license, and an Item that points to the Zarr store. Use PySTAC to save the catalog as JSON files. Include relevant metadata in the Collection and Item to make the dataset discoverable.

:::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::::::::::::::::

## STAC in databases: pgSTAC / PgPystac

Static STAC catalogs (JSON files in object storage or on a web server) are simple to publish and work well for small to medium datasets.

As the number of Items grows (e.g. millions of scenes or many Zarr datacubes), searching via JSON files becomes less efficient. Many deployments therefore store STAC metadata in a database (like [pgSTAC](https://github.com/stac-utils/pgstac)) and expose it via a [STAC API](https://github.com/radiantearth/stac-api-spec). This allows for fast queries, filtering, and pagination. Advantages of database-backed STAC:

- Fast spatial/temporal and attribute queries over large catalogs.
- Indexes and query planners handle complex filters.
- Single source of truth for STAC metadata that can be updated transactionally.

::::::::::::::::::::::::::::::::::::::: challenge

## Exercise 2 - Discuss JSON vs database-backed STAC

1. When would you prefer a simple static STAC catalog (JSON files in object storage)?
2. When would you prefer a database-backed STAC API (pgSTAC/PgPystac)?
3. How might you evolve from a static JSON catalog to a STAC API over time as your collection of Zarr datasets grows?

::::::::::::::: solution

Static STAC is ideal for simple, low-maintenance publishing, while database-backed STAC scales better for large collections and complex search. You might start with a static catalog for a small number of Zarr datasets, and as your collection grows, migrate to a database-backed STAC API to support efficient queries and updates.

:::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::::::::::::::::

:::::::::::::::::::::::::::::::::::::::::: checklist

## Visualising your catalog with STAC Browser

[STAC Browser](https://github.com/radiantearth/stac-browser) is a web application for browsing static STAC catalogs and STAC APIs as a human‑friendly UI.

To visualise your catalog:

1. Upload your `stac/` directory to a web server or object storage bucket. To upload the files, you can use `boto3` commands in python:

```python
import boto3
from botocore.config import Config
import os

s3 = boto3.client(
    "s3",
    endpoint_url="https://atlantis-vis-o.s3-ext.jc.rl.ac.uk",
    aws_access_key_id="your-access-key",
    aws_secret_access_key="your-secret-key",
    # This is necessary for JASMIN object store, but may not be needed for other S3-compatible stores.
    config=Config(
        request_checksum_calculation="when_required",
        response_checksum_validation="when_required",
    ),
)

bucket_name = "my-bucket" # Replace with your bucket name

for root, dirs, files in os.walk("stac"):
    for file in files:
        local_path = os.path.join(root, file)
        s3_path = os.path.relpath(local_path, "stac")
        s3.upload_file(local_path, bucket_name, f"stac/{s3_path}")
```

2. With the link to your `catalog.json`, you can use a hosted STAC Browser demo (e.g. <https://browser.moregeo.it/>) and enter your catalog URL.

::::::::::::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::::: challenge

## Exercise 3 (optional) - Publish and browse your catalog

1. Upload your `stac/` directory to the object store.
2. Use a hosted STAC Browser demo.
3. Explore:
   - Your root catalog and collection.
   - The Item representing your Zarr dataset.
   - Asset metadata and links.

Questions:

- How does browsing the STAC catalog compare to looking at raw JSON or Zarr paths?
- What would you add to your STAC metadata to make your Zarr datasets more understandable (keywords, variables, thumbnails, links to documentation)?

::::::::::::::: solution

STAC makes their Zarr datasets visible in a generic browser without writing custom UI code. You can see how the hierarchical structure of Catalog → Collection → Item → Assets helps users navigate and understand the datasets.

:::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::::::::::::::::


:::::::::::::::::::::::::::::::::::::::::: keypoints

- "STAC standardises how we describe and organise geospatial assets, including Zarr data cubes, making them easier to discover and integrate into cloud-native workflows."
- "Catalogs, Collections, and Items form a three-layer hierarchy: Catalog as entry point, Collection as grouped datasets, Item as individual spatiotemporal assets with linked data files."
- "PySTAC provides a Python API to create and manage STAC Catalogs, Collections, and Items, and to save them as static JSON catalogs or feed them into STAC APIs."
- "Zarr stores in object storage can be referenced as STAC Item assets, connecting cloud-native array data to the broader STAC ecosystem."
- "STAC metadata can be stored either as static JSON (simple publishing) or in databases like pgSTAC/PgPystac (scalable, queryable STAC APIs), and tools like STAC Browser can visualise both."

::::::::::::::::::::::::::::::::::::::::::::::::::
