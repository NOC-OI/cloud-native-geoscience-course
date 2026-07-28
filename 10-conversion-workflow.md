---
title: Conversion Workflow of Traditional Formats to Zarr
teaching: 25
exercises: 25
---

:::::::::::::::::::::::::::::::::::::::::: objectives

- "Explain the main steps in converting NetCDF datasets to Zarr."
- "Choose appropriate chunking strategies for the target Zarr dataset."
- "Use xarray, Dask, and `to_zarr` to convert NetCDF to Zarr in parallel."
- "Upload Zarr stores to object storage and verify basic analyses on the converted data."

::::::::::::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::::::::: questions

- "Why convert NetCDF data to Zarr, and what changes in the way we access and process data?"
- "How do we choose chunk sizes for Zarr when starting from NetCDF files?"
- "How can we use Dask and xarray to convert and write data to Zarr efficiently?"
- "How do we test that the converted data is usable and correct (e.g. computing mean values)?"

::::::::::::::::::::::::::::::::::::::::::::::::::


## Why convert NetCDF to Zarr?

NetCDF is widely used for ocean, climate, and meteorological data and works well on local filesystems and HPC storage.
However, as datasets grow and we move to cloud and parallel computing, Zarr offers several advantages:

- Chunked storage aligned with object stores and HTTP/S3 access.
- Easy parallel reads/writes of chunks by Dask or other frameworks.
- Flexible layout for large collections (many NetCDF files → one Zarr dataset).

Converting NetCDF to Zarr does not change the scientific content, but it changes how data is organised and accessed, enabling cloud-native workflows and more efficient analysis at scale.

### Converting from NetCDF to Zarr with xarray

To convert NetCDF to Zarr, we typically use xarray's `to_zarr` method, optionally with Dask for parallelism. The main steps are:

1. Inspect the NetCDF dataset(s) to understand variables, dimensions, coordinates, and existing chunking.

We will use a simple example NetCDF dataset to illustrate the conversion workflow.

```python
import xarray as xr

ds_nc = xr.open_dataset("data/ocean_temperature.nc")
print(ds_nc)
print(ds_nc.dims)
print(ds_nc.data_vars)
print(ds_nc.coords)
print(ds_nc.encoding)  # may show chunking and compression info
```

2. Decide how the Zarr dataset should be chunked

The chunking scheme should match the way the data will be used later. A simple example chunking choice might look like this:

```python
ds_zarr = ds_nc.chunk({"valid_time": 10, "latitude": 100, "longitude": 100})
```

3. Write the Zarr store

Once the chunking looks right, write the dataset to Zarr.

```python
ds_zarr.to_zarr("data/example.zarr", mode="w")
```

Now you have a Zarr store that can be uploaded to object storage and accessed in parallel by multiple clients.


::::::::::::::::::::::::::::::::::::::: challenge

## Exercise 1 - Convert a NetCDF dataset to Zarr

Using the NetCDF dataset provided (`data/ocean_temperature.nc`), open a single file with `open_dataset`, chunk it with `{"valid_time": 10, "latitude": 100, "longitude": 100}`, and write it to a local Zarr store.

::::::::::::::: solution

```python
import xarray as xr
ds_nc = xr.open_dataset("data/ocean_temperature.nc")
ds_zarr = ds_nc.chunk({"valid_time": 10, "latitude": 100, "longitude": 100})
ds_zarr.to_zarr("data/example.zarr", mode="w")
```

:::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::::::::::::::::


## Overall workflow: NetCDF → Zarr → object store

Typical steps:

1. **Understand the NetCDF input**: check variables, dimensions, coordinates, and chunking.
2. **Decide chunking strategy for Zarr**: based on expected access patterns and approximate chunk sizes in MB.
3. **Open NetCDF with xarray**: using `open_dataset` or `open_mfdataset` for multiple files.
4. **Apply chunking and encoding**: use `.chunk()` and define any compression or encoding options.
5. **Write Zarr to local storage** (Optional): using `Dataset.to_zarr()`, possibly with Dask for parallel writes.
6. **Upload Zarr store to object storage**: using cloud CLI tools or filesystem libraries.
7. **Verify by reopening from object storage and running analyses** (e.g. computing mean values).

We'll walk through these steps conceptually and via exercises. For the example below and the exercises, we will use a subset of the ERA5 reanalysis dataset related to significant wave height (`swh`). The data is located in the `data/daily_swh` directory and is provided as multiple NetCDF files, one per day. The goal is to convert this collection of NetCDF files into a single Zarr store that can be accessed efficiently in parallel.

## Step 1 - Inspect NetCDF inputs

Before converting, inspect the NetCDF file(s) you plan to convert.

Example:

```python
import xarray as xr

ds_nc = xr.open_dataset("data/daily_swh/swh_2025-01-01.nc")
print(ds_nc)
print(ds_nc.dims)
print(ds_nc.data_vars)
print(ds_nc.coords)
print(ds_nc.encoding)  # may show chunking and compression info
```

Questions to answer:

- Are there multiple files (e.g. one per time slice) or just a single file?
- Which dimensions are present (e.g. `time`, `lat`, `lon`, `depth`, `member`)?
- Are variables chunked already, and if so, how?
- Are all files consistent in terms of variables and coordinates if you plan to use `open_mfdataset`?

For multiple NetCDF files:

```python
ds_nc_multi = xr.open_mfdataset(
    "data/daily_swh/swh_*.nc",
    combine="by_coords",
)
print(ds_nc_multi)
```

It is important to mention that `open_mfdataset` will combine multiple files into a single xarray dataset, but it requires that the variables and coordinates are consistent across files. And we are not loading the data into memory yet. Xarray will lazily load data as needed.

## Step 2 - Decide chunking strategy for Zarr

Chunks in Zarr have a major impact on performance. A common rule of thumb is to aim for compressed chunk sizes in the range of 10-100 MB, but exact choices depend on workloads and hardware.

Consider:

- Dimensions: which ones correspond to time, space, depth, ensemble member?
- Workloads: time series at points, spatial means, regional subsets, ensemble statistics.
- Storage: local HPC disk vs object store; network bandwidth; parallel framework (Dask).

Example chunk strategy for a dataset with dimensions `(time, latitude, longitude)`:

- If time series are common: `{"time": 360, "latitude": 100, "longitude": 100}`.
- If spatial maps per time are common: `{"time": 1, "latitude": 361, "longitude": 720}`.

You can experiment with different chunk shapes and use Dask's dashboard and performance measurements to refine choices.

## Step 3 - Convert NetCDF to Zarr with xarray and Dask

Once chunking decisions are made, you can use xarray's `to_zarr` method to write out the dataset. This can be done serially or in parallel using Dask.

Parallel conversion with Dask:

```python
from dask.distributed import Client
import xarray as xr

client = Client(n_workers=2, threads_per_worker=2)

ds_nc = xr.open_mfdataset("data/daily_swh/swh_*.nc", combine="by_coords")

# Choose chunking
ds_chunked = ds_nc.chunk({"time": 12, "latitude": 100, "longitude": 100})

# Write to Zarr; use consolidated metadata if desired
ds_chunked.to_zarr("data/era5_swh.zarr", mode="w", consolidated=True)
```

:::::::::::::::::::::::::::::::::::::::::: callout

## Consolidated metadata

When writing a Zarr store, you can choose to consolidate metadata. This means the store writes a single metadata index file, usually called `.zmetadata`, that collects the metadata for all arrays and attributes in one place.

This is useful because opening the dataset can be faster, especially when the store has many variables or many chunks. Instead of reading lots of small metadata files one by one, xarray can read the consolidated metadata in a single step.

A good practice is to use consolidated metadata when the dataset will be read many times after writing, especially in cloud or object-store workflows. It is usually a good default for analysis-ready Zarr stores.

::::::::::::::::::::::::::::::::::::::::::

:::::::::::::::::::::::::::::::::::::::::: callout

## Encoding and compression

When writing Zarr stores, you can choose how each variable is encoded and compressed. In most teaching and analysis workflows, the goal is not to maximise compression at all costs, but to choose a strategy that balances file size, speed, and ease of reading.

A simple example looks like this:

```python
import zarr
from zarr.codecs import BloscCodec, BloscShuffle
import xarray as xr

ds = xr.open_dataset("data/ocean_temperature.nc")
compressor = BloscCodec(cname="zstd", clevel=3, shuffle=BloscShuffle.shuffle)
encoding = {
    "sst": {
        "compressors": compressor,
        "chunks": (10, 100, 100),
    }
}

ds.to_zarr(
    "data/example_compressed.zarr",
    mode="w",
    encoding=encoding,
    consolidated=True,
)
```

In this example:
- `zstd` gives good compression with reasonable speed.
- `clevel=3` is a moderate compression level, which is often a good compromise between speed and size.
- `shuffle=2` usually improves compression for many numerical datasets.
- The same compressor is applied to each data variable through the `encoding` dictionary.

For scientific data, moderate compression is often a good default because:

- It reduces storage size.
- It can lower network transfer costs.
- It usually keeps reads and writes fast enough for practical use.

Very aggressive compression can save more space, but it may slow down writing and reading. For large workflows, that trade-off is often not worth it.

::::::::::::::::::::::::::::::::::::::::::


## Step 4 - Upload Zarr to object storage

After successfully writing Zarr locally, you can upload the Zarr directory store to an object store (AWS S3, GCS, MinIO, etc.).

Examples for s3cmd (using CLI tools):

```bash
s3cmd mb s3://my-bucket
s3cmd put -r data/era5_swh.zarr s3://my-bucket/era5_swh.zarr
```

Once uploaded, you can access the Zarr store directly from the object store using xarray and filesystem adapters, as in previous lessons.

In our lesson here, instead of generating the zarr dataset locally and upload then later to the object store, we will directly write the zarr dataset to the object store using fsspec. This is a more efficient approach and avoids unnecessary local storage usage.

```python
import xarray as xr
import fsspec

store_url = "s3://my-bucket/era5_swh.zarr"

storage_options = {
    "key": "your-access-key",
    "secret": "your-secret-key",
    "client_kwargs": {
        "endpoint_url": "https://atlantis-vis-o.s3-ext.jc.rl.ac.uk",
    },
}

# Create a mapper for the object store
mapper = fsspec.get_mapper(
    store_url,
    **storage_options,
)

# Write the dataset directly to the object store
ds_chunked.to_zarr(
    mapper,
    mode="w",
    consolidated=True,
)
```

## Step 5 - Verify with a simple analysis

To verify that the converted Zarr dataset is usable, run a simple analysis, such as computing a mean value.

Example:

```python
import xarray as xr
import fsspec

mapper = fsspec.get_mapper("s3://my-bucket/era5_swh.zarr")
ds_zarr = xr.open_zarr(mapper, consolidated=True)

# You can also access the dataset directly, as it is in a public bucket:
ds_zarr = xr.open_zarr("https://my-bucket/era5_swh.zarr", consolidated=True)

# Choose a variable and compute a global mean over space
var = ds_zarr["swh"]
global_mean = var.mean(dim=("latitude", "longitude"))
print(global_mean)

# Compute a time series mean over space
swh_mean = var.mean(dim=("latitude", "longitude")).compute()
print(swh_mean)
```

::::::::::::::::::::::::::::::::::::::: challenge

## Exercise 2 - Understand your NetCDF dataset

Using the NetCDF dataset(s) provided on the server:

1. Open a single file with `open_dataset` and print:
   - Dimensions and coordinates.
   - Data variables.
   - Encoding information (especially chunking).
2. Then, open multiple files with `open_mfdataset` and verify:
   - Consistency of variables and coordinates.
   - Combined time dimension length.

What are the main variables and dimensions you care about, and what is the existing chunking layout (if any)?

::::::::::::::: solution

Open a single NetCDF file:

```python
import xarray as xr
ds_nc = xr.open_dataset("data/daily_swh/swh_2025-01-01.nc")
print(ds_nc.dims)
print(ds_nc.data_vars)
print(ds_nc.coords)
print(ds_nc.encoding)
```

Open multiple NetCDF files:

```python
ds_nc = xr.open_mfdataset(
    "data/daily_swh/swh_*.nc",
    combine="by_coords",
)
print(ds_nc.dims)
print(ds_nc.data_vars)
print(ds_nc.coords)
```

Now you should have a clear picture of the input data: variables, dimensions, coordinates, and any existing chunking or compression. This understanding is essential before deciding a Zarr chunk layout or doing any conversion.

:::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::::: challenge

## Exercise 3 - Propose Zarr chunk sizes

Based on your NetCDF dataset and expected workloads:

1. Propose a chunk dictionary, e.g. `{"time": 12, "lat": 100, "lon": 100}` or similar.
2. Estimate (roughly) the size of each chunk in bytes/MB (using variable sizes and dimension lengths).
3. Explain how your chunk choices support your intended analyses.

You do not need exact numbers; focus on reasoning.

::::::::::::::: solution

```python
# Example chunk proposal
chunking = {"time": 12, "latitude": 100, "longitude": 100}

chunk_size_bytes = (
    ds_chunked["swh"].dtype.itemsize  # bytes per element
    * chunking["time"]
    * chunking["latitude"]
    * chunking["longitude"]
)
chunk_size_MB = chunk_size_bytes / (1024 ** 2)
print(f"Estimated chunk size: {chunk_size_MB:.2f} MB")
```

After calculating the estimated chunk size, you can reason about how this chunking supports your analysis:

```python
ds_chunked = ds_nc.chunk(chunking)
```

You'd propose reasonable chunk sizes that reflect your workloads and aim for manageable chunk sizes.
You will practice linking chunk shapes to analysis patterns, preparing for actual conversion.

:::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::::::::::::::::


::::::::::::::::::::::::::::::::::::::: challenge

## Exercise 4 - Convert your NetCDF dataset to Zarr

Using the NetCDF dataset on the server, after opening and chunking it, convert it to Zarr and write to a local directory (e.g. `data/converted.zarr`). Please make sure you are using dask for parallel conversion.

Questions:

- How long does the conversion take?
- Does the size of the Zarr store seem reasonable compared to the original NetCDF files?
- Were there any variables or attributes you needed to drop or adjust?

::::::::::::::: solution


```python
import xarray as xr
from dask.distributed import Client

client = Client(n_workers=2, threads_per_worker=2)

ds_chunked.to_zarr("data/converted.zarr", mode="w", consolidated=True)
```

Now you have a local Zarr store from the NetCDF dataset and begin to see how conversion time and output size depend on chunking and compression.
They will also encounter practical issues (e.g. problematic variables, encoding quirks) that often arise in real conversions.

:::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::::: challenge

## Exercise 5 - Upload and reopen from object storage

Now, repeat the last step, but instead of writing to local disk, write the Zarr store directly to an object store (e.g. JASMIN) using fsspec. And then, reopen the dataset from the object store and verify that it is usable, confirming that dimensions, variables, and attributes match expectations.

Please use the credentials provided by the instructor to access the object store. In this lesson, also use the JASMIN dask cluster to run the conversion and upload, as it has better network access to the object store.

::::::::::::::: solution

Create the cluster:

```python
import dask_gateway

# Create a connection to dask-gateway.
gw = dask_gateway.Gateway("https://dask-gateway.jasmin.ac.uk", auth="jupyterhub")

options = gw.cluster_options()
options.worker_cores = 2
options.scheduler_cores = 1
options.account = "workshop"
options.worker_setup='source /apps/jasmin/jaspy/miniforge_envs/jaspy3.11/mf3-23.11.0-0/bin/activate /work/scratch-nopw2/colinsau/esces-env'
clusters = gw.list_clusters()
if not clusters:
    cluster = gw.new_cluster(options, shutdown_on_close=False)
else:
   cluster = gw.connect(clusters[0].name)
```

Now that we have a running cluster, we can get a client object from the cluster:

```python
client = cluster.get_client()
cluster.adapt(minimum=1, maximum=4)
```

Now you can use the client to run the conversion and upload to object storage:

```python
import xarray as xr
import fsspec

storage_options = {
    "key": "your-access-key",
    "secret": "your-secret-key",
    "client_kwargs": {
        "endpoint_url": "https://atlantis-vis-o.s3-ext.jc.rl.ac.uk",
    },
}

store_url = "s3://my-bucket/converted.zarr"
mapper = fsspec.get_mapper(store_url, storage_options=storage_options)

ds_chunked.to_zarr(mapper, mode="w", consolidated=True)
ds_zarr = xr.open_zarr(mapper, consolidated=True)
print(ds_zarr)
```

Once the Zarr store is in object storage, you can open it almost as easily as a local dataset, using the right filesystem adapter.
It can confirm that the conversion and upload have preserved structure and metadata, and that the dataset is ready for analysis.

:::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::::::::::::::::


::::::::::::::::::::::::::::::::::::::: challenge

## Exercise 6 - Check consistency and compute a mean

Using both:

- The original NetCDF files dataset (`ds_nc`).
- The converted Zarr dataset (`ds_zarr` from object storage).

1. Compute the same statistic in both cases (e.g. global mean temperature at a given time):

2. Compare the results for equality (or near-equality within floating point tolerance).

::::::::::::::: solution


```python
client = Client(n_workers=2, threads_per_worker=2)

# NetCDF
ds_nc = xr.open_mfdataset("data/daily_swh/swh_*.nc", combine="by_coords")
var_nc = ds_nc["swh"]
# slice to a single time step for comparison
var_nc = var_nc.isel(time=0)

mean_nc = var_nc.mean(dim=("latitude", "longitude"))

# Zarr
# Assuming ds_zarr opened as above
var_zarr = ds_zarr["swh"]
var_zarr = var_zarr.isel(time=0)

mean_zarr = var_zarr.mean(dim=("latitude", "longitude"))
mean_zarr = mean_zarr.compute()

print("NetCDF mean:", float(mean_nc.values))
print("Zarr mean:", float(mean_zarr.values))
```

When done carefully, converting NetCDF to Zarr and back preserves scientific values. The mean values should match within floating point tolerance, confirming that the conversion workflow is valid.

:::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::::::::::::::::

:::::::::::::::::::::::::::::::::::::::::: keypoints

- "Converting NetCDF to Zarr enables cloud-native, chunked, and parallel-friendly access to large scientific datasets."
- "Effective conversion requires understanding input data, choosing chunk sizes based on workloads, and using xarray's `to_zarr` with appropriate encoding and compression."
- "Dask can parallelise the conversion process, making it feasible to handle large collections of NetCDF files."
- "Uploading Zarr stores to object storage allows distributed teams and tools to access the same datasets efficiently."
- "Checking basic statistics (e.g. mean values) in NetCDF and Zarr versions helps verify that conversions preserve scientific content."

::::::::::::::::::::::::::::::::::::::::::::::::::
