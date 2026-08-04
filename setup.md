---
title: Setup
---


## Setup

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

If you see a folder called `data`, you have access to the example data. If you see an error, please ask for help.

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
- At least 4 GB of free disk space for the environment.
- At least 30 GB of free disk space for the example data and for data that will be generated during the workshop.
- A stable internet connection for downloading packages and data.
- A terminal application (e.g., Terminal on macOS, Console or Terminal in Linux, or [Git Bash](https://git-scm.com/downloads) or [WSL](https://learn.microsoft.com/en-us/windows/wsl/install) on Windows).

The steps below describe how to install Python and conda/mamba on your own computer. If you already have a working Python 3.12 installation with conda or mamba, you can skip to the next section.

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
conda 24.11.3 # or similar, depending on the version you installed
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
mkdir -p ~/cloud-native-geoscience-course/environment
mkdir -p ~/cloud-native-geoscience-course/data
```

### Install the environment

Download the [`environment.yml`](files/environment.yml) file from the lesson repository:

```bash
cd ~/cloud-native-geoscience-course/environment

curl -L https://raw.githubusercontent.com/NOC-OI/prep-work-cloud-native-geoscience-course/refs/heads/main/episodes/files/environment.yml -o environment.yml
```

Then create the environment:

```bash
conda env create -f environment.yml
```

**Note:* This command can take several minutes to complete, depending on your internet connection and computer speed.**

This will create a new environment with all the required packages. The environment name is defined in the `environment.yml` file, and it is **cloud-native-geoscience-course** for this workshop.
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
curl -L https://atlantis-vis-o.s3-ext.jc.rl.ac.uk/cloud-native-geoscience-course/cloud-native-geoscience-course-data.tar.gz -o data.tar.gz
tar -xzf data.tar.gz -C ~/cloud-native-geoscience-course/
```

**Note:** The extraction process can take some time, because we are extracting a large number of files. You can check the progress by running `ls -lh ~/cloud-native-geoscience-course/data/` in another terminal window.

### Launch Python interface

To start working with Python, we need to launch a program that will interpret and execute our Python commands. In this workshop, we are working mainly in Jupyter notebooks.

A Jupyter notebook provides a browser-based interface for working with Python. You can launch a notebook from the terminal:

::::::::::::::::: spoiler

## Unix shell

Navigate to the `cloud-native-geoscience-course/` directory. If you're using a Unix shell application, such as Terminal on macOS, Console or Terminal in Linux, or [Git Bash][gitbash] on Windows, execute the following command:

```bash
cd ~/cloud-native-geoscience-course/
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
cd /D %userprofile%\cloud-native-geoscience-course/
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
import pandas
import xarray
import s3fs
import dask
import dask_gateway
import zarr
import netCDF4
import matplotlib
import cartopy
import fsspec
import icechunk
import virtualizarr
import pystac
import topozarr
import boto3
import shapely
import obstore
import eccodes
import cfgrib

print("numpy:", numpy.__version__)
print("pandas:", pandas.__version__)
print("xarray:", xarray.__version__)
print("s3fs:", s3fs.__version__)
print("dask:", dask.__version__)
print("dask_gateway:", dask_gateway.__version__)
print("zarr:", zarr.__version__)
print("netCDF4:", netCDF4.__version__)
print("matplotlib:", matplotlib.__version__)
print("cartopy:", cartopy.__version__)
print("fsspec:", fsspec.__version__)
print("icechunk:", icechunk.__version__)
print("virtualizarr:", virtualizarr.__version__)
print("pystac:", pystac.__version__)
print("topozarr:", topozarr.__version__)
print("boto3:", boto3.__version__)
print("shapely:", shapely.__version__)
print("obstore:", obstore.__version__)
print("eccodes:", eccodes.__version__)
print("cfgrib:", cfgrib.__version__)
```

You should see something like:

```output
numpy: 2.4.3
pandas: 3.0.2
xarray: 2026.7.0
s3fs: 2026.4.0
dask: 2026.3.0
dask_gateway: 2026.3.0
zarr: 3.2.1
netCDF4: 1.7.4
matplotlib: 3.11.1
cartopy: 0.25.0
fsspec: 2026.4.0
icechunk: 2.1.1
virtualizarr: 2.7.1
pystac: 1.15.2
topozarr: 0.1.2
boto3: 1.43.46
shapely: 2.1.2
obstore: 0.11.0
eccodes: 2.48.0
cfgrib: 0.9.15.1
```

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
