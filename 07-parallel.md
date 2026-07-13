---
title: Parallel Processing for Zarr
teaching: 25
exercises: 15
---

:::::::::::::::::::::::::::::::::::::::::: objectives

- Explain why parallel processing matters for large Zarr datasets.
- Use Dask with xarray and Zarr to parallelise common array computations.
- Understand how chunking, lazy loading, and task graphs work together.
- Recognise a few other Python parallelism patterns that are sometimes used with Zarr workflows.

::::::::::::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::::::::: questions

- Why do we need parallel processing for large Zarr datasets?
- How does Dask parallelise xarray and Zarr computations?
- How do chunking and lazy loading support parallel work?
- What other parallelism tools do Python users sometimes combine with Zarr?

::::::::::::::::::::::::::::::::::::::::::::::::::


## Why parallel processing?

Large environmental datasets can be too slow or too large to process on a single core or in a single process. This becomes especially important with Zarr because the format is designed to be read and written chunk-by-chunk, which makes it a natural fit for parallel work.

Processing such data on a single core or in a single process can be:

- **Slow** - long runtimes for even simple operations.
- **Memory-limited** - datasets or intermediate results may not fit into RAM.
- **I/O-bound** - reading and writing data dominates computation time, especially from disks or across networks.

Parallel processing helps when your workflow needs to:

- read many chunks from a Zarr store,
- compute statistics over large spatial or temporal regions,
- or run the same operation over many independent files, variables, or ensemble members.

In practice, parallelism works best when both the compute layer and the data layout are aligned. Chunked storage such as Zarr lets multiple workers read different chunks independently, while Dask can schedule the work across those chunks.


## Dask: distributed processing for Python

Dask is a Python library for parallel and distributed computing. It offers two main sets of features:

- **High-level collections** - Dask arrays, Dask dataframes, Dask bags, and Xarray integration, which behave like NumPy, pandas, etc. but execute lazily and in parallel.
- **Low-level task scheduling** - delayed functions and futures to run arbitrary Python functions in parallel, coordinated by a scheduler.

Key ideas:

- **Task graph**: Dask builds a graph of tasks and dependencies when you write code, but does not execute immediately.
- **Lazy evaluation**: computations are only executed when you call `.compute()` or similar methods.
- **Cluster**: a set of workers managed by a scheduler; can be local (one machine) or remote (HPC cluster, Kubernetes, etc.).

Basic setup on a local machine:

```python
from dask.distributed import Client, progress

client = Client(processes=False, threads_per_worker=4,
                n_workers=1, memory_limit="2GB")
client  # Show cluster information
```

![Dask Setup](fig/dask_setup.png){alt="Dask setup showing a local cluster with one worker and four threads."}

### Using the Dask dashboard

In the information about the Dask cluster is a link to a Dashboard webpage. From the Dashboard we can monitor our Dask cluster and see how busy it is, view a graph of task dependencies, memory usage and the status of the Dask workers. This can be really useful when checking if our Dask cluster is behaving correctly and working out how optimially our code is making use of Dask's parallelism. Note that it is not possible (or at least not without significant additional complexity) to access the Dask dashboard when running on the JASMIN notebook service.


![Dask dashboard](fig/dask_dashboard.png){alt="Dask dashboard showing task progress and worker status."}


### Dask arrays and lazy computation

Dask arrays are chunked arrays that mimic NumPy's API but execute lazily and in parallel.

Example:

```python
import dask.array as da

# Create a 10000x10000 array of random numbers, chunked 1000x1000
x = da.random.random((10000, 10000), chunks=(1000, 1000))
print(x)

# Simple operation: add ones
y = da.ones((10000, 10000), chunks=(1000, 1000))
z = x + y
print(z)  # Still a Dask array, not yet computed

# Trigger computation
result = z.mean().compute()
print("Mean value:", result)
```
Important points:

- Creating `x` and `y` defines a Dask array with specified chunking, but no data is computed until needed.
- Operations build a task graph; `.compute()` runs the tasks.
- Chunking lets Dask process different blocks in parallel and only load chunks into memory when required.

::::::::::::::::::::::::::::::::::::::: challenge

## Exercise 1 - Compare Dask and NumPy performance

Compare the performance of the following code using Numpy and Dask functions. Use the %%time magic in the cells to find out the execution time. Ensure you only time the core computation and not the Dask cluster setup or library imports, this means you'll have to write this code into multiple cells. Dask version (note you'll need to do the Dask client setup first):

```python
# Dask version
import dask.array as da
x = da.random.random((20000,20000), chunks=(1000,1000))
x_mean = x.mean()
x_mean.compute()
```

Numpy version:

```python
import numpy as np
npx = np.random.random((20000,20000))
npx_mean = npx.mean()
```

Which went faster overall? Why do you think you got the result you did? Try making the dataset a little larger, going much beyond 25000x25000 might use too much memory. Try running the top command in a terminal while your notebook is running, look at the CPU % when running the Numpy and Dask versions and compare them. Try changing the number of Dask threads and see what effect this has on the CPU %.

::::::::::::::::::::::::::::::::::::::::::::::::::

:::::::::::::::::::::::::::::::::::::::::: callout

## Troubleshooting Dask

Sometimes Dask can jam up and stop executing tasks. If this happens try the following:

- Shutdown the client and restart it.
- Shutdown the kernel of your notebook and rerun the notebook.

::::::::::::::::::::::::::::::::::::::::::::::::::

:::::::::::::::::::::::::::::::::::::::::: callout

## Some pitfalls to watch out for

Combining lazy loading with parallel processing can be tricky. It is difficult to know when the data are actually being read from disk or downloaded from a remote source. This can lead to unexpected memory usage or performance issues.

So, when do the download and processing actually happen? Things that are usually obvious but catch us all off guard

::::::::::::::::::::::::::::::::::::::::::::::::::


### Dask with Zarr via xarray

The most common pattern in this workshop is to open a Zarr store with xarray and ask xarray to use Dask-backed chunks. Xarray then keeps the data lazy until you compute something.

First, create the client:
```python
import xarray as xr
from dask.distributed import Client

client = Client(n_workers=2, threads_per_worker=2, memory_limit="1GB")
client
```

Then open the Zarr dataset with chunking:

```python
ds = xr.open_zarr("data/example.zarr", chunks={})
print(ds)

temp = ds["temperature"]
print(temp)
```

The `chunks` argument tells xarray to load the data into Dask arrays with a chosen chunk structure. If the chunking matches your workload, the resulting operations can run efficiently across multiple workers.

### A simple parallel computation

```python
corrected = temp * 1.1 - 1.0
global_mean = corrected.mean(dim=("lat", "lon"))
result = global_mean.compute()
print(result)
```

This workflow is important because the data stay lazy until `.compute()` is called. At that point Dask schedules the work across the chunks.

::::::::::::::::::::::::::::::::::::::: challenge

## Exercise 1 - Dask and xarray together

Using your Zarr dataset:

1. Open the dataset with `xr.open_zarr`, specifying chunk sizes.
2. Compute a spatial mean over latitude and longitude.
3. Select a regional subset and compute its mean.
4. Call `.compute()` and compare the result with and without Dask-backed chunks.

Questions:

- Which chunk sizes seem most suitable for your workload?
- How many chunks do you think the operation touches?
- Does the computation feel faster or easier to reason about with Dask?

::::::::::::::: solution

xarray plus Dask gives them a lazy, chunk-aware workflow. The important link is that chunking from the storage lesson now becomes the basis for parallel computation.

:::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::::::::::::::::

## Other parallelism patterns

Dask is the main tool in this workshop, but Python users also use a few other parallelism patterns with Zarr workflows.

### `dask.delayed`

`dask.delayed` is useful when you want to parallelise a custom Python function instead of an array operation. It lets you build a task graph from normal Python code.

```python
import dask

@dask.delayed
def apply_correction(x):
    return x * 1.01 + 0.1
```

This is helpful when a workflow is not naturally expressed as a NumPy or xarray operation.

### Futures

Dask futures are useful when you want tasks to start immediately and manage results asynchronously. They are a good fit for workflows that submit many independent jobs, such as processing many Zarr groups or many files.

```python
from dask.distributed import Client

client = Client()

future = client.submit(sum, )
print(future.result())
```

:::::::::::::::::::::::::::::::::::::::::: callout

## Other ways to parallelise Zarr workflows

Dask is the main parallel tool in this lesson, but Python has other ways to run work in parallel. The best choice depends on whether the task is CPU-bound, I/O-bound, or already chunked.

### Threads

Thread-based parallelism can help when the work spends time waiting on I/O, which happens when reading chunks from a Zarr store. Threads are often useful for data access and downloading, but they are less flexible than Dask for organising a whole analysis workflow.

### Multiprocessing

Python's built-in multiprocessing is useful for independent tasks that do not need to share much state. It can work well for embarrassingly parallel jobs, but it is usually less convenient than Dask for Zarr and xarray because it does not understand chunks, task graphs, or labelled arrays automatically.

### Cubed

Cubed is a library for chunked, out-of-core array processing. It is designed for large array computations and can be a good fit when you want to process Zarr-like data in a memory-aware way, especially in cloud-oriented workflows.

::::::::::::::::::::::::::::::::::::::::::::::::::

## Parallelism-friendly Zarr storage

Zarr is a good fit for parallel processing because it stores data in chunks, and each chunk can be read independently. That means multiple workers can read different chunks at the same time without needing a single giant file.

This is especially useful when the data live in object storage or another backend that can serve chunk-sized requests efficiently. In other words, Zarr gives Dask something it can parallelise cleanly.

::::::::::::::::::::::::::::::::::::::: challenge

## Exercise 2 - A small parallel workflow

Using the same dataset:

1. Open the dataset with chunk sizes chosen for your workload.
2. Run one spatial statistic and one time-based statistic.
3. Repeat the same operation with a different chunk size.
4. Compare which version creates a more efficient task structure.

Questions:

- Which dimensions matter most for your workflow?
- Did the chunk layout help or hurt parallel execution?
- How might the answer change if the data were much larger?

::::::::::::::: solution

The main lesson is that parallelism works best when the data are already organised in a way that matches the common workload. Zarr provides the storage layout, and Dask provides the parallel execution.

:::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::::::::::::::::

:::::::::::::::::::::::::::::::::::::::::: callout

## Visualising the Task Graph

To see how Dask is scheduling work, you can visualise the task graph. This can help you understand how many tasks are being created and how they depend on each other.

```python
import dask
import xarray as xr

ds = xr.open_zarr("data/example.zarr", chunks={})
temp = ds["temperature"]
corrected = temp * 1.1 - 1.0
global_mean = corrected.mean(dim=("lat", "lon"))
dask.visualize(global_mean, filename='task_graph.png')
```

This will create a PNG file showing the task graph for the computation. You can also use `global_mean.visualize()` to see the graph directly in a Jupyter notebook.

::::::::::::::::::::::::::::::::::::::::::::::::::

:::::::::::::::::::::::::::::::::::::::::: keypoints

- "Dask is the main parallel tool used here."
- "Zarr and Dask work well together because Zarr stores data in independent chunks."
- "Xarray can open Zarr data lazily and hand chunked work to Dask."
- "Other Python parallelism tools exist, but Dask is the most natural fit for chunked environmental data."

::::::::::::::::::::::::::::::::::::::::::::::::::
