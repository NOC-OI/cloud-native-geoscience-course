# Cloud-Native Architectures and Modern Data Formats for Geoscience

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

Reference workshop page (timing source):
<https://noc-oi.github.io/2026-brazil/en/>

## Workshop Timing and Materials

The table below maps the workshop schedule to this repository's lesson materials, using the current episode names.

### Day 1 - Foundations and Formats

| Time (UTC-3) | Session | Material |
|:--|:--|:--|
| 09:15 | Course introduction, goals, and setup | [01 - Introduction to Data Formats, Metadata, and Vocabulary][episode01] |
| 09:45 | Introduction to data formats, metadata, and vocabulary | [01 - Introduction to Data Formats, Metadata, and Vocabulary][episode01] |
| 10:20 | Challenges of n-dimensional data | [02 - Challenges of N-dimensional Data][episode02] |
| 11:00 | What are cloud-native formats? | [03 - What Are Cloud-Native Formats?][episode03] |
| 11:20 | Zarr: data model, metadata, and chunked storage | [04 - Zarr - Data Model, Metadata, and Chunked Storage][episode04] |
| 13:30 | Python tools for working with Zarr | [05 - Python Tools for Working with Zarr][episode05] |
| 15:00 | Hands-on: Zarr datasets | [08 - Hands-On with Open Zarr Datasets Using Python][episode08] |
| 15:50 | Hands-on: Zarr datasets (continued) | [08 - Hands-On with Open Zarr Datasets Using Python][episode08] |
| 16:20 | How to choose chunks for analysis and processing at scale | [06 - How to Choose Chunks for Analysis and Processing at Scale][episode06] |

### Day 2 - Storage, Conversion, and Processing

| Time (UTC-3) | Session | Material |
|:--|:--|:--|
| 09:10 | Object storage: concepts, remote access, and organizing data in the cloud | [09 - Object Storage - Concepts, Remote Access, and Data Organization in the Cloud][episode09] |
| 10:00 | Scientific data layouts in object storage | [09 - Object Storage - Concepts, Remote Access, and Data Organization in the Cloud][episode09] |
| 11:00 | Converting traditional formats to Zarr | [10 - Conversion Workflow of Traditional Formats to Zarr][episode10] |
| 13:30 | Parallel processing | [07 - Parallel Processing for Zarr][episode07] |
| 15:10 | Hands-on: conversion and processing pipeline | [10 - Conversion Workflow of Traditional Formats to Zarr][episode10] |
| 15:50 | Hands-on: conversion and processing pipeline (continued) | [07 - Parallel Processing for Zarr][episode07] + [10 - Conversion Workflow of Traditional Formats to Zarr][episode10] |

### Day 3 - Versioning, Interoperability, and Visualization

| Time (UTC-3) | Session | Material |
|:--|:--|:--|
| 09:10 | Data versioning with Icechunk | [12 - Data Versioning with Icechunk][episode12] |
| 11:00 | Talk session: how are other institutions using Zarr? | [11 - Invited Case Studies][episode11] |
| 11:45 | Creating virtual Zarr stores with Virtualizarr | [13 - Creating Virtual Zarr Stores with Virtualizarr][episode13] |
| 12:20 | Organizing Zarr data in the cloud with STAC | [14 - Zarr Data Organization in the Cloud with STAC][episode14] |
| 13:30 | Organizing Zarr data in the cloud with STAC (continued) | [14 - Zarr Data Organization in the Cloud with STAC][episode14] |
| 13:50 | Visualization: multiscale Zarr and GeoZarr | [15 - Visualization - Multiscale Zarr and GeoZarr][episode15] |
| 14:50 | Hands-on: visualizing data in the cloud | [15 - Visualization - Multiscale Zarr and GeoZarr][episode15] |
| 15:50 | Hands-on: visualizing data in the cloud (continued) | [15 - Visualization - Multiscale Zarr and GeoZarr][episode15] |
| 16:30 | Discussion: architectures and best practices | [16 - Architectures and Best Practices][episode16] |

## Episode Index

| # | Episode |
|--:|:--|
| 1 | [Introduction to Data Formats, Metadata, and Vocabulary][episode01] |
| 2 | [Challenges of N-dimensional Data][episode02] |
| 3 | [What Are Cloud-Native Formats?][episode03] |
| 4 | [Zarr - Data Model, Metadata, and Chunked Storage][episode04] |
| 5 | [Python Tools for Working with Zarr][episode05] |
| 6 | [How to Choose Chunks for Analysis and Processing at Scale][episode06] |
| 7 | [Parallel Processing for Zarr][episode07] |
| 8 | [Hands-On with Open Zarr Datasets Using Python][episode08] |
| 9 | [Object Storage - Concepts, Remote Access, and Data Organization in the Cloud][episode09] |
| 10 | [Conversion Workflow of Traditional Formats to Zarr][episode10] |
| 11 | [Invited Case Studies][episode11] |
| 12 | [Data Versioning with Icechunk][episode12] |
| 13 | [Creating Virtual Zarr Stores with Virtualizarr][episode13] |
| 14 | [Zarr Data Organization in the Cloud with STAC][episode14] |
| 15 | [Visualization - Multiscale Zarr and GeoZarr][episode15] |
| 16 | [Architectures and Best Practices][episode16] |

## Contributing

Contributions are welcome.

Please read [CONTRIBUTING.md](CONTRIBUTING.md) for contribution workflow and lesson development guidance.

## Maintainer
The lesson maintainer is [Tobias Ferreira](https://github.com/soutobias).

## License

Instructional material is available under the
[Creative Commons Attribution 4.0 International][cc-by-human] license.

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
[cc-by-human]: https://creativecommons.org/licenses/by/4.0/
