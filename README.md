# Cloud-Native Architectures and Modern Data Formats for Geoscience

Short title: Cloud-Native Geoscience Data Workflows

## About This Course

This repository contains lesson material for a three-day course on modern, cloud-native workflows for oceanography, climate, and meteorology data at multi-terabyte scale.

The course covers the end-to-end pipeline for n-dimensional scientific data, including:

- cloud-native data concepts and vocabulary
- Zarr data model, metadata, and chunking
- Python tooling for reading and analyzing Zarr data
- object storage architecture and organization patterns
- conversion workflows from traditional formats (for example NetCDF) to Zarr
- parallel processing strategies
- versioning with Icechunk
- interoperability with VirtualiZarr and STAC
- visualization with multiscale Zarr and GeoZarr
- architectural best practices for reproducible, scalable workflows

Rendered lesson website:
<https://noc-oi.github.io/cloud-native-geoscience-course/>

Reference setup and prep-work website:
<https://noc-oi.github.io/prep-work-cloud-native-geoscience-course/>


Reference workshop page (timing source):
<https://noc-oi.github.io/2026-brazil/en/>

## Workshop Timing and Materials

The table below maps the workshop schedule to this repository's lesson materials:


## Episodes

| # | Episode | Teaching (min) | Exercises (min) | Total (min) | Question(s) |
|--:|:---------|:--------------:|:---------------:|:-----------:|:------------|
| 1 | [Data Formats, Metadata, and Vocabulary][episode01] | 30 | 15 | 45 | What are the main data formats used to store ocean and atmosphere data?<br>What is metadata, and how does it help other people find and understand my data?<br>What is a controlled vocabulary, and why is it better than free text?<br>Which international standards should I be aware of when publishing environmental data?<br>How do these standards connect to modern cloud-native formats like Zarr? |
| 2 | [Challenges of N-Dimensional Data][episode02] | 25 | 20 | 45 | How do NetCDF, GRIB, and HDF5 organise large n-dimensional arrays?<br>Why have typical datasets in meteorology and oceanography grown so much in size?<br>What happens when we open these files in standard tools like xarray?<br>Why is it important to share data in a way that supports partial access rather than whole-file downloads?<br>How do chunking and parallel processing affect performance, scalability, and memory use? |
| 3 | [Cloud-Native Formats][episode03] | 15 | 10 | 25 | What makes a format cloud-native?<br>Why are traditional file-based formats not always a good fit for cloud storage?<br>Which cloud-native or cloud-optimized approaches are relevant for Earth system data?<br>How do cloud-native formats change the way we work with environmental data? |
| 4 | [Zarr Data Model and Chunked Storage][episode04] | 30 | 20 | 50 | What is Zarr, and how does its data model differ from formats like NetCDF or HDF5?<br>How does Zarr store metadata about arrays and groups?<br>What is chunked storage in Zarr, and why is it useful for large multidimensional datasets?<br>How is Zarr changing the way ocean, climate, and meteorological data are stored and accessed? |
| 5 | [Python for Zarr][episode05] | 25 | 25 | 50 | What Python packages are commonly used to work with Zarr data?<br>How can I inspect a Zarr store directly with the `zarr` library?<br>How does xarray represent Zarr datasets as labelled N-dimensional data?<br>When does the data actually get loaded into memory?<br>What basic xarray operations are useful for oceanography, climate, and meteorology? |
| 6 | [Choosing Chunks at Scale][episode06] | 35 | 30 | 65 | What is a chunk, and why does its shape matter for performance?<br>How should I choose chunk sizes for different types of analysis?<br>What are the trade-offs between large and small chunks?<br>How can I rechunk a Zarr dataset and save it for future use? |
| 7 | [Parallel Processing with Zarr][episode07] | 35 | 30 | 65 | Why do we need parallel processing for large Zarr datasets?<br>How does Dask parallelise xarray and Zarr computations?<br>How do chunking and lazy loading support parallel work?<br>What other parallelism tools do Python users sometimes combine with Zarr? |
| 8 | [Reading Real-World Zarr Datasets in Python][episode08] | 30 | 35 | 65 | Which publicly available Zarr datasets can I use for experimentation and learning?<br>How do I open Zarr datasets from cloud object storage (Google Cloud, AWS S3) with Python?<br>How do irregular grids, ragged arrays, and ensembles appear in Zarr + xarray?<br>How do chunks and storage layout influence how I analyse these datasets? |
| 9 | [Object Storage and Cloud Data Organization][episode09] | 30 | 25 | 55 | What is an object store, and how is it different from storing data on a server filesystem?<br>Why is object storage a good fit for large-scale data sharing and cloud-native science?<br>How does object storage support secure, concurrent, and parallel access?<br>How can I use commercial cloud object stores and self-hosted solutions like MinIO? |
| 10 | [Converting Traditional Formats to Zarr][episode10] | 30 | 40 | 70 | Why convert NetCDF data to Zarr, and what changes in the way we access and process data?<br>How do we choose chunk sizes for Zarr when starting from NetCDF files?<br>How can we use Dask and xarray to convert and write data to Zarr efficiently?<br>How do we test that the converted data is usable and correct (e.g. computing mean values)? |
| 11 | [Case Studies][episode11] | 45 | 0 | 45 | How are different teams and organisations actually converting and serving data in practice?<br>What problems do they face, and which solutions have worked well (or badly)?<br>What lessons can we take from their architectures and workflows for our own projects? |
| 12 | [Versioning Data with Icechunk][episode12] | 35 | 35 | 70 | What problems arise when we use plain Zarr for shared, evolving datasets?<br>How does Icechunk add safety, consistency, and reproducibility on top of Zarr?<br>How can we use transactions to update data atomically and avoid partial writes?<br>How can we reference and replay specific versions of data for reproducible analysis? |
| 13 | [Virtual Zarr with Virtualizarr][episode13] | 25 | 25 | 50 | Why might we prefer a virtual Zarr store over fully converting data to Zarr?<br>How do we create a virtual dataset from NetCDF files already on the server?<br>How does a virtual Zarr store look when opened with xarray?<br>What do we gain by virtualising instead of copying all the data? |
| 14 | [Organizing Cloud Zarr Data with STAC][episode14] | 30 | 25 | 55 | How does STAC help us organise and discover Zarr data cubes in the cloud?<br>What is the difference between a STAC Catalog, Collection, and Item?<br>How can we programmatically build STAC metadata for our Zarr datasets?<br>Why might we store STAC metadata in a database instead of a set of JSON files? |
| 15 | [Visualizing Multiscale Zarr and GeoZarr][episode15] | 25 | 30 | 55 | How does chunking affect interactive visualisation of Zarr data in the cloud?<br>What is a multiscale pyramid, and why does it come from Cloud-Optimized GeoTIFF ideas?<br>What does GeoZarr add on top of Zarr for geospatial visualisation?<br>How can we use Topozarr to create multiscale Zarr from our example dataset and explore it in a browser? |
| 16 | [Architecture and Best Practices][episode16] | 10 | 25 | 35 | What does a good cloud-native architecture for geospatial and climate data look like?<br>How do we decide when to use physical Zarr, virtual Zarr, STAC, Icechunk, GeoZarr, and multiscale pyramids?<br>What practices help keep data systems robust, reproducible, and future-proof? |

## Contributing

Contributions are welcome.

Please read [CONTRIBUTING.md](CONTRIBUTING.md) for contribution workflow and lesson development guidance.

## Maintainer
The lesson maintainer is [Tobias Ferreira](https://github.com/soutobias).

## License

Instructional material is available under the
[Creative Commons Attribution-NonCommercial 4.0 International][cc-by-nc-human] license.

Except where otherwise noted, example programs and software are available under the
[MIT License][mit-license].

For full details, see [LICENSE.md](LICENSE.md).

## Citation

To cite this lesson, see [CITATION.cff](CITATION.cff).

[episode01]: episodes/01-intro.md
[episode02]: episodes/02-ndata-challenges.md
[episode03]: episodes/03-cloudnative-formats.md
[episode04]: episodes/04-zarr.md
[episode05]: episodes/05-python-zarr.md
[episode06]: episodes/06-chunks.md
[episode07]: episodes/07-parallel.md
[episode08]: episodes/08-handson-zarr.md
[episode09]: episodes/09-object_store.md
[episode10]: episodes/10-conversion-workflow.md
[episode11]: episodes/11-case-studies.md
[episode12]: episodes/12-icechunk.md
[episode13]: episodes/13-virtualizarr.md
[episode14]: episodes/14-stac.md
[episode15]: episodes/15-visualisation.md
[episode16]: episodes/16-best-practices.md

[mit-license]: https://opensource.org/licenses/mit-license.html
[cc-by-nc-human]: https://creativecommons.org/licenses/by-nc/4.0/
