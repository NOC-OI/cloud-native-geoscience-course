---
title: Architectures and Best Practices
teaching: 10
exercises: 25
---

:::::::::::::::::::::::::::::::::::::::::: objectives

- "Reflect on how the different tools and formats fit together into end‑to‑end architectures."
- "Identify trade‑offs between simplicity, performance, cost, and maintainability."
- "Discuss best practices for designing cloud‑native workflows with Zarr, STAC, Icechunk, Virtualizarr, and GeoZarr."

::::::::::::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::::::::: questions

- "What does a good cloud‑native architecture for geospatial and climate data look like?"
- "How do we decide when to use physical Zarr, virtual Zarr, STAC, Icechunk, GeoZarr, and multiscale pyramids?"
- "What practices help keep data systems robust, reproducible, and future‑proof?"

::::::::::::::::::::::::::::::::::::::::::::::::::


## Why architecture and practice matter

Over the course of the lessons, you have seen many building blocks: formats (NetCDF, Zarr), metadata (CF Conventions), object storage, virtualisation (Virtualizarr), versioning (Icechunk), cataloguing (STAC), and visualisation (GeoZarr, multiscale pyramids). Each solves a specific problem, but they only deliver real value when combined into coherent architectures.

A good architecture makes it easy to discover data, access it efficiently, update it safely, and visualise it interactively, while still being understandable and maintainable by your team. Best practices are the habits and patterns that keep such architectures healthy over time: clear metadata, sensible chunking, documented decisions, and automation for repetitive tasks.


::::::::::::::::::::::::::::::::::::::: challenge

## Exercise 1 - Sketch an end‑to‑end architecture

Individually or in small groups:

1. Sketch an architecture for a hypothetical project (e.g. a global reanalysis, an ensemble forecast system, or a satellite product).
2. Decide:
   - Where raw data lives (formats, storage).
   - How to handle metadata.
   - How and when you convert to Zarr or virtual Zarr.
   - How you apply versioning (Icechunk) and cataloguing (STAC).
   - How users discover, analyse, and visualise data (e.g. GeoZarr, browser tools).
3. Capture your choices and the reasoning behind them in a few bullet points.

Share and discuss:

- Where did you prioritise simplicity?
- Where did you invest in more complex tooling for performance or governance?

::::::::::::::: solution

:::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::::::::::::::::


::::::::::::::::::::::::::::::::::::::: challenge

## Exercise 2 - Draft your personal best‑practice checklist

Take 5-10 minutes to draft a short checklist for future projects, for example:

- "Always define clear metadata and vocabularies for new datasets."
- "Plan chunking and multiscale layout based on expected access patterns, not just defaults."
- "Use STAC for discovery and Zarr for analysis. Keep them in sync."
- "Adopt versioning (Icechunk or similar) for any dataset that is updated regularly."
- "Prefer virtualisation (Virtualizarr, kerchunk) when full conversion is impractical."
- "Document architectural decisions and revisit them periodically."

Compare your checklist with a neighbour's and refine it into something you might actually use in your own work.

::::::::::::::: solution

:::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::::::::::::::::

:::::::::::::::::::::::::::::::::::::::::: keypoints

- "Cloud-native workflows are strongest when formats, metadata, storage, and operations are designed together as one architecture."
- "There is no single best stack: choices between physical Zarr, virtual Zarr, STAC, Icechunk, and GeoZarr depend on workload, governance, and resources."
- "Practical trade-offs among simplicity, performance, cost, and maintainability should be documented and revisited over time."
- "Chunking, metadata quality, and discoverability are foundational decisions that strongly affect usability and long-term sustainability."
- "A short, explicit best-practice checklist helps teams apply course concepts consistently in real projects."

::::::::::::::::::::::::::::::::::::::::::::::::::
