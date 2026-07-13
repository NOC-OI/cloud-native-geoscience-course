---
title: Setup
---

## Introduction

This workshop uses Python tools for working with large environmental datasets. You will install the workshop environment, connect to the lesson data, and learn the key terms that will appear throughout the workshop.

::::::::::::::::::::::::::::::::::::::: challenge

## What do you work with?

1. In the Etherpad, write a sentence about the data you work with and how large it is.
2. Describe any problems you have had when working with data that is too large for your computer.
3. List any tools, libraries, or computing systems you have used to help with this.

::::::::::::::::::::::::::::::::::::::::::::::::::

In this workshop, we will use:

- `numpy`.
- `xarray`.
- `zarr`.
- `dask`.
- `fsspec`.
- `s3fs` or other storage backends when needed.
- `matplotlib`.
- `cartopy` for maps.
- `netCDF4`.
- `jupyterlab`.
- `ipykernel`.

The exact environment is provided in the lesson repository as an [environment.yaml](environment.yaml) file.

## Jargon busting

### CPU

The **CPU** is the Central Processing Unit. It runs instructions and performs calculations for the computer.

### Core

A **core** is one processing unit inside a CPU. More cores can allow more work to happen at the same time.

### Process

A **process** is one running instance of a program. When you start Python, you create a Python process.

### Thread

A **thread** is a smaller unit of work inside a process. Threads share memory within the same process.

### Parallel processing

**Parallel processing** means splitting work so that different parts run at the same time.

### RAM

**RAM** is the computer's short-term memory. Data must be loaded into RAM before it can be processed.

### Storage

**Storage** is where data live more permanently, such as on disk, SSD, shared storage, or object storage.

### Cluster

A **cluster** is a group of connected computers that work together.

### Node

A **node** is one computer inside a cluster.

### HPC

**High-performance computing (HPC)** refers to large shared systems built for heavy computation and large data processing.


![](fig/hpc.png){alt="A picture of the JASMIN HPC system."}

### JASMIN

[JASMIN](https://jasmin.ac.uk) is the UK's data analysis facility for data-intensive environmental science. It provides notebook services, shared storage, and computing resources for environmental data work.

![](fig/jasmin.png){alt="An image illustrating the JASMIN system."}

### SSH

**SSH** (Secure Shell) is a secure way to connect to a remote computer over a network.

### Jupyter notebook

A **Jupyter notebook** is a browser-based environment for running code, text, and plots together.

### Kernel

A **kernel** is the Python environment that runs notebook code.

### Conda / Mamba

**Conda** and **mamba** are tools for creating and managing Python environments.

### Environment

An **environment** is a self-contained set of Python packages and versions.

### Zarr

**Zarr** is a chunked data format for large N-dimensional arrays.

### Xarray

**Xarray** is a labelled array library for working with multidimensional scientific data.

### Chunk

A **chunk** is a smaller piece of a larger array. Zarr and xarray use chunks so they can read only part of a dataset at a time.

### Lazy loading

**Lazy loading** means data are not fully read into memory when a dataset is opened. The data are only loaded when you select, compute, or plot them.

### Coordinate

A **coordinate** is a named dimension value such as time, latitude, longitude, or depth.

### Data variable

A **data variable** is the main measured or modelled data, such as temperature or salinity.

### Metadata

**Metadata** is information about the data, such as units, long names, chunk sizes, and coordinate definitions.

### Group workspace

A **group workspace** is shared storage on JASMIN for collaborative work and course data.


## Accessing the workshop environment

There are two ways to join the workshop:

- **In person:** use a JASMIN training account and the JASMIN notebook service.
- **Remote:** install the environment on your own computer.

::::::::::::::::: spoiler

## In-person setup (JASMIN)

We will be using the Notebooks service on the JASMIN system for this workshop. This will open a Jupyter notebook in your web browser, from which you can type in Python code and it will run on the JASMIN system. JASMIN is the UK's data analysis facility for environmental science and co-locates both data storage and data processing facilities. It will also be possible to run much of the code in this workshop on your own computer, but some of the larger examples will probably exceed the memory and processing power of your computer.

You received an email with instructions to access a JASMIN training account.

### Launching JupyterLab

In your browser, connect to [https://notebooks.jasmin.ac.uk](https://notebooks.jasmin.ac.uk). Please use the username and password provided in your training account by email. There is a two-factor authentication step on the notebook service that will email you a code; enter this code and you will be connected to the Notebook service.

### Setting up the environment

A preconfigured INPO Conda environment is available for use. This environment includes all necessary packages and dependencies and is built using the `environment.yml` file from the Paidiverpy GitHub repository.

Since the environment is stored in a non-standard location (`/gws/nopw/j04/paidiver/paidiver-env`), Jupyter will not detect it automatically. Follow these steps to set it up:

- Open a Terminal.

From the Jupyter launcher, click the Terminal icon.

- Register the INPO kernel.

Run the following command:

```bash
mamba run -p /gws/nopw/j04/paidiver/paidiver-env python -m ipykernel install --user --name INPO
```

If the command above fails, try running these commands first, then repeat the registration step:

```bash
mamba init
exec bash
```

- Launch Jupyter with the INPO environment.

Open a new Jupyter launcher by clicking **File > New Launcher**. Then a new notebook and console option named INPO should now be available. This may take about a minute to appear.

![Jupyter kernel choice](fig/jupyter-kernel-choice.png){alt="Jupyter kernel choice mine."}

Click on INPO to open a notebook.

### Data access

In-person participants will use a shared JASMIN group workspace for the example data. You do not need to download the data locally. You only need to confirm that you can access the folder and read the files.

Please run the following command in a terminal on the JASMIN notebook service to check that you can access the data:

```bash
ls /gws/nopw/j04/paidiver/data
```

If you see a list of files, you have access to the data. If you see an error, please ask for help.

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

I will show below the steps to install Python and conda/mamba on your own computer. If you already have a working Python 3.12 installation with conda or mamba, you can skip to the next section.

### Install Miniforge

If Conda has not been installed on your machine, then install [Miniforge](https://conda-forge.org/download/) for your OS. As the name suggests, Miniforge is a "mini" version of the [Anaconda Python distribution](https://www.anaconda.com/download) that includes only Conda, a Python 3 distribution, and any necessary OS-specific dependencies.

For convenience, here are links to the 64-bit Miniforge installers:

- [Windows](https://github.com/conda-forge/miniforge/releases/latest/download/Miniforge3-Windows-x86_64.exe)
- [Mac OSX - Intel CPU](https://github.com/conda-forge/miniforge/releases/latest/download/Miniforge3-MacOSX-x86_64.sh)
- [Mac OSX - Apple M1/2/3 CPU](https://github.com/conda-forge/miniforge/releases/latest/download/Miniforge3-MacOSX-arm64.sh)
- [Linux](https://github.com/conda-forge/miniforge/releases/latest/download/Miniforge3-Linux-x86_64.sh)

You can also use the Anaconda distribution, but it is larger and may take longer to install. If you already have Anaconda installed, you can skip this step.

::::::::::::::::: spoiler

#### Windows installation


After you download the [Windows installer](https://github.com/conda-forge/miniforge/releases/latest/download/Miniforge3-Windows-x86_64.exe), double-click it and follow the instructions, including accepting the license.

Make sure you tick the **"Add Miniforge3 to my PATH environment variable"** option.


:::::::::::::::::::::::::

::::::::::::::::: spoiler

#### Mac OSX or Linux installation

First, download the 64-bit Python 3 install script for Miniforge either by clicking the link above or using this command in your terminal:

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
workshop/
├── environment/
├── data/
└── notebooks/
```

Keep the environment files in `workshop/environment/` and the data in `workshop/data/`.

### Install the environment

Download the `environment.yaml` file from the lesson repository:

```bash
curl -L <LINK-TO-ENVIRONMENT-YAML> -o environment.yaml
```

Then create the environment:

```bash
conda env create -f environment.yaml
```

This will create a new environment with all the required packages. The environment name is defined in the `environment.yaml` file, and it is **INPO** for this workshop.

```bash
conda activate INPO
```

If the command above fails, try running these commands first, then repeat the activation step:

```bash
conda init
exec bash
```

### Download the example data

Remote participants need to download the example data locally.

The full example data is about 20 GB, so make sure you have enough free disk space before you start.

Download the zip file from the lesson repository, then unzip it into your `workshop/data/` folder:

```bash
curl -L <LINK-TO-DATA-ZIP> -o data.zip
unzip data.zip -d data/
```

If you are on Windows, you can use a graphical unzip tool or a command-line tool that can extract zip archives.

### Launch Python interface

To start working with Python, we need to launch a program that will interpret and execute our Python commands. In this workshop, we are working mainly in Jupyter notebooks.

A Jupyter notebook provides a browser-based interface for working with Python. You can launch a notebook from the terminal:

::::::::::::::::: spoiler

## Unix shell

Navigate to the `data` directory. If you're using a Unix shell application, such as Terminal on macOS, Console or Terminal in Linux, or [Git Bash][gitbash] on Windows, execute the following command:

```bash
cd ~/Desktop/swc-python/data
```

To launch the Jupyter server, run:

```bash
jupyter notebook
```

:::::::::::::::::::::::::

::::::::::::::::: spoiler

## Command Prompt (Windows)

On Windows, you can use its native Command Prompt program. The easiest way to start it up is by pressing <kbd>Windows Logo Key</kbd>+<kbd>R</kbd>, entering `cmd`, and hitting <kbd>Return</kbd>. In the Command Prompt, use the following command to navigate to the `data` folder:

```source
cd /D %userprofile%\Desktop\swc-python\data
```

To launch the Jupyter server, run:

```source
python -m notebook
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

print("numpy:", numpy.__version__)
print("xarray:", xarray.__version__)
print("dask:", dask.__version__)
print("zarr:", zarr.__version__)
print("netCDF4:", netCDF4.__version__)
print("matplotlib:", matplotlib.__version__)
print("cartopy:", cartopy.__version__)
print("numba:", numba.__version__)
print("fsspec:", fsspec.__version__)
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
```

## About the example data

The workshop uses a large environmental dataset stored in Zarr format. The data are chunked, so you can work with small parts of the dataset instead of loading everything at once.

### What to expect

- The data are large enough that you should not expect to open the full dataset into memory.
- You will often work on one variable, one time slice, or one small spatial region.
- This is intentional: the workshop is about working efficiently with large datasets.

### Why this matters

Zarr and xarray make it possible to inspect and analyse only the portion of data you need. That is especially useful on laptops, on JASMIN notebooks, and in cloud-style workflows.
