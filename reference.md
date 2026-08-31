---
layout: Glossary
---

## Glossary

[**asset (STAC)**]{#asset-stac}
:   A data link inside a STAC Item that points to the real file or store
    (for example a Zarr store, NetCDF file, or image), usually with media type and role metadata.

[**block storage**]{#block-storage}
:   Storage exposed as fixed-size blocks, typically used for virtual disks and databases.
    It is different from object storage and file storage.

[**branch (Icechunk)**]{#branch-icechunk}
:   A named line of dataset history in an Icechunk repository, such as `main` or `dev`.

[**bucket**]{#bucket}
:   A top-level container in object storage. Objects are addressed by key inside a bucket.

[**BUFR**]{#bufr}
:   Binary Universal Form for the Representation of meteorological data.
    A WMO exchange format often used for observational data.

[**catalog (STAC Catalog)**]{#catalog-stac}
:   A STAC object used as an entry point and hierarchy for linking Collections and Items.

[**CF conventions**]{#cf-conventions}
:   Climate and Forecast metadata conventions that define how Earth science variables,
    coordinates, and attributes are described in self-describing array data.

[**chunk**]{#chunk}
:   A smaller piece of a larger array. Chunking allows selective reads and parallel access.

[**chunk shape**]{#chunk-shape}
:   The size of each chunk along each dimension (for example time, lat, lon).
    Chunk shape should match typical access patterns.

[**cloud-native format**]{#cloud-native-format}
:   A format or layout designed for efficient remote access over object storage and web APIs,
    so tools can read metadata quickly and fetch only required subsets.

[**cloud-optimized**]{#cloud-optimized}
:   A data layout tuned for cloud access patterns, usually minimizing metadata reads
    and supporting range/subset access.

[**collection (STAC Collection)**]{#collection-stac}
:   A STAC object that groups related Items and includes shared metadata
    such as extent, license, keywords, and providers.

[**cluster**]{#cluster}
:   A group of connected computers that run workloads together.

[**consolidated metadata (Zarr)**]{#consolidated-metadata}
:   Zarr metadata combined into a single index file to speed up dataset opening,
    especially for stores with many arrays.

[**controlled vocabulary**]{#controlled-vocabulary}
:   A curated list of standard terms and identifiers used to avoid ambiguity in metadata.

[**coordinate**]{#coordinate}
:   A named axis value used to locate data (for example time, latitude, longitude, depth).

[**COG (Cloud-Optimized GeoTIFF)**]{#cog}
:   A GeoTIFF layout designed for cloud access using range requests and internal overviews.

[**core (CPU core)**]{#core}
:   One compute unit inside a CPU that can execute tasks.

[**CPU**]{#cpu}
:   Central Processing Unit, the hardware that executes instructions.

[**Conda / Mamba**]{#conda-mamba}
:   Package and environment managers used to create and maintain Python environments
    for reproducible analysis.

[**data variable**]{#data-variable}
:   The main measured or modeled variable in a dataset,
    such as temperature, salinity, or wave height.

[**Dask**]{#dask}
:   A Python parallel computing framework used for chunked arrays,
    task scheduling, and distributed processing.

[**discovery metadata**]{#discovery-metadata}
:   Higher-level metadata records (often JSON/XML) used by catalogs and portals
    to describe, find, and assess datasets.

[**environment (Python environment)**]{#environment}
:   A controlled set of Python packages and versions used to run the lesson software.

[**fsspec**]{#fsspec}
:   A Python filesystem abstraction used to access local and remote storage
    with a common interface.

[**GeoParquet**]{#geoparquet}
:   A geospatial convention on top of Parquet for efficient cloud-friendly vector data.

[**GeoZarr**]{#geozarr}
:   A set of conventions for representing geospatial datasets in Zarr,
    including CRS, spatial transforms, and multiscale metadata.

[**GRIB**]{#grib}
:   GRIdded Binary, a compact WMO format widely used for weather and forecast grids.

[**HDF5**]{#hdf5}
:   Hierarchical Data Format v5, a self-describing format for large multidimensional data.

[**HPC**]{#hpc}
:   High-performance computing: shared systems designed for large-scale computation and data processing.

[**Icechunk**]{#icechunk}
:   A versioning layer for Zarr that adds repositories, transactions, snapshots, branches, and tags.

[**item (STAC Item)**]{#item-stac}
:   A STAC object representing one spatiotemporal asset with geometry,
    time metadata, and links to data assets.

[**JASMIN**]{#jasmin}
:   A UK data analysis facility for environmental science,
    used in the course context as an example shared compute and storage platform.

[**Kerchunk**]{#kerchunk}
:   A reference-based approach that maps existing NetCDF/HDF/GRIB data
    into a Zarr-like access model without rewriting full data payloads.

[**kernel (Jupyter kernel)**]{#kernel}
:   The runtime process that executes notebook code in a selected Python environment.

[**lazy loading**]{#lazy-loading}
:   Data is not fully loaded at open time; chunks are loaded only when needed for operations.

[**metadata**]{#metadata}
:   Data about data, such as variables, units, coordinates, provenance, and processing details.

[**multiscale pyramid**]{#multiscale-pyramid}
:   A hierarchy of lower-resolution dataset levels used for fast visualization at different zoom levels.

[**N-dimensional data**]{#n-dimensional-data}
:   Data arrays with multiple dimensions (for example time, latitude, longitude, depth, member).

[**NetCDF**]{#netcdf}
:   Network Common Data Form, a common self-describing format for scientific array data.

[**NERC Vocabulary Server (NVS)**]{#nvs}
:   A service publishing controlled vocabularies and stable identifiers used in marine metadata.

[**node (cluster node)**]{#node}
:   One machine inside a compute cluster.

[**object**]{#object}
:   A unit of data in object storage containing payload plus metadata,
    addressed by bucket and key.

[**object key**]{#object-key}
:   The identifier for an object within a bucket.
    Key prefixes are often used to emulate folder-like organization.

[**object storage**]{#object-storage}
:   A storage model that keeps data as objects in buckets and is accessed through APIs such as S3.

[**open_mfdataset**]{#open-mfdataset}
:   An xarray function for opening and combining multiple files into one logical dataset.

[**parallel processing**]{#parallel-processing}
:   Running multiple parts of work at the same time across threads, processes, or workers.

[**Parquet**]{#parquet}
:   A columnar table format designed for efficient analytics and cloud storage workflows.

[**process**]{#process}
:   A running instance of a program (for example one Python interpreter process).

[**RAM**]{#ram}
:   Short-term memory used by running programs.
    Large datasets often require careful chunking to avoid exhausting RAM.

[**rechunking**]{#rechunking}
:   Changing chunk layout to better match downstream workloads or storage constraints.

[**reference file**]{#reference-file}
:   A JSON or Parquet mapping file that describes how virtual data access points
    to underlying source files.

[**repository (Icechunk repository)**]{#repository-icechunk}
:   The top-level Icechunk object that stores dataset history and versioned state.

[**S3**]{#s3}
:   Amazon Simple Storage Service API pattern, widely used directly or through compatible systems.

[**s3fs**]{#s3fs}
:   A Python filesystem interface for S3-compatible object storage, built on fsspec.

[**self-describing data**]{#self-describing-data}
:   Data files that contain enough structural metadata for tools to interpret content
    without external manuals.

[**snapshot (Icechunk)**]{#snapshot-icechunk}
:   An immutable dataset version created when an Icechunk transaction is committed.

[**STAC**]{#stac}
:   SpatioTemporal Asset Catalog specifications for describing and organizing geospatial assets.

[**SSH**]{#ssh}
:   Secure Shell protocol used to connect to remote systems over a network.

[**storage class**]{#storage-class}
:   A tier in object storage with different cost and performance profiles,
    such as frequent-access, infrequent-access, or archive.

[**tag (Icechunk)**]{#tag-icechunk}
:   A human-readable label pointing to a specific Icechunk snapshot.

[**thread**]{#thread}
:   A lightweight execution unit inside a process that shares memory with other threads.

[**Topozarr**]{#topozarr}
:   A Python tool used in the course to build multiscale Zarr pyramids for visualization.

[**transaction (Icechunk)**]{#transaction-icechunk}
:   An atomic write session where all changes succeed together or none are applied.

[**VirtualiZarr / virtual Zarr store**]{#virtualizarr}
:   A virtual layer that presents existing files as a Zarr-like dataset without fully copying data.

[**xarray**]{#xarray}
:   A Python library for labeled N-dimensional arrays and datasets,
    commonly used with NetCDF, Zarr, and Dask.

[**Zarr**]{#zarr}
:   A chunked format and data model for N-dimensional arrays,
    well suited for object storage and parallel cloud workflows.
