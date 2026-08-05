---
title: Setup
---


## Setup

Before starting this workshop, **complete the preparation guide available in the [Setup Instructions](https://noc-oi.github.io/prep-work-cloud-native-geoscience-course/#setup)**. This guide contains everything you need to get ready for the course, including instructions for both in-person and remote participants.

The setup guide covers:

- Installing the required software.
- Downloading the example datasets used throughout the workshop.
- Verifying that your environment is correctly configured.

In addition to the technical setup, the guide includes prerequisite material to help you prepare for this course. It consists of three introductory episodes:

1. Shell basics for navigating files, editing text, and using help tools.
2. Introductory `xarray` workflows for opening, slicing, plotting, and processing NetCDF data.
3. Git foundations for tracking changes, branching, and merging.

Completing the setup guide ensures that all participants begin the workshop with a working environment and a shared foundation in the command line, `xarray`, and Git workflows.

## About the example data

This workshop uses example ocean and atmospheric datasets in **NetCDF**, **GRIB**, and **Zarr** formats to demonstrate different approaches for storing and accessing scientific data.

The datasets are derived from widely used public products, including:

* **ERA5 hourly data on single levels** from the [Copernicus Climate Data Store](https://cds.climate.copernicus.eu/datasets/reanalysis-era5-single-levels) (ECMWF). ERA5 is the fifth-generation global atmospheric reanalysis, providing hourly estimates of atmospheric, land-surface, and ocean-wave variables from 1940 to the present. It combines numerical weather prediction models with observations through data assimilation to produce a consistent, high-quality record for weather and climate applications. For this workshop, we use NetCDF, GRIB, and Zarr files containing the significant wave height (`swh`) and sea surface temperature (`sst`) variables.

* **GLORYS12V1 Ocean Reanalysis** from the [Copernicus Marine Service](https://data.marine.copernicus.eu/product/GLOBAL_MULTIYEAR_PHY_001_030/description). GLORYS12V1 is a global, high-resolution ocean reanalysis based on the NEMO ocean model, combining numerical simulations with satellite and in situ observations through data assimilation to provide a consistent representation of the ocean state. For this workshop, we use NetCDF and Zarr files containing sea surface height (`ssh`), potential temperature (`thetao`), salinity (`so`), and the zonal and meridional current velocity (`uo` and `vo`) variables.

These datasets provide realistic examples for exploring cloud-native geoscience workflows throughout the workshop.

### What to expect

- The data are large enough that you should not expect to open the full dataset into memory.
- You will often work on one variable, one time slice, or one small spatial region.
- This is intentional: the workshop is about working efficiently with large datasets.
