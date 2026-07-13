---
title: Zarr - Data Model, Metadata, and Chunked Storage
teaching: 20
exercises: 10
---

:::::::::::::::::::::::::::::::::::::::::: objectives

- "Explain Zarr's core data model: groups, arrays, stores, and chunks."
- "Describe how Zarr metadata works and how conventions build on it."
- "Understand Zarr's chunked storage and why it matters for oceanography, climate, and meteorology."
- "Practice opening a Zarr store with Python and xarray to see the structure in practice."

::::::::::::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::::::::: questions

- "What is Zarr, and how does its data model differ from formats like NetCDF or HDF5?"
- "How does Zarr store metadata about arrays and groups?"
- "What is chunked storage in Zarr, and why is it useful for large multidimensional datasets?"
- "How is Zarr changing the way ocean, climate, and meteorological data are stored and accessed?"

::::::::::::::::::::::::::::::::::::::::::::::::::


## Zarr in context

Zarr is an open-source format and data model for storing chunked, N-dimensional arrays in a way that works naturally with cloud object storage and file systems. It was originally developed in the scientific Python community, and has since grown into a multi-language ecosystem with implementations in [Python](https://zarr.readthedocs.io/en/stable/), [Rust](https://zarrs.dev/), [Julia](https://github.com/JuliaIO/Zarr.jl), [Matlab](https://github.com/mathworks/MATLAB-support-for-Zarr-files), [R](https://cran.r-project.org/web/packages/zarr/index.html), [JavaScript](https://zarrita.dev/)  and others. In the Zarr website and documentation, you can find a list of [implementations](https://zarr.dev/).

For Earth and climate data, Zarr is being adopted by major providers (e.g. ESA's Sentinel products and National Weather Service-USA) and projects such as Earth Data Hub, Pangeo, and DestinE, precisely because it scales to petabyte-sized datasets while remaining accessible from tools like xarray.

## Zarr as a data model

At a high level, Zarr is a way to represent large N-dimensional arrays as smaller pieces that can be stored and accessed efficiently.

A Zarr dataset is usually organised into three main ideas:

- **Arrays**: N-dimensional arrays of a single data type, split into chunks.
- **Groups**: hierarchical containers that can hold arrays and child groups, similar to folders or HDF5 groups.
- **Stores**: the underlying storage system that holds the data and metadata, such as a local directory, an S3 bucket, or another key-value backend.

A typical climate or ocean dataset might use one group containing arrays such as `temperature`, `salinity`, `u_wind`, and `v_wind`, each organised by dimensions such as `time`, `lat`, `lon`, and `level`. Instead of one monolithic file, each array is split into chunks, and each chunk can be stored and read separately.

![Zarr Organisation. Source: https://tutorial.xarray.dev/intermediate/intro-to-zarr.html](fig/zarr_organisation.png){alt="Zarr organisation, showing groups and arrays with chunked storage."}

## Zarr v2

Zarr version 2 is the older, widely used storage specification. In Zarr v2, the dataset is organised as a directory-like structure with a few small metadata files plus folders for groups, arrays, and chunks.

At the top level, you usually see:

- [**`.zgroup`**](data/zarr_group_metadata.json) for group metadata.
- [**`.zattrs`**](data/zarr_group_attributes.json) for user-defined attributes.
- [**`.zmetadata`**](data/zarr_consolidated_metadata.json) for consolidated metadata, when enabled.

Inside the group, each array usually appears as its own directory, and that array directory contains:

- [**`.zarray`**](data/zarr_array_metadata.json) for array metadata.
- [**`.zattrs`**](data/zarr_array_attributes.json) for array-specific attributes.
- Chunk files, named by their chunk coordinates, such as `0.0.0`, `0.0.1`, `1.0.0`, and so on.

A simple Zarr v2 layout might look like this:

```text
sos_abs.zarr/
├── .zattrs
├── .zgroup
├── .zmetadata
├── sos_abs/
│   ├── .zarray
│   ├── .zattrs
│   ├── 0.0.0
│   ├── 0.0.1
│   ├── 0.1.0
│   └── ...
├── time_centered/
│   ├── .zarray
│   ├── .zattrs
│   └── ...
├── time_counter/
│   ├── .zarray
│   ├── .zattrs
│   └── ...
├── x/
│   ├── .zarray
│   ├── .zattrs
│   └── ...
└── y/
    ├── .zarray
    ├── .zattrs
    └── ...
```

In this example, `sos_abs` is one data variable, while `time_centered`, `time_counter`, `x`, and `y` are coordinate arrays. Each array has its own `.zarray` file describing the array shape, data type, chunk layout, fill value, and other storage settings.

This tells us that the `sos_abs` array is three-dimensional, stored in chunks of size `1 × 250 × 250`, and encoded as 32-bit floating-point values. The actual chunk data is stored separately in files such as `0.0.0`, `0.0.1`, `0.1.0`, and so on.

A key feature of Zarr v2 is that the metadata is small, human-readable, and easy to inspect without opening the full dataset. That makes it convenient for browsing dataset structure and understanding how the data are organised before any analysis begins.


![Zarr V2. Source: https://aws.amazon.com/pt/blogs/publicsector/decrease-geospatial-query-latency-minutes-seconds-using-zarr-amazon-s3/](fig/zarrv2.png){alt="Zarr V2 metadata structure, showing .zgroup, .zarray, and .zattrs files."}

## Zarr v3

Zarr version 3 is the newer core specification. It keeps the same basic idea of chunked arrays and groups, but it updates the metadata structure and some of the terminology to better support modern scientific and cloud-native use cases.

At a high level, a Zarr v3 store still contains groups, arrays, and chunks, but the metadata is more explicit and consolidated. The main difference is that Zarr v3 uses a more explicit metadata layout. Instead of several small hidden JSON files spread through a directory tree, Zarr v3 uses `zarr.json` files to describe groups and arrays. In the metadata itself, the terminology has changed slightly to make it clearer and more flexible. Some important changes from v2 to v3 are:

- `dtype` becomes `data_type`.
- `chunks` becomes `chunk_grid`.
- `dimension_separator` becomes `chunk_key_encoding`.
- `order` is replaced by the transpose codec.
- `filters` and `compressor` are replaced by a more general `codecs` field.


A simplified Zarr v3 structure might look like this:

```text
sos_abs.zarr/
├── zarr.json
├── sos_abs/
│   ├── zarr.json
│   ├── c/0/0/0
│   ├── c/0/0/1
│   ├── c/0/1/0
│   └── ...
├── time_centered/
│   ├── zarr.json
│   └── c/0
├── time_counter/
│   ├── zarr.json
│   └── c/0
├── x/
│   ├── zarr.json
│   └── c/0
└── y/
    ├── zarr.json
    └── c/0
```

In this example, the root `zarr.json` describes the top-level group, and each array has its own `zarr.json` file describing its shape, chunk grid, data type, codecs, and other metadata. The chunk data are stored separately under chunk key paths such as `c/0/0/0`.


![Zarr V3. Source: https://earthmover.io/blog/what-is-zarr/](fig/zarrv3.png){alt="Zarr V3 metadata structure, showing zarr.json files"}

An [example of a Zarr v3 metadata file for an array](data/zarr_v3_array_metadata.json) might look like this:

```json
{
  "zarr_format": 3,
  "data_type": "<f4",
  "shape": [3650, 1800, 3600],
  "chunk_grid": [10, 100, 100],
  "codecs": [
    {
      "id": "zlib",
      "level": 1
    }
  ],
  "fill_value": null,
  "dimension_separator": "/"
}
```

Zarr v3 also adds more explicit support for features such as sharding, where several chunks can be grouped together inside a larger storage object. This helps reduce the overhead of managing very large numbers of tiny files or keys, especially in cloud object storage.

## Chunked storage: how Zarr stores large arrays

Chunking is central to Zarr's design. Instead of storing one very large array as a single block, Zarr splits it into many smaller pieces called **chunks**. Each chunk is a small N-dimensional block of the array, for example a subset in `time × lat × lon`, and each chunk can be stored and read separately.

You can think of this like cutting a very large map into tiles. If you only want to look at one region, you do not need to unroll the whole map — you only fetch the tiles you need. Zarr does the same thing for multidimensional data.

For environmental data, chunking is useful because:

- **Selective reads**: a time series at one point, one region, or one variable can often be read without scanning the whole dataset.
- **Parallelism**: different chunks can be processed at the same time by different workers.
- **Compression**: each chunk can be compressed separately, which can reduce storage costs and data transfer.

This is one reason Zarr fits cloud workflows well: object storage and HTTP-based access work naturally when data is organised into many addressable pieces rather than one large monolithic file.


### A simple way to think about chunks

Imagine an ocean temperature dataset with dimensions:

- `time = 3650`
- `lat = 1800`
- `lon = 3600`

If this were stored as one giant array, even small operations could require reading a very large amount of data. With chunking, the dataset can instead be divided into smaller blocks, such as:

- one chunk per group of timesteps,
- one chunk per spatial tile,
- or a combination of both.

When a user asks for “the temperature time series at this point” or “this region for this month”, the software can request only the chunks that overlap that query, rather than the whole array.

![How chunking affects data access.](fig/chunk_effect.png){alt="Effect of chunking on data access."}

:::::::::::::::::::::::::::::::::::::::::: callout

## Chunking also has trade-offs

Chunking is powerful, but it is not free. If chunks are too large, each read may bring in much more data than needed. If chunks are too small, the dataset may contain a huge number of tiny objects, and managing all those requests can become inefficient, especially in cloud object storage.

So there is always a balance:

- larger chunks can reduce metadata and request overhead,
- smaller chunks can improve fine-grained access,
- the best chunk shape depends on the kinds of analysis people usually perform.

This trade-off is one reason newer Zarr work has added **sharding**.

::::::::::::::::::::::::::::::::::::::::::::::::::



## Why Zarr matters for oceanography, climate, and meteorology

For these disciplines, Zarr brings several important shifts:

- **From files to data cubes**: archives can be published as coherent Zarr "stores" representing large data cubes (e.g. global ERA5 climate fields or Sentinel EO products) rather than thousands of individual NetCDF/GRIB files.
- **Direct cloud access**: scientists can open datasets directly from S3/HTTPS in notebooks or applications, reading only needed chunks instead of downloading entire files.
- **Interoperability and tooling**: Zarr integrates well with xarray, dask, kerchunk, TileDB, Icechunk, and visualization tools such as browzarr and browser-based explorers, enabling rich, interactive workflows.
- **Community-driven standards**: the Zarr Summit outcomes show a strong momentum for conventions, cross-language conformance, sparse arrays, and linked arrays, helping climate and ocean communities align around robust, shared practices.

In short, Zarr doesn't replace scientific semantics like CF or the Common Data Model—it gives us a new storage and access layer that works natively with cloud infrastructure and modern analysis tools.

::::::::::::::::::::::::::::::::::::::: challenge

## Exercise 1 - Inspecting a Zarr store with Python

Assume you have a Zarr store in your lesson data directory, for example `data/ocean_temperature.zarr`, containing a hierarchical dataset with several arrays.

1. Use the Python `zarr` library to open the store.
2. Inspect the group hierarchy and list available arrays.
3. Explore the metadata files (`.zarray`, `.zgroup`, `.zattrs`) for one array.

```python
import zarr

store = zarr.open_group("data/ocean_temperature.zarr", mode="r")
print(store)             # Overview of groups and arrays
print(list(store.arrays()))  # List arrays
print(list(store.groups()))  # List child groups

# Inspect a specific array
temp = store["temperature"]
print(temp)
print(temp.shape, temp.chunks, temp.dtype)

# Access attributes
print(temp.attrs)
```

Questions:

- How are arrays and groups organised in the store (e.g. top-level group, subgroups)?
- What chunk shape and data type does the `temperature` array use?
- Which attributes (metadata) are present on the array and group?

::::::::::::::: solution

The Zarr store behaves like a hierarchical container: the `Group` printed by `zarr.open_group` lists child arrays and groups, and each array shows shape, chunk shape, and data type.
Inspecting `temp.attrs` reveals arbitrary metadata such as units, long names, CF standard names, and possibly application-specific tags, illustrating how Zarr combines a simple core model with flexible attributes.

:::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::::: challenge

## Exercise 2 - Thinking about chunked storage

Using the same Zarr dataset, answer these conceptual questions:

1. If your array has dimensions `(time, lat, lon)` and a chunk shape `(10, 100, 100)`, what does each chunk represent in terms of space and time?
2. How might different chunk shapes (e.g. more time vs. more space in each chunk) affect typical operations, such as:
   - Reading a time series at one grid point.
   - Computing a spatial mean for each time step.
3. Imagine you are designing a Zarr layout for a high‑resolution global reanalysis. What types of operations would you prioritise when choosing chunk shapes?

You do not need to change any code—focus on reasoning about chunks and operations.

::::::::::::::: solution

A chunk shape such as `(10, 100, 100)` represents 10 time steps over a 100×100 spatial block.
Chunk shapes that are "tall in time" can be efficient for time series at specific locations, while chunk shapes that cover larger spatial regions may be better for spatial aggregates; in practice, chunk shapes are a compromise based on dominant workloads.
This exercise prepares you to think critically about chunking decisions, sharding, and performance optimisation covered in later lessons (e.g. on cloud-native formats, Zarr V3 features, and tools like Icechunk and Virtualizarr).

:::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::::::::::::::::
