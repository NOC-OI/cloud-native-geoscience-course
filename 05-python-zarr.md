---
title: Python Tools for Working with Zarr
teaching: 20
exercises: 10
---

:::::::::::::::::::::::::::::::::::::::::: objectives

- Import the core Python libraries for working with Zarr data.
- Inspect a Zarr store directly with the `zarr` library.
- Open a Zarr dataset with xarray and explore variables, dimensions, and coordinates.
- Understand lazy loading and why it matters for large datasets.
- Use a few basic xarray operations on environmental Zarr data.

::::::::::::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::::::::: questions

- What Python packages are commonly used to work with Zarr data?
- How can I inspect a Zarr store directly with the `zarr` library?
- How does xarray represent Zarr datasets as labelled N-dimensional data?
- When does the data actually get loaded into memory?
- What basic xarray operations are useful for oceanography, climate, and meteorology?

::::::::::::::::::::::::::::::::::::::::::::::::::

## Python ecosystem for Zarr

Zarr has a rich ecosystem of Python tools:

- [**zarr**](https://zarr.readthedocs.io/en/stable/) - the core Python implementation of Zarr's chunked N‑dimensional arrays and groups.
- [**xarray**](https://xarray.dev/en/stable/) - a labelled N‑dimensional array library that can open Zarr datasets and present them as `Dataset` objects with named dimensions and coordinates.
- [**fsspec**](https://filesystem-spec.readthedocs.io/en/latest/) / [**s3fs**](https://s3fs.readthedocs.io/en/latest/) / [**gcsfs**](https://gcsfs.readthedocs.io/en/latest/) / [**obstore**](https://obstore.readthedocs.io/en/latest/) - filesystem adapters to access Zarr stores on local disk, S3, Google Cloud Storage, and other backends.
- **Supporting tools** - Virtualizarr Icechunk and others build on Zarr and xarray for specialised workflows (rechunking, transactional storage, cloud‑ready archives).

In this lesson we focus on the essentials:

- How to use `zarr` directly for low-level inspection.
- How to use xarray for most day‑to‑day analysis with environmental data in Zarr format.

## Opening with `zarr` library

The `zarr` package provides low-level access to Zarr stores, allowing you to:

- Open groups and arrays.
- Inspect shapes, chunk shapes, data types.
- Read and write data into arrays.
- View and edit attributes (metadata).

On the data directory, open the Zarr store `data/ocean_temperature.zarr`

```python
import zarr

root = zarr.open_group("data/ocean_temperature.zarr", mode="r")
print(root)
print(list(root.arrays()))
print(list(root.groups()))
print("Group attributes:", root.attrs)
```

You can then inspect one array directly:

```python
temp = root["temperature"]
print(temp)
print("Shape:", temp.shape)
print("Chunks:", temp.chunks)
print("Data type:", temp.dtype)
print("Array attributes:", temp.attrs)
```

A Zarr array tells you how the data are organised on disk or in object storage. Its `shape` tells you the full size of the array, `chunks` tells you how it is split up, and `attrs` can store useful metadata such as units or variable names.

::::::::::::::::::::::::::::::::::::::: challenge

## Exercise 1 - Explore the store

Using the example Zarr store:

1. List all arrays in the root group.
2. Choose one data variable and print its `shape`, `chunks`, and `dtype`.
3. Inspect the array attributes and identify at least three useful metadata fields.

Questions:

- What dimensions do you think the `shape` represents?
- How does `chunks` divide the data?
- Which attributes look useful for analysis?

::::::::::::::: solution

Each array has a fixed shape and chunk layout, and that metadata provides context such as units, long names, and standard names:

```python
import zarr

root = zarr.open_group("data/ocean_temperature.zarr", mode="r")
print(list(root.arrays()))
temp = root["temperature"]
print(temp)
print("Shape:", temp.shape)
print("Chunks:", temp.chunks)
print("Data type:", temp.dtype)
print("Array attributes:", temp.attrs)
```

:::::::::::::::

::::::::::::::::::::::::::::::::::::::::::::::::::

:::::::::::::::::::::::::::::::::::::::::: callout

## Lazy loading and memory

Opening a Zarr store usually reads metadata first, not the full data values. This is often called **lazy loading**: the dataset structure is available immediately, but the chunk data are only read when you ask for values.

With `zarr`, properties such as `shape`, `chunks`, `dtype`, and `.attrs` come from metadata. The actual data are read when you index or slice the array.

```python
first_slice = temp[0, :, :]
```

At that point, Zarr reads only the chunks needed for that selection. It does not need to load the full array unless you request the full array.

The same idea applies with xarray: opening the dataset is cheap, and data are loaded only when you select, compute, plot, or ask for values.

::::::::::::::::::::::::::::::::::::::::::::::::::


## Opening with xarray

While `zarr` is ideal for low-level inspection and manipulation, xarray is usually the main tool for analysing multidimensional environmental data. It provides:

- **Datasets** - collections of data variables (arrays) with shared dimensions and coordinates.
- **Labelled dimensions** - names like `time`, `lat`, `lon`, `depth` instead of numeric indices.
- **Coordinates** - explicit arrays for latitude, longitude, time, etc.
- **High-level operations** - selection (`.sel`), reduction (`.mean`, `.sum`), resampling, plotting, etc.

xarray can open NetCDF, GRIB (via backends), and Zarr data. Because of this, it makes Zarr datasets accessible in a familiar, analysis-ready format. And it also makes it really easy to update workflows to use cloud-native Zarr stores without changing the analysis code.

For Zarr it uses `xr.open_zarr`, which understands the Zarr metadata and conventions. Basic usage:

```python
import xarray as xr

ds = xr.open_zarr("data/ocean_temperature.zarr")
print(ds)           # Overview of variables and coordinates
print(ds.dims)      # Dimensions
print(ds.data_vars) # Data variables
print(ds.coords)    # Coordinate variables
```

Important points:

- `ds` is an xarray `Dataset`.
- `ds.data_vars` lists data variables (e.g. `temperature`, `salinity`).
- `ds.coords` typically includes `time`, `lat`, `lon`, and any other coordinate variables.
- Metadata from Zarr arrays and attributes is mapped into xarray's structure so that CF conventions and other patterns can be used.


The result is an xarray `Dataset`. Each variable is an xarray `DataArray`, and the metadata from the Zarr store is carried into the xarray structure.

```python
temp = ds["temperature"]
print(temp)
print("Temperature dims:", temp.dims)
print("Temperature attrs:", temp.attrs)
```

::::::::::::::::::::::::::::::::::::::: challenge

## Exercise 2 - Open with xarray

Using the same dataset:

1. Open it with `xr.open_zarr`.
2. Print `ds.dims`, `ds.data_vars`, and `ds.coords`.
3. Inspect one variable and print its dimensions and attributes.
4. Note whether the dataset opens quickly even if it is large.

Questions:

- Which variables are data variables versus coordinates?
- Which dimensions are present?
- What does xarray tell you about the dataset without loading all the values?

::::::::::::::: solution

Xarray organises the dataset into named dimensions and separates data variables from coordinate variables, which makes the structure easier to understand. Opening the dataset is fast because xarray reads metadata first and loads data only when needed.

:::::::::::::::

::::::::::::::::::::::::::::::::::::::::::::::::::

### Basic xarray operations

For this lesson, keep xarray to a small set of core operations:

- Select data with `.sel()` using coordinate values.
- Use `.isel()` when you want numeric positions.
- Compute a mean over one or more dimensions.
- Plot a slice quickly with `.plot()`.

```python
surface = ds["temperature"].sel(depth=0)
time_slice = ds["temperature"].sel(time=slice("2020-01-01", "2020-01-31"))
global_mean = ds["temperature"].mean(dim=("lat", "lon"))
point_ts = ds["temperature"].sel(lat=0.0, lon=0.0, method="nearest")
```

In the operations above, the data is not loaded into memory until you request values or plot the results. For example, the following code will trigger data loading and plotting:

```python
surface.sel(time="2020-01-01").plot()
point_ts.plot()
```

These operations are especially useful for environmental data, because they keep the code readable and let you work with named coordinates rather than raw array positions.


### What comes into memory?

A useful rule of thumb is:

- **Opening** a Zarr store loads metadata.
- **Inspecting** `shape`, `chunks`, `dims`, and attributes mostly uses metadata.
- **Selecting**, **computing**, **plotting**, or calling `.values` reads data chunks into memory.

For example, this mainly inspects metadata:

```python
print(ds["temperature"])
print(ds["temperature"].dims)
print(ds["temperature"].attrs)
```

But this requests actual data values:

```python
ds["temperature"].isel(time=0).values
```

This distinction is one of the main reasons Zarr works well for large environmental datasets: you can explore the dataset structure first, and only load the parts you need for analysis.

::::::::::::::::::::::::::::::::::::::: challenge

## Exercise 3 - First analysis on a Zarr dataset

Using your example Zarr dataset and xarray:

1. Inspect the available variables and pick one that makes sense for a simple analysis (e.g. sea surface temperature).
2. Compute a spatial mean over latitude and longitude and inspect the resulting time series.
3. Select a small spatial region (e.g. a box around a particular ocean basin) and compute a mean for that region.

Example:

```python
import xarray as xr

ds = xr.open_zarr("data/ocean_temperature.zarr")

# Adjust variable and dimension names to match your dataset
temp = ds["temperature"]

# Global mean time series
global_ts = temp.mean(dim=("lat", "lon"))
print(global_ts)

# Regional mean (example box)
regional_ts = temp.sel(lat=slice(-30, 30), lon=slice(-60, 0)).mean(dim=("lat", "lon"))
print(regional_ts)

# Additional plotting if desired
regional_ts.plot()
```

Questions:

- How does xarray's interface make these operations readable compared to direct array indexing?
- What would change if the dataset were too large to fit in memory (you'll explore this in later lessons with Dask and cloud‑native formats)?
- How comfortable do you feel with xarray's basic usage now?

::::::::::::::: solution

xarray's labelled dimensions and high‑level methods (`.mean`, `.sel`, `.plot`) make common operations much more expressive and less error‑prone than raw array indexing.
Opening Zarr datasets with xarray offers a consistent interface across storage formats, laying the groundwork for later lessons on performance, chunking, and cloud‑native workflows.

:::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::::: challenge

## Exercise 3 - Read only a small part

Use xarray to open the Zarr dataset, then select only a small part of it. Select temperature data for the first month of 2020, for a small region in the Brazilian Southest Coast (min lat: -30, max lat: -20, min lon: -50, max lon: -40). Compute the mean of that subset.

Questions:

- Did you need to load the full dataset?
- What parts were actually read into memory?
- Why is this useful for large datasets?


::::::::::::::: solution

```python
import xarray as xr

ds = xr.open_zarr("data/ocean_temperature.zarr")
temp = ds["temperature"]

subset = temp.sel(time="2020-01-01", lat=slice(-30, -20), lon=slice(-50, -40))
print(subset)
print(subset.mean())
```

We can open a huge dataset and still process only a small portion of it. This is one of the main advantages of Zarr and xarray for large environmental data.

:::::::::::::::

::::::::::::::::::::::::::::::::::::::::::::::::::


:::::::::::::::::::::::::::::::::::::::::: keypoints

- "The zarr Python package provides low-level access to Zarr stores, including groups, arrays, and attributes."
- "xarray is the main high-level tool for working with environmental Zarr datasets as labelled N-dimensional data."
- "Opening Zarr with xarray (`xr.open_zarr`) gives you variables, dimensions, and coordinates in a familiar `Dataset` structure."
- "Data values are loaded only when they are selected or used in a computation."
- "Basic xarray operations such as selection, reduction, and plotting work the same on Zarr data as on NetCDF data."

::::::::::::::::::::::::::::::::::::::::::::::::::
