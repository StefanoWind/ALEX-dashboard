# ALEX-dashboard
Deployment of the ALEX hazard library

An interactive [Streamlit](https://streamlit.io) dashboard to browse the event library produced by the ALEX pipeline: event table and filters, outage probabilities, feature importance, time series of each episode, and NEXRAD radar scans around the event.

## Requirements

- Python 3.11 or newer (3.12 recommended)
- [Git](https://git-scm.com/downloads)
- [Miniforge](https://github.com/conda-forge/miniforge) (or any conda/mamba installation) — recommended, because `Cartopy`, `arm_pyart`, `netCDF4` and `cfgrib` depend on compiled geospatial libraries (GEOS, PROJ, HDF5, ecCodes) that conda-forge ships prebuilt for Windows, macOS (Intel and Apple Silicon) and Linux
- Internet access while running (NEXRAD scans are fetched on demand from the public AWS archive, and map tiles from OpenTopoMap)

## Installation

### 1. Clone the repository

```bash
git clone https://github.com/StefanoWind/ALEX-dashboard.git
cd ALEX-dashboard
```

### 2a. Create the environment with conda (recommended, all platforms)

Open a terminal (on Windows, use the **Miniforge Prompt**) and run:

```bash
conda create -n alex-dashboard -c conda-forge python=3.12 cartopy arm_pyart netcdf4 cfgrib eccodes
conda activate alex-dashboard
pip install -r requirements.txt
```

The conda step installs the packages with compiled dependencies; `pip` then installs the remaining pure-Python packages and skips those already present.

### 2b. Alternative: plain `venv` + `pip`

This works on most Linux and macOS machines and on recent Windows installations, where binary wheels are available for all packages.

macOS / Linux:

```bash
python3 -m venv .venv
source .venv/bin/activate
python -m pip install --upgrade pip
pip install -r requirements.txt
```

Windows (PowerShell):

```powershell
py -m venv .venv
.venv\Scripts\Activate.ps1
python -m pip install --upgrade pip
pip install -r requirements.txt
```

If PowerShell blocks the activation script, run `Set-ExecutionPolicy -Scope CurrentUser RemoteSigned` once. If `pip` fails while building `Cartopy`, `arm_pyart`, `netCDF4` or `cfgrib`, use the conda route (2a) instead.

### 3. Add the event library data

The library files are not tracked in git (`*.xlsx` and `*.nc` are ignored). Copy the output of the ALEX pipeline into:

```
ALEX-dashboard/
└── data/
    ├── merged_outages_gf_15min/
    │   ├── <source>.library.xlsx      # one per data source (sheets: Library, Data, Site, ...)
    │   └── <source>.nc                # matching NetCDF file with the raw time series
    └── nexrad_cache/                  # created automatically; stores downloaded radar scans
```

The dashboard automatically lists every `*.library.xlsx` found in `data/merged_outages_gf_15min/`. A different library file can also be selected from the sidebar with **Manual path override**.

## Running the dashboard

From the repository folder, with the environment activated:

```bash
streamlit run dashboard.py
```

Streamlit prints a local URL (by default http://localhost:8501) and opens it in the browser. Stop the server with `Ctrl+C`.

Useful options:

```bash
streamlit run dashboard.py --server.port 8502          # use a different port
streamlit run dashboard.py --server.headless true      # do not open a browser (e.g., on a remote server)
```

To run on a remote machine (e.g., HPC or a cloud VM), start it headless and forward the port over SSH from your laptop:

```bash
ssh -L 8501:localhost:8501 <user>@<remote-host>
```

then open http://localhost:8501 locally.

## Updating

```bash
git pull
pip install -r requirements.txt --upgrade
```

## Troubleshooting

- **`ModuleNotFoundError`**: the environment is not activated; run `conda activate alex-dashboard` (or activate `.venv`) before `streamlit run`.
- **No sources listed / file not found**: check that the `.library.xlsx` and `.nc` files are in `data/merged_outages_gf_15min/`.
- **Radar panel empty or slow**: the first view of an event downloads NEXRAD scans from AWS; later views load from `data/nexrad_cache/`. The cache can be deleted at any time to free disk space.
- **SHAP plots missing**: `shap` failed to install; the rest of the dashboard still works. Reinstall it with `pip install shap`.
