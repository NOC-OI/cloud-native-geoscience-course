---
title: Setup
---

## Introduction

This workshop introduces practical tools and workflows for working with large environmental datasets in a cloud-friendly way. Over the course of the lesson, you will set up a Python environment, access example data, and explore formats such as NetCDF, Zarr, and STAC that are designed for scalable analysis. You will also learn key concepts such as chunking, metadata, parallel processing, and versioned data storage. The aim is to help you understand not only how these tools work, but also why they are useful for analysing, sharing, and managing environmental data more efficiently.

::::::::::::::::::::::::::::::::::::::: challenge

## What do you work with?

Before we start, take a moment to think about your own experience with environmental data. This short activity will help us understand the kinds of datasets and challenges you bring to the workshop.

1. In the shared notes document (CodiMD), write one sentence about the data you usually work with and roughly how large it is.
2. Describe one challenge you have faced when working with data that is too large for your computer or too slow to process.
3. List one tool, library, or computing system you have used to help with analysis, storage, or data access.

::::::::::::::::::::::::::::::::::::::::::::::::::

In this workshop, we will use:

- `jupyterlab`
- `numpy`
- `netCDF4`
- `xarray`
- `zarr`
- `dask`
- `fsspec`, `s3fs`, `boto3`, and `obstore` for cloud storage access
- `matplotlib` and `cartopy` for plotting
- `icechunk` for versioning
- `virtualizarr` for virtual Zarr stores
- `stac` for cataloging
- `topozarr` for multiscale Zarr visualization
- others...

The exact environment is provided in the course repository as an [environment.yml file for creating a conda environment](files/environment.yml).

## Jargon busting

Here are some of the main terms that appear throughout the workshop.

### CPU, core, process, and thread

The **CPU** is the Central Processing Unit that executes instructions and performs calculations. A **core** is one processing unit inside a CPU, while a **process** is one running instance of a program and a **thread** is a smaller unit of work inside that process.

### Parallel processing and Dask

**Parallel processing** means splitting work so that different parts run at the same time, often across cores, processes, or workers. **Dask** is a Python library that facilitates parallel computing by breaking down large computations into smaller tasks that can be executed concurrently.

### RAM and storage

**RAM** is the computer's short-term memory. **Storage** is where data live more permanently, such as on disk, SSDs, shared storage, or object storage.

### Cluster, node, and HPC

A **cluster** is a group of connected computers that work together, and a **node** is one computer within that cluster. **High-performance computing (HPC)** refers to large shared systems designed for heavy computation and large data processing.

![](fig/hpc.png){alt="A picture of the JASMIN HPC system."}

### JASMIN and SSH

[JASMIN](https://jasmin.ac.uk) is the UK's data analysis facility for data-intensive environmental science. It provides notebook services, shared storage, and computing resources for environmental data work. **SSH** (Secure Shell) is a secure way to connect to a remote computer over a network.

![](fig/jasmin.png){alt="An image illustrating the JASMIN system."}

### Group workspace

A **group workspace** is shared storage on JASMIN for collaborative work and course data.

### Jupyter notebook, and kernel

A **Jupyter notebook** is a browser-based environment for running code, text, and plots together. A **kernel** is the Python environment that runs notebook code.

### Environment.yml and conda/mamba

An **environment.yml** file is used to define a Conda environment, specifying the Python version and the packages required. **Conda** and **mamba** are tools for managing these environments and installing packages.

### Dataset, array, coordinate, and data variable

A **dataset** is a collection of related scientific data and metadata in different formats. An **array** is a multi-dimensional grid of values, while a **coordinate** is a named axis such as time, latitude, longitude, or depth. A **data variable** is the main measured or modelled value, such as temperature or salinity.

### Metadata and controlled vocabulary

**Metadata** is information about the data, such as units, long names, chunk sizes, and coordinate definitions. A **controlled vocabulary** is a curated list of standard terms and identifiers used to avoid ambiguity in metadata.

### Zarr, Xarray, chunk, and lazy loading

**Zarr** is a chunked data format for large N-dimensional arrays. **Xarray** is a labelled array library for working with multidimensional scientific data. A **chunk** is a smaller piece of a larger array, and **lazy loading** means data are not fully read into memory when a dataset is opened; they are loaded only when needed.

### Object storage and bucket

**Object storage** keeps data as independent objects in a storage system, often accessed through APIs such as S3. A **bucket** is a top-level container for objects in object storage.

![Source: https://blog.itkonekt.com/](fig/bucket_and_object_store.png){alt="A diagram showing a bucket containing objects in object storage."}

### POSIX and Object-Store file system

A **POSIX file system** is a traditional file-system model with directories, files, and paths that are accessed through a local or mounted storage interface. It is commonly used for shared filesystems and local disks.

An **object-store file system** is a storage interface that exposes object storage through filesystem-like operations, often used in cloud workflows. It is different from a traditional POSIX filesystem because data are accessed through object APIs rather than a normal directory tree.

### Cloud-native format

A **cloud-native format** is designed to work efficiently with object storage and remote access patterns, often by allowing partial reads and parallel access.

### STAC

**STAC** is a standard for describing and organizing geospatial assets.

![STAC example. Source: https://stacspec.org/](fig/stac_example.png){alt="An example of a STAC catalog."}

### Icechunk

**Icechunk** adds versioning and history tracking to Zarr stores.

### VirtualiZarr

**VirtualiZarr** lets you work with existing files as if they were Zarr datasets without copying all of the data.

### Multiscale and pyramid

A **multiscale** dataset stores the same data at multiple resolutions, and a **pyramid** is the layered structure that makes it possible to display coarse views quickly and zoom in later.


## Accessing the workshop environment

There are two ways to join the workshop:

- **In person:** use a JASMIN training account and the JASMIN notebook service.
- **Remote:** download the data locally and install the environment on your own computer.

::::::::::::::::: spoiler

## In-person setup (JASMIN)

We will be using the Notebooks service on the JASMIN system for this workshop. This will open a Jupyter notebook in your web browser, from which you can type in Python code and it will run on the JASMIN system. JASMIN is the UK's data analysis facility for environmental science and co-locates both data storage and data processing facilities. It will also be possible to run much of the code in this workshop on your own computer, but some of the larger examples will probably exceed the memory and processing power of your computer.

You received an email with instructions to access a JASMIN training account.

### Launching JupyterLab

In your browser, connect to [https://notebooks.jasmin.ac.uk](https://notebooks.jasmin.ac.uk). Please use the username and password provided in your training account by email. There is a two-factor authentication step on the notebook service that will email you a code; enter this code and you will be connected to the Notebook service.

### Setting up the environment

A preconfigured Conda environment is available for use: `cloud-native-geoscience-course`. This environment includes all necessary packages and dependencies and is built using the [environment.yml file available here](files/environment.yml).

Since the environment is stored in a non-standard location (`/work/scratch-nopw2/tobfer/cloud-native-geoscience-course-env`), Jupyter will not detect it automatically. Follow these steps to set it up:

- Open a Terminal.

From the Jupyter launcher, click the Terminal icon.

- Register the `cloud-native-geoscience-course` kernel.

Run the following command:

```bash
mamba run -p /work/scratch-nopw2/tobfer/cloud-native-geoscience-course-env python -m ipykernel install --user --name cloud-native-geoscience-course
```

If the command above fails, try running these commands first, then repeat the registration step:

```bash
mamba init
exec bash
```

- Launch Jupyter with the `cloud-native-geoscience-course` environment.

Open a new Jupyter launcher by clicking **File > New Launcher**. Then a new notebook and console option named `cloud-native-geoscience-course` should now be available. This may take about a minute to appear.

![Jupyter kernel choice](fig/jupyter-kernel-choice.png){alt="Jupyter kernel choice mine."}

Click on `cloud-native-geoscience-course` to open a notebook.

### Data access

In-person participants will use a shared JASMIN group workspace for the example data. You do not need to download the data locally. You only need to confirm that you can access the folder and read the files.

Please run the following command in a terminal on the JASMIN notebook service to check that you can access the data:

```bash
ls /gws/ssde/j25b/atlantis_vis/cloud-native-geoscience-course/
```

If you see a list of files, you have access to the data. If you see an error, please ask for help.

During the course, remember to use the full path `/gws/ssde/j25b/atlantis_vis/cloud-native-geoscience-course/` when accessing the data.

:::::::::::::::::::::::::

::::::::::::::::: spoiler

## Remote setup (your own computer)

If you are joining remotely, you will install the environment on your own computer.

### Before you start

You will need:

- Python 3.12 or newer.
- Conda or mamba installed.
- At least 8 GB of RAM, with 16 GB preferred.
- At least 10 GB of free disk space for the environment.
- At least 20 GB of free disk space for the example data.
- A stable internet connection for downloading packages and data.
- A terminal application (e.g., Terminal on macOS, Console or Terminal in Linux, or [Git Bash](https://git-scm.com/downloads) or [WSL](https://learn.microsoft.com/en-us/windows/wsl/install) on Windows).

I will show below the steps to install Python and conda/mamba on your own computer. If you already have a working Python 3.12 installation with conda or mamba, you can skip to the next section.

### Install Miniforge

If Conda has not been installed on your machine, then install it. You can install the [Anaconda distribution](https://www.anaconda.com/download). This is a conda distribution that includes many scientific packages. If you install Anaconda, you will have a working Python environment with conda already installed.

However, Anaconda is large and may take a long time to install.

Another option is to install [Miniforge](https://conda-forge.org/download/) for your OS. As the name suggests, Miniforge is a "mini" version of the Anaconda Python distribution that includes only Conda, a Python 3 distribution, and any necessary OS-specific dependencies.

For convenience, here are links to the 64-bit Miniforge installers:

- [Windows](https://github.com/conda-forge/miniforge/releases/latest/download/Miniforge3-Windows-x86_64.exe)
- [Mac OSX - Intel CPU](https://github.com/conda-forge/miniforge/releases/latest/download/Miniforge3-MacOSX-x86_64.sh)
- [Mac OSX - Apple M1/2/3 CPU](https://github.com/conda-forge/miniforge/releases/latest/download/Miniforge3-MacOSX-arm64.sh)
- [Linux](https://github.com/conda-forge/miniforge/releases/latest/download/Miniforge3-Linux-x86_64.sh)

::::::::::::::::: spoiler

#### Windows installation

If you are using miniforge on Windows, after you download the [Windows installer](https://github.com/conda-forge/miniforge/releases/latest/download/Miniforge3-Windows-x86_64.exe), double-click it and follow the instructions, including accepting the license.

Make sure you tick the **"Add Miniforge3 to my PATH environment variable"** option.

:::::::::::::::::::::::::

::::::::::::::::: spoiler

#### Mac OSX or Linux installation

If you are using miniforge on Mac OSX or Linux, you can install it from the command line. First, download the 64-bit Python 3 install script for Miniforge either by clicking the link above or using this command in your terminal:

```bash
wget "https://github.com/conda-forge/miniforge/releases/latest/download/Miniforge3-$(uname)-$(uname -m).sh"
```

Run the Miniforge install script from your terminal. Follow the prompts on the installer screens. If you are unsure about any setting, accept the defaults.

```bash
bash Miniforge3-$(uname)-$(uname -m).sh
```

Once the install script completes, you can remove it.

```bash
rm Miniforge3-$(uname)-$(uname -m).sh
```

:::::::::::::::::::::::::

### Verifying your Conda installation

To verify that you have installed Conda correctly, run the `conda help` command. The output should look similar to the following.

```bash
$ conda help
usage: conda [-h] [-V] command ...

conda is a tool for managing and deploying applications, environments and packages.

Options:

positional arguments:
  command
    clean        Remove unused packages and caches.
    config       Modify configuration values in .condarc. This is modeled
                 after the git config command. Writes to the user .condarc
                 file (/Users/drpugh/.condarc) by default.
    create       Create a new conda environment from a list of specified
                 packages.
    help         Displays a list of available conda commands and their help
                 strings.
    info         Display information about current conda install.
    init         Initialize conda for shell interaction. [Experimental]
    install      Installs a list of packages into a specified conda
                 environment.
    list         List linked packages in a conda environment.
    package      Low-level conda package utility. (EXPERIMENTAL)
    remove       Remove a list of packages from a specified conda environment.
    uninstall    Alias for conda remove.
    run          Run an executable in a conda environment. [Experimental]
    search       Search for packages and display associated information. The
                 input is a MatchSpec, a query language for conda packages.
                 See examples below.
    update       Updates conda packages to the latest compatible version.
    upgrade      Alias for conda update.

optional arguments:
  -h, --help     Show this help message and exit.
  -V, --version  Show the conda version number and exit.

conda commands available from other packages:
  env
```

At the bottom of the help menu you will see a section with some optional arguments for the `conda` command. In particular, you can pass the `--version` flag, which will return the version number. Again, the output should look similar to the following:

```bash
$ conda --version
conda 4.8.2
```

### Recommended folder layout

We suggest a simple folder structure like this:

```text
cloud-native-geoscience-course/
├── environment/
└── data/
```

Keep the environment files in `cloud-native-geoscience-course/environment/` and the data in `cloud-native-geoscience-course/data/`. And save all the jupyter notebooks you create in the `cloud-native-geoscience-course/` folder. This will make it easier to find your files later.

Please create the `cloud-native-geoscience-course/` folder in your home directory, and then create the `environment/` and `data/` subfolders.

```bash
mkdir -p ~/Documents/cloud-native-geoscience-course/environment
mkdir -p ~/Documents/cloud-native-geoscience-course/data
```

### Install the environment

Download the `environment.yaml` file from the lesson repository:

```bash
cd ~/Documents/cloud-native-geoscience-course/environment

curl -L https://raw.githubusercontent.com/NOC-OI/cloud-native-geoscience-course/refs/heads/main/episodes/files/environment.yml -o environment.yaml
```

Then create the environment:

```bash
conda env create -f environment.yaml
```

**Note:* This command can take several minutes to complete, depending on your internet connection and computer speed.**

This will create a new environment with all the required packages. The environment name is defined in the `environment.yaml` file, and it is **cloud-native-geoscience-course** for this workshop.
Now activate the environment:

```bash
conda activate cloud-native-geoscience-course
```

If the command above fails, try running these commands first, then repeat the activation step:

```bash
conda init
exec bash
```

### Download the example data

Remote participants need to download the example data locally.

The full example data is about 20 GB, so make sure you have enough free disk space before you start.

Download the zip file from the lesson repository, then unzip it into your `cloud-native-geoscience-course/data/` folder:

```bash
curl -L https://atlantis-vis-o.s3-ext.jc.rl.ac.uk/cloud-native-geoscience-course/data.zip -o data.zip
unzip data.zip -d ~/Documents/cloud-native-geoscience-course/data/
```

**Note:** The unzip process can take some time, because we are extracting a large number of files. You can check the progress by running `ls -lh ~/Documents/cloud-native-geoscience-course/data/` in another terminal window.

### Launch Python interface

To start working with Python, we need to launch a program that will interpret and execute our Python commands. In this workshop, we are working mainly in Jupyter notebooks.

A Jupyter notebook provides a browser-based interface for working with Python. You can launch a notebook from the terminal:

::::::::::::::::: spoiler

## Unix shell

Navigate to the `cloud-native-geoscience-course/` directory. If you're using a Unix shell application, such as Terminal on macOS, Console or Terminal in Linux, or [Git Bash][gitbash] on Windows, execute the following command:

```bash
cd ~/Documents/cloud-native-geoscience-course/
```

To launch the Jupyter server, run:

```bash
jupyter lab
```

:::::::::::::::::::::::::

::::::::::::::::: spoiler

## Command Prompt (Windows)

On Windows, you can use its native Command Prompt program. The easiest way to start it up is by pressing <kbd>Windows Logo Key</kbd>+<kbd>R</kbd>, entering `cmd`, and hitting <kbd>Return</kbd>. In the Command Prompt, use the following command to navigate to the `cloud-native-geoscience-course/` folder:

```source
cd /D %userprofile%\Documents\cloud-native-geoscience-course/
```

To launch the Jupyter server, run:

```source
python -m jupyter lab
```

:::::::::::::::::::::::::

### Access the notebooks
Once the Jupyter server is running, it will open a new tab in your web browser showing the notebook dashboard. Launch the notebook by clicking on the "New" button on the right and selecting "Python 3" from the drop-down menu:

![](fig/jupyter-notebook-launch-notebook2.png){alt='Anaconda Navigator Notebook directory'}


:::::::::::::::::::::::::


### Test the installation

Run the following code to check that the core packages are available:

```python
import numpy
import xarray
import dask
import zarr
import netCDF4
import matplotlib
import cartopy
import numba
import fsspec
import icechunk
import virtualizarr
import pystac
import topozarr

print("numpy:", numpy.__version__)
print("xarray:", xarray.__version__)
print("dask:", dask.__version__)
print("zarr:", zarr.__version__)
print("netCDF4:", netCDF4.__version__)
print("matplotlib:", matplotlib.__version__)
print("cartopy:", cartopy.__version__)
print("numba:", numba.__version__)
print("fsspec:", fsspec.__version__)
print("icechunk:", icechunk.__version__)
print("virtualizarr:", virtualizarr.__version__)
print("pystac:", pystac.__version__)
print("topozarr:", topozarr.__version__)
```

You should see this output:

```output
numpy: 1.26.2
xarray: 2024.12.0
dask: 2024.12.0
zarr: 2.24.0
netCDF4: 1.6.4
matplotlib: 3.9.2
cartopy: 0.24.1
numba: 0.59.0
fsspec: 2024.12.0
icechunk: 0.1.0
virtualizarr: 0.1.0
pystac: 1.6.0
topozarr: 0.1.0
```

## About the example data

The workshop uses several example datasets that represent common environmental data products in the ocean and atmosphere domains. These examples are provided in different formats, including NetCDF, GRIB, and Zarr, so you can see how the same scientific data can be organised and accessed in different ways.

The data come from public or widely used sources such as:

- ERA5 reanalysis fields from the [Copernicus Climate Data Store](https://cds.climate.copernicus.eu/datasets/reanalysis-era5-single-levels?tab=analysis_ready_data) (ECMWF), including wave and surface variables.
- GLORYS reanalysis products from the [Copernicus Marine Service](https://data.marine.copernicus.eu/product/GLOBAL_MULTIYEAR_PHY_001_030/description).


The data are chunked, so you can work with small parts of the dataset instead of loading everything at once.

### What to expect

- The data are large enough that you should not expect to open the full dataset into memory.
- You will often work on one variable, one time slice, or one small spatial region.
- This is intentional: the workshop is about working efficiently with large datasets.
