---
title: Zarr Data Organization in the Cloud with STAC
teaching: 25
exercises: 15
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

- **Catalog** - provides a simple, hierarchical structure of links that group Items (and other Catalogs/Collections) for browsing.
- **Collection** - extends Catalog with additional metadata (spatial/temporal extent, license, keywords, providers) for a coherent group of Items.
- **Item** - describes a single spatiotemporal asset (e.g. one satellite scene, one product, or one Zarr datacube at a given extent/time).

The actual data files (e.g. Zarr stores, NetCDF files, Cloud Optimized GeoTIFFs) are referenced as **assets** in the Item. Each asset has a URL (or S3 path), media type, and optional roles (e.g. "data", "thumbnail", "metadata").

For cloud‑native workflows, STAC is important because:

- It standardises how we describe and organise datasets (including Zarr) so that tools and portals can discover them without custom code for each provider.
- It works equally well as **static JSON files in object storage** or as **dynamic APIs backed by databases**, making it flexible for different scales and infrastructures.
- Many ecosystem tools (STAC Browser, PySTAC, STAC API servers, pgSTAC) already understand STAC, so using STAC makes our Zarr data "plug‑and‑play" in that ecosystem.

### STAC Catalog

A STAC Catalog is a simple JSON object that links to child Catalogs, Collections, and Items:

```json
{
  "stac_version": "1.0.0",
  "type": "Catalog",
  "id": "root-catalog",
  "description": "Root catalog for our cloud-native Zarr data",
  "links": [
    {
      "rel": "child",
      "href": "collections/era5-collection.json",
      "type": "application/json"
    }
  ]
}
```

The Catalog:

- Acts as an entry point.
- Organises content hierarchically via `links` with `rel` values like `child`, `item`, `collection`.

Catalog spec: <https://github.com/radiantearth/stac-spec/tree/master/catalog-spec>


### STAC Collection

A STAC Collection extends Catalog with extra fields describing a coherent group of Items:

```json
{
  "stac_version": "1.0.0",
  "type": "Collection",
  "id": "era5-zarr",
  "description": "ERA5 surface Zarr datacubes",
  "license": "CC-BY-4.0",
  "extent": {
    "spatial": { "bbox": [[-180, -90, 180, 90]] },
    "temporal": { "interval": [["1950-01-01T00:00:00Z", null]] }
  },
  "links": [
    {
      "rel": "item",
      "href": "../items/era5-zarr-2020.json",
      "type": "application/json"
    }
  ],
  "summaries": {
    "platform": ["ERA5"],
    "keywords": ["reanalysis", "atmosphere", "zarr"]
  }
}
```

Here:

- `extent` gives spatial and temporal coverage.
- `summaries` can list common attributes across Items (e.g. platforms, variables).

Collection spec: <https://github.com/radiantearth/stac-spec/tree/master/collection-spec>



### STAC Item

A STAC Item is the smallest unit in STAC - an atomic bundle of inseparable metadata and assets, representing data at a specific place and time. It is a GeoJSON Feature with additional STAC fields:

```json
{
  "stac_version": "1.0.0",
  "type": "Feature",
  "id": "example-item",
  "bbox": [5.5, 46.0, 8.0, 47.4],
  "geometry": { "type": "Polygon", "coordinates": [/* ... */] },
  "properties": {
    "datetime": "2020-12-11T22:38:32Z"
  },
  "collection": "example-collection",
  "links": [],
  "assets": {
    "data": {
      "href": "s3://my-bucket/path/to/data.zarr",
      "type": "application/vnd+zarr",
      "roles": ["data"]
    }
  }
}
```

Key points:

- It is a GeoJSON Feature with additional STAC fields (`stac_version`, `assets`, `links`, etc.).
- `assets` is where we point to actual data files - here a Zarr store in object storage.

See the Item spec for more details: <https://github.com/radiantearth/stac-spec/tree/master/item-spec>


## STAC and cloud-native Zarr workflows

For Zarr datasets in object storage:

- Each Zarr **store** is typically referenced as an **asset** in a STAC Item pointing to the store's root (e.g. `s3://bucket/path.zarr`).
- Collections can group multiple Zarr stores that share a product (e.g. "ERA5 surface daily Zarr cubes" or "CMIP6 model X Zarr outputs").
- Catalogs provide top-level organisation (e.g. per provider, per project).

Benefits for cloud-native workflows:

- STAC makes Zarr datasets **discoverable** by spatial/temporal criteria and keywords, rather than "hidden" as raw paths in buckets.
- Tools like **STAC Browser** can display and search your STAC catalog or API, making Zarr assets visible in a generic UI: <https://github.com/radiantearth/stac-browser>
- STAC API servers backed by databases (e.g. pgSTAC/PgPystac) can support fast, scalable search over many Zarr datasets. See STAC API and pgSTAC docs (e.g. <https://github.com/stac-utils/pgstac>).


## PySTAC basics: creating Catalogs, Collections, and Items

[PySTAC](https://pystac.readthedocs.io/en/latest/) is a Python library for working with STAC objects in code, mirroring the JSON structure.[https://pystac.readthedocs.io/en/latest/quickstart.html](https://pystac.readthedocs.io/en/latest/quickstart.html)

### Create a catalog

```python
import pystac

catalog = pystac.Catalog(
    id="root-catalog",
    description="Root catalog for our cloud-native Zarr data"
)
```

### Create a collection

```python
from datetime import datetime

extent = pystac.Extent(
    spatial=pystac.SpatialExtent(bboxes=[[-180, -90, 180, 90]]),
    temporal=pystac.TemporalExtent(intervals=[[datetime(1950, 1, 1), None]])
)

collection = pystac.Collection(
    id="era5-zarr",
    description="ERA5 surface Zarr datacubes",
    extent=extent,
    license="CC-BY-4.0"
)
catalog.add_child(collection)
```

### Create an item pointing to a Zarr store

Assume you have a Zarr dataset from the conversion lesson, stored at `s3://my-bucket/converted.zarr` or `data/converted.zarr` on disk.

```python
from shapely.geometry import box

bbox = [-180.0, -90.0, 180.0, 90.0]
geom = box(*bbox).__geo_interface__

item = pystac.Item(
    id="era5-zarr-2020",
    geometry=geom,
    bbox=bbox,
    datetime=datetime(2020, 1, 1),
    properties={
        "title": "ERA5 surface Zarr datacube for 2020",
        "source_format": "NetCDF → Zarr"
    }
)

# Add a Zarr asset
item.add_asset(
    "zarr-data",
    pystac.Asset(
        href="s3://my-bucket/converted.zarr",  # or a local path
        media_type="application/vnd+zarr",
        roles=["data"]
    )
)

collection.add_item(item)
```

### Save as a static catalog (JSON files)

```python
catalog.normalize_hrefs("stac")  # write into ./stac directory
catalog.save()
```

This will create `catalog.json`, `collections/era5-zarr.json`, and `items/era5-zarr-2020.json` in the `stac/` directory. See the PySTAC quickstart for more details: <https://pystac.readthedocs.io/en/latest/quickstart.html>.

### Upload the catalog to object storage

To upload the files, you can use `boto3` commands in python:

```python
import boto3
s3 = boto3.client('s3')
for root, dirs, files in os.walk("stac"):
    for file in files:
        local_path = os.path.join(root, file)
        s3_path = os.path.relpath(local_path, "stac")
        s3.upload_file(local_path, "my-bucket", f"stac/{s3_path}")
```

::::::::::::::::::::::::::::::::::::::: challenge

## Exercise 1 - Build a STAC catalog for your Zarr dataset

Using the Zarr dataset you generated in the NetCDF→Zarr conversion lesson:

1. Install PySTAC if needed:

   ```bash
   pip install pystac
   ```

2. In Python:

   - Create a root `Catalog` with a clear description.
   - Create a `Collection` describing your dataset group (e.g. "Example Zarr conversions").
   - Create an `Item` whose `assets` include your Zarr store (local path or S3 URL).

3. Use `catalog.normalize_hrefs("stac")` and `catalog.save()` to write a static STAC catalog to a `stac/` directory.

Questions:

- How did you choose the `id`, `description`, and `extent` for your Collection?
- What information did you include in the Item's `properties` and `assets` to make the Zarr dataset discoverable?

::::::::::::::: solution

Learners should end up with a small STAC catalog on disk that describes their Zarr dataset in terms of space, time, and metadata.
They will see how PySTAC mirrors the STAC JSON structure and how easy it is to attach a Zarr store as an Item asset.

:::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::::::::::::::::


## STAC in databases: pgSTAC / PgPystac

Static STAC catalogs (JSON files in object storage or on a web server) are simple to publish and work well for small to medium datasets.

As the number of Items grows (e.g. millions of scenes or many Zarr datacubes), searching via JSON files becomes less efficient. Many deployments therefore:

- Store STAC objects in a **PostgreSQL/PostGIS database** using schemas like **pgSTAC** (often used via *PgPystac* or other tooling).
- Provide a **STAC API** that exposes `/stac/search` and other endpoints backed by database queries.

Advantages of database-backed STAC:

- Fast spatial/temporal and attribute queries over large catalogs.
- Indexes and query planners handle complex filters.
- Single source of truth for STAC metadata that can be updated transactionally.

Useful references:

- pgSTAC: <https://github.com/stac-utils/pgstac>
- STAC API overview: <https://github.com/radiantearth/stac-api-spec>


::::::::::::::::::::::::::::::::::::::: challenge

## Exercise 2 - Discuss JSON vs database-backed STAC

In pairs or small groups, discuss:

1. When would you prefer a simple static STAC catalog (JSON files in object storage)?
2. When would you prefer a database-backed STAC API (pgSTAC/PgPystac)?
3. How might you evolve from a static JSON catalog to a STAC API over time as your collection of Zarr datasets grows?

Write a brief summary of your conclusions.

::::::::::::::: solution

Learners should recognise that static STAC is ideal for simple, low‑maintenance publishing, while database-backed STAC scales better for large collections and complex search.
They will see that PySTAC lets them use the same object model regardless of whether the final catalog lives as JSON or in a database.

:::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::::::::::::::::

:::::::::::::::::::::::::::::::::::::::::: callout

## Visualising your catalog with STAC Browser

[STAC Browser](https://github.com/radiantearth/stac-browser) is a web application for browsing static STAC catalogs and STAC APIs as a human‑friendly UI.

To visualise your catalog:

1. Serve your `stac/` directory over HTTP (for example with a simple Python server):

   ```bash
   cd stac
   python -m http.server 8000
   # catalog is now at http://localhost:8000/catalog.json
   ```

2. Serve the stac catalog already in the object storage bucket (if you uploaded it) via a public URL or signed URL.

::::::::::::::::::::::::::::::::::::::::::::::::::


::::::::::::::::::::::::::::::::::::::: challenge

## Exercise 3 (optional) - Publish and browse your catalog

1. Serve your `stac/` directory with a simple local web server.
2. Run STAC Browser locally (Docker) or use a hosted STAC Browser demo.
3. Configure STAC Browser to use your catalog URL (e.g. `http://localhost:8000/catalog.json`).
4. Explore:
   - Your root catalog and collection.
   - The Item representing your Zarr dataset.
   - Asset metadata and links.

Questions:

- How does browsing the STAC catalog compare to looking at raw JSON or Zarr paths?
- What would you add to your STAC metadata to make your Zarr datasets more understandable (keywords, variables, thumbnails, links to documentation)?

::::::::::::::: solution

STAC makes their Zarr datasets visible in a generic browser without writing custom UI code. You can see how the hierarchical structure of Catalog → Collection → Item helps users navigate and understand the datasets.

:::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::::::::::::::::


:::::::::::::::::::::::::::::::::::::::::: keypoints

- "STAC standardises how we describe and organise geospatial assets, including Zarr data cubes, making them easier to discover and integrate into cloud-native workflows."
- "Catalogs, Collections, and Items form a three-layer hierarchy: Catalog as entry point, Collection as grouped datasets, Item as individual spatiotemporal assets with linked data files."
- "PySTAC provides a Python API to create and manage STAC Catalogs, Collections, and Items, and to save them as static JSON catalogs or feed them into STAC APIs."
- "Zarr stores in object storage can be referenced as STAC Item assets, connecting cloud-native array data to the broader STAC ecosystem."
- "STAC metadata can be stored either as static JSON (simple publishing) or in databases like pgSTAC/PgPystac (scalable, queryable STAC APIs), and tools like STAC Browser can visualise both."

::::::::::::::::::::::::::::::::::::::::::::::::::
