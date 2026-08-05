---
title: Virtual Zarr with Virtualizarr
teaching: 25
exercises: 25
---

:::::::::::::::::::::::::::::::::::::::::: objectives

- "Explain what a virtual Zarr store is."
- "Create a virtual Zarr dataset from local NetCDF files."
- "Open the virtual dataset with xarray and inspect its structure."
- "Compare virtual access with direct NetCDF access."

::::::::::::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::::::::: questions

- "Why might we prefer a virtual Zarr store over fully converting data to Zarr?"
- "How do we create a virtual dataset from NetCDF files already on the server?"
- "How does a virtual Zarr store look when opened with xarray?"
- "What do we gain by virtualising instead of copying all the data?"

::::::::::::::::::::::::::::::::::::::::::::::::::

## Why virtual Zarr stores?

Converting all archival data to Zarr is sometimes impractical. Archives may contain many NetCDF files, and fully rewriting them into physical Zarr can take time and storage space. Some sites also need to keep the original NetCDF files for legacy workflows or operational reasons.

A virtual Zarr store gives you a way to access those files in a Zarr-like way without copying all the data. Instead of writing new chunk files, [Virtualizarr](https://virtualizarr.readthedocs.io/en/stable/) builds a virtual dataset that points back to the original NetCDF data.

![](fig/fake_zarr.png){alt="An illustration NetCDF files trying to looking like a Zarr Store."}


### When to use Virtualizarr instead of physical conversion

Common scenarios:

- You have a large archive of NetCDF/GRIB files and want fast, chunked access with xarray and Dask, but cannot or do not want to rewrite everything.
- You must keep the original files unchanged for regulatory, legacy, or operational reasons.
- Your storage system struggles with huge numbers of small files that physical Zarr stores generate.
- You want to prototype cloud‑optimised data access before committing to a full migration.

However, physical Zarr conversion is still useful when you expect heavy use and want maximum performance, or when you want to restructure the data into a more efficient layout.

Virtualizarr provides a middle path: a lightweight abstraction layer that lets archival data behave like Zarr without copying or restructuring underlying files.

Virtualizarr lets you create a virtual dataset from one file or many files. From the user's point of view, the result looks like a normal xarray dataset, but the data are still backed by the original NetCDF files rather than copied into new Zarr chunks.

In this lesson, we will use the NetCDF files already available on the server. That means the workflow stays small and practical: open the files, virtualise them, and inspect the result.


::::::::::::::::::::::::::::::::::::::: challenge

## Exercise 1 - Decide when to virtualise

Discuss these situations in pairs:

1. A small archive used by one team.
2. A large archive that must keep the original NetCDF files.
3. A dataset that will be queried repeatedly by many users.

For each case, decide whether you would:

- Keep the files as NetCDF.
- Create a virtual Zarr store.
- Fully convert to physical Zarr.

::::::::::::::: solution

A reasonable set of answers is:

1. **A small archive used by one team**
   Keep the files as NetCDF if the workflow is already working well and there is no strong need for browser-style or chunked access. If the team wants easier interactive analysis without copying data, a virtual Zarr store is a good middle ground.

2. **A large archive that must keep the original NetCDF files**
   Create a virtual Zarr store. This gives you Zarr-like access without rewriting the archive, while preserving the original files for legacy or operational use.

3. **A dataset that will be queried repeatedly by many users**
   Fully convert to physical Zarr if performance is the top priority and you expect heavy repeated use. Physical Zarr is often the best choice when you want the fastest access and the storage cost of conversion is acceptable.

:::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::::::::::::::::

## Virtualizarr workflow

The VirtualiZarr workflow is similar to a physical conversion workflow, but with one key difference: instead of copying data into new Zarr chunks, VirtualiZarr creates a virtual Zarr dataset whose chunks reference the original NetCDF files. This avoids duplicating the underlying data while providing a Zarr-compatible interface.

In this lesson, we will use a subset of the [ERA5 Reanalysis dataset](https://cds.climate.copernicus.eu/datasets/reanalysis-era5-single-levels) containing significant wave height data. The dataset is provided as multiple NetCDF files, one per day, in the `data/daily_swh/` directory.

All examples in this lesson use files stored on the server for simplicity. However, the same workflow also works with NetCDF files stored in an object store.

### Step 1: inspect the NetCDF files

First, inspect the NetCDF files with xarray so you know what you are working with.

```python
import xarray as xr

base_path = "/gws/ssde/j25b/atlantis_vis/cloud-native-geoscience-course/" # or "" if you have the data in your current working directory

ds_nc = xr.open_mfdataset(f"{base_path}data/daily_swh/swh_*.nc", combine="by_coords")

print(ds_nc)
print(ds_nc.dims)
print(ds_nc.data_vars)
print(ds_nc.coords)
```

### Step 2: create a virtual dataset

Now create a virtual dataset from the NetCDF files. Virtualizarr will read the files and build a lightweight virtual store that points back to the original data.

We first import the required modules and set up access to the local files. In this example, we use [`obstore`](https://developmentseed.org/obstore/latest/) to provide a consistent interface to the local filesystem. It lets Virtualizarr work with the NetCDF files in the same way it would work with other storage backends.

```python
import glob
from obstore.store import LocalStore
from obspec_utils.registry import ObjectStoreRegistry
from virtualizarr import open_virtual_mfdataset
from virtualizarr.parsers import HDFParser
```

Next, collect the NetCDF file paths. Here we use a local directory containing daily significant wave height data.

```python
data_dir = f"{base_path}data/daily_swh/"

file_paths = sorted(glob.glob(f"{data_dir}*.nc"))
urls = [f"file://{path}" for path in file_paths]
```

Then create a local store and register it. The `LocalStore` gives Virtualizarr access to the files on disk, and the `ObjectStoreRegistry` tells it where those files live.

```python
store = LocalStore(data_dir)
registry = ObjectStoreRegistry({f"file://{data_dir}": store})
```

Because these are HDF5-based NetCDF files, we also need the HDF parser.

```python
parser = HDFParser()
```

Now open the virtual dataset. This returns an xarray-like dataset backed by virtual references rather than copied Zarr chunks.

```python
vds = open_virtual_mfdataset(
    urls=urls,
    parser=parser,
    registry=registry,
    combine="nested",
    concat_dim="time", # the files are combined along the time dimension
)

print(vds)
```

The important point is that this does **not** copy the data into new Zarr files. It creates a virtual representation that can be opened and analysed like a Zarr-style dataset.

:::::::::::::::::::::::::::::::::::::::::: spoiler

## Obstore

[`obstore`](https://developmentseed.org/obstore/latest/) gives a simple Python interface to cloud object storage that is built for speed and works well with safer, more predictable data access patterns. It can be a good choice when you want fast S3/GCS/Azure reads and writes without the extra complexity of working directly with provider-specific clients or filesystem-style wrappers.

Take a look at this [talk](https://www.youtube.com/watch?v=z_CkATeUe-Y) if you want to explore it further. It is not the focus of this lesson, but it is interesting if you want to work with cloud object storage more efficiently.

::::::::::::::::::::::::::::::::::::::::::

### Step 3: create a local Icechunk repository

One advantage of virtual datasets is that they can be saved to an Icechunk repository. This allows you to create a snapshot of the virtual dataset that can be reopened later.

First you need to create a local Icechunk repository. This is done by specifying a path where the repository will be stored and creating a configuration that allows Icechunk to resolve the virtual chunks.

```python
import icechunk as ic

# This is the path where the Icechunk repository will be created
storage = ic.local_filesystem_storage("data/daily_swh_icechunk/")

# Because we are using a virtual dataset, we need to tell Icechunk where to find the original NetCDF files when reopening the snapshot. This is done by creating a virtual chunk container that points to the local filesystem store.
config = ic.RepositoryConfig.default()
config.set_virtual_chunk_container(
    ic.VirtualChunkContainer(f"file://{data_dir}", ic.local_filesystem_store(data_dir))
)

repo = ic.Repository.create(storage, config)
repo.save_config()
```

Now you have a local Icechunk repository that can store virtual datasets. The next step is to write the virtual dataset into the repository.

### Step 4: write the virtual dataset into Icechunk

Now write the virtual dataset into the repository and commit it as a snapshot.

```python
session = repo.writable_session("main")

vds.vz.to_icechunk(session.store)

snapshot_id = session.commit("Initial virtual reference set for daily_swh NetCDF files")
print("Committed snapshot:", snapshot_id)
```

### Step 5: reopen the snapshot

With the virtual dataset saved in Icechunk, you can now reopen it:

```python
# Because we are using a virtual dataset, we need to authorize access to the original NetCDF files when reopening the snapshot.
repo = ic.Repository.open(
    storage,
    authorize_virtual_chunk_access={
        f"file://{data_dir}": ic.credentials.LocalFileSystemAccess,
    },
)
session = repo.writable_session("main")
ds_main = xr.open_zarr(session.store, consolidated=False)
print(ds_main)
```

You can see that the dataset structure is the same as before, but now it is backed by the Icechunk repository. You can work with the virtual dataset like a Zarr store, while the original NetCDF files remain unchanged.

### Compare a simple analysis

Let's compare a simple analysis on the original NetCDF files and on the virtual dataset. We will compute the mean significant wave height over latitude and longitude for both datasets and measure the time taken for each operation.

```python
import time

pattern = f"{base_path}data/daily_swh/swh_*.nc"

t0 = time.time()
ds_nc = xr.open_mfdataset(pattern, combine="by_coords")
result_nc = ds_nc["swh"].isel(time=0).mean(dim=("latitude", "longitude")).compute()
print(f"Result for original dataset: {result_nc.values}")
t1 = time.time()
print("NetCDF time:", t1 - t0)

t0 = time.time()
repo = ic.Repository.open(
    storage,
    authorize_virtual_chunk_access={
        f"file://{data_dir}": ic.credentials.LocalFileSystemAccess,
    },
)
session = repo.writable_session("main")
ds_virtual = xr.open_zarr(session.store, consolidated=False)
result_virtual = ds_virtual["swh"].isel(time=0).mean(dim=("latitude", "longitude")).compute()
print(f"Result for virtual zarr: {result_virtual.values}")
t1 = time.time()
print("Virtual time:", t1 - t0)
```

The exact timings will depend on the dataset and environment, but we expect the virtual dataset to be faster than opening all the NetCDF files directly. The goal is to show that virtualisation can give you a Zarr-like + Icechunk access pattern without fully converting the data.

::::::::::::::::::::::::::::::::::::::: challenge

## Exercise 2 - Open a virtual dataset from local NetCDF files

Using the NetCDF files related to daily significant wave height from ERA5 (`data/daily_swh/*.nc`):

1. Identify a pattern covering several files with compatible dimensions.
2. Use `virtualizarr.open_virtual_mfdataset` to create a virtual dataset.
3. Print:
   - Dimensions (`.dims`).
   - Data variables (`.data_vars`).
   - Coordinates (`.coords`).

::::::::::::::: solution

```python
import glob
from obstore.store import LocalStore
from obspec_utils.registry import ObjectStoreRegistry
from virtualizarr import open_virtual_mfdataset
from virtualizarr.parsers import HDFParser

data_dir = f"{base_path}data/daily_swh/"

file_paths = sorted(glob.glob(f"{data_dir}*.nc"))
urls = [f"file://{p}" for p in file_paths]

store = LocalStore(data_dir)
registry = ObjectStoreRegistry({f"file://{data_dir}": store})

parser = HDFParser()
vds = open_virtual_mfdataset(
    urls=urls,
    parser=parser,
    registry=registry,
    combine="nested",
    concat_dim="time"
)

print(vds.dims)
print(vds.data_vars)
print(vds.coords)
```

:::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::::: challenge

## Exercise 3 - Add Icechunk and create the first snapshot

Now use the virtual dataset inside Icechunk.

1. With the virtual dataset, write it into the Icechunk repository.
2. Create the first snapshot.
3. Reopen `main` and confirm that the dataset structure is the same as before.

::::::::::::::: solution

A typical workflow is:

1. Write the virtual dataset into Icechunk.

```python
import icechunk as ic

storage = ic.local_filesystem_storage("data/daily_swh_icechunk/")

config = ic.RepositoryConfig.default()
config.set_virtual_chunk_container(
    ic.VirtualChunkContainer(
        f"file://{data_dir}",
        ic.local_filesystem_store(data_dir),
    )
)

repo = ic.Repository.create(storage, config)
repo.save_config()
```

2. Commit the virtual dataset as a snapshot.

```python
session = repo.writable_session("main")
vds.vz.to_icechunk(session.store)
snapshot_id = session.commit("Initial virtual reference set for daily_swh NetCDF files")
print("Committed snapshot:", snapshot_id)
```

3. Reopen `main` and confirm that the dataset structure is the same as before. Remember to first authorize access to the original NetCDF files:

```python
repo = ic.Repository.open(
    storage,
    authorize_virtual_chunk_access={
        f"file://{data_dir}": ic.credentials.LocalFileSystemAccess,
    },
)
session = repo.writable_session("main")
ds_main = xr.open_zarr(session.store, consolidated=False)
print(ds_main)
```

:::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::::::::::::::::

:::::::::::::::::::::::::::::::::::::::::: caution

## Watch API churn

VirtualiZarr and Icechunk are both moving fast. The API for opening/authorizing has changed release to release. Pin versions in your environment and note the versions in any shared code, since a working script today may need updating on `pip install --upgrade` in a few months.

::::::::::::::::::::::::::::::::::::::::::::::::::

### Why this is useful

Virtualizarr keeps the original files in place, but lets you work with a chunked, cloud-friendly view of the data. You can choose chunks that match your access pattern, such as time-based chunks for time series or spatial chunks for maps.

This becomes even more useful when combined with Icechunk. Virtualizarr can create the virtual references, and Icechunk can store them as versioned snapshots. That means you can build a cloud-optimised virtual dataset without copying the original NetCDF files, while still gaining version history, reproducibility, and safe incremental updates.

The same idea works not only for local NetCDF files, but also for NetCDF files that are already stored in object storage such as S3, GCS, Azure, or compatible systems. In that case, the virtual references point directly to the remote objects, so you can create a Zarr-like view of an existing archive in place, without moving or rewriting the source data. Please take a look at this [talk](https://www.youtube.com/watch?v=QBkZQ53vE6o) with more information about virtualisation and Icechunk directly from cloud object storage.

A small caveat is that performance still depends on the layout of the original files. If the underlying files are poorly aligned with your chosen chunks, access may be less efficient because the virtual store still has to read from those files underneath. In practice, the best chunk strategy is usually the one that matches your analysis needs while staying reasonably aligned with the source layout.

The table below compares different approaches to accessing large NetCDF archives[^nicholas].

<div style="background-color: #f9f9f9; border-left: 5px solid #ccc; display:flex; justify-content:center; align-items:center; margin-bottom: 1em;">
  <table>
    <thead>
      <tr style="color: black;">
        <th>Format</th>
        <th>NetCDF4</th>
        <th>“Native” Zarr</th>
        <th>Kerchunk</th>
        <th>Icechunk</th>
      </tr>
    </thead>
    <tbody>
      <tr>
        <th style="color: black;"># of URLs</th>
        <td style="color: red;">500,000</td>
        <td style="color: green;">1</td>
        <td style="color: green;">1</td>
        <td style="color: green;">1</td>
      </tr>
      <tr>
        <th style="color: black;">Time to open</th>
        <td style="color: red;">~1 year</td>
        <td style="color: green;">&lt; 1 sec</td>
        <td style="color: green;">&lt; 1 sec</td>
        <td style="color: green;">&lt; 1 sec</td>
      </tr>
      <tr>
        <th style="color: black;">Storage increase</th>
        <td style="color: green;">0%</td>
        <td style="color: red;">100%</td>
        <td style="color: red;">0.0004%</td>
        <td style="color: red;">0.0004%</td>
      </tr>
      <tr>
        <th style="color: black;">Convert using Xarray?</th>
        <td style="color: black;">N/a</td>
        <td style="color: green;">Yes</td>
        <td style="color: red;">No</td>
        <td style="color: green;">Yes</td>
      </tr>
      <tr>
        <th style="color: black;">Version-controlled?</th>
        <td style="color: red;">No</td>
        <td style="color: red;">No</td>
        <td style="color: red;">No</td>
        <td style="color: green;">Yes</td>
      </tr>
      <tr>
        <th style="color: black;">Update-safe?</th>
        <td style="color: red;">No</td>
        <td style="color: red;">No</td>
        <td style="color: red;">No</td>
        <td style="color: green;">Yes</td>
      </tr>
    </tbody>
  </table>
</div>

:::::::::::::::::::::::::::::::::::::::::: keypoints

- "Virtualizarr creates virtual Zarr datasets without copying the data."
- "It is useful when full conversion is impractical or unnecessary."
- "You can create a virtual dataset directly from NetCDF files already on the server."
- "xarray can open the virtual dataset and analyse it like a familiar dataset."
- "Virtualisation is a useful bridge between NetCDF archives and Zarr-style workflows."

::::::::::::::::::::::::::::::::::::::::::

[^nicholas]: Nicholas, T. VirtualiZarr + Icechunk: Build Cloud-Optimised DataCube of Archival Files. Cloud-Native Geospatial Conference, 2025. Available at [https://www.youtube.com/watch?v=QBkZQ53vE6o](https://www.youtube.com/watch?v=QBkZQ53vE6o), 2025.
