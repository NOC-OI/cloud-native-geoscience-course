---
title: Learner Profiles
---

This course is designed for researchers, data scientists, and engineers working with
oceanography, climate, meteorology, and related environmental data at gigabyte to
multi-terabyte scale.

Participants are expected to be comfortable with Python fundamentals and command-line
usage, but do not need prior experience with cloud-native formats or cloud platforms.

## Profile 1: Domain Scientists Modernizing Existing NetCDF Workflows

- Background:
	- Ocean, climate, or meteorology researchers already working with NetCDF/GRIB and
		xarray-based analysis.
- Motivation:
	- Need faster access and analysis for larger datasets, especially for remote and
		collaborative workflows.
	- Want to move from local file-heavy workflows to object storage and cloud-native
		data access.
- Challenges:
	- Mapping established practices to newer concepts like chunking, multiscale
		structures, and object-store layout.
	- Understanding trade-offs between conversion cost, storage cost, and analysis
		performance.
- Learning objectives:
	- Understand Zarr and cloud-native data patterns.
	- Choose chunking and layout strategies for real access patterns.
	- Build reproducible conversion and processing pipelines.

## Profile 2: Data Engineers and Platform Staff Supporting Environmental Programs

- Background:
	- Engineers and technical staff responsible for data pipelines, storage systems,
		interoperability, and service reliability.
- Motivation:
	- Need robust architectures that support discovery, versioning, processing, and
		visualisation across institutions and teams.
	- Want practical patterns for integrating Zarr, STAC, Icechunk, and object storage
		in production-oriented workflows.
- Challenges:
	- Designing systems that balance performance, maintainability, governance, and
		cost.
	- Aligning metadata and catalog standards with operational requirements.
- Learning objectives:
	- Design end-to-end cloud-native architecture for scientific data.
	- Apply best practices for metadata, cataloging, and versioning.
	- Support scalable data access for analysis and web visualization.

## Profile 3: Python Users New to Large-Scale Scientific Data Infrastructure

- Background:
	- Participants with solid Python skills (NumPy/xarray/pandas) and scientific data
		experience, but limited exposure to cloud-native geoscience systems.
- Motivation:
	- Want to become independent in converting, organizing, and analyzing large
		multidimensional datasets.
	- Need practical knowledge that can be applied immediately in research and
		operations.
- Challenges:
	- Connecting many concepts at once: object storage, chunking, parallel processing,
		virtual datasets, catalog metadata, and map-based visualization.
	- Avoiding trial-and-error approaches that produce slow or brittle pipelines.
- Learning objectives:
	- Build confidence working with modern geoscience data stacks.
	- Select tools based on clear trade-offs, not trends.
	- Leave with a practical blueprint for their next project.

## Shared Outcomes Across Profiles

By the end of the course, learners should be able to:

- Explain core cloud-native concepts and vocabulary for n-dimensional data.
- Convert and organize data for scalable remote access.
- Use parallel processing and chunk-aware workflows for performance.
- Apply interoperability patterns with STAC and related standards.
- Create maintainable architectures for analysis and visualization of environmental
	data.
