# Forcing data

iSnobal requires hourly weather forcing (shown below) for every grid cell. SMRF builds these forcing grids at run time.
```text
- air temperature
- humidity
- wind
- precipitation
- net solar radiation
- incoming longwave radiation
```

The recommended distributed input is [HRRR](https://rapidrefresh.noaa.gov/hrrr/), NOAA's hourly
3 km weather model (archived from 2014), with wind downscaled by Katana. For using weather station
or other distributed forcing inputs, please see the section on [other forcing sources](forcings/other_sources.md).

HRRR forcing data is downloaded and processed as follows:

```{mermaid}
flowchart LR
    A["`**HRRR archive** `"<br/>Google, AWS, Azure, UofU] -->|download_hrrr.sh| B["`**HRRR GRIB2** `" <br/>hourly files, 3 km]
    B -->|Katana + WindNinja| C[Downscaled wind<br/>200 m]
    T[topo.nc] --> C
    B --> D[SMRF, run by awsm]
    C --> D
```

| Step | Code | Environment | Details |
|---|---|---|---|
| 1. [Install conda environments](#install-envs) | `conda/hrrr.yaml`, `conda/katana.yaml` | `hrrr`, `katana` |
| 2. [Download HRRR](#download-hrrr) | `scripts/HRRR/download_hrrr.sh` | `hrrr` | [Variables](forcings/hrrr_variables.md) · [Download area](forcings/hrrr_area.md) |
| 3. [Downscale wind](#downscale-wind) | Katana (runs WindNinja) | `katana` | [Katana configuration](forcings/katana.md) |
| 4. [Direct AWSM to HRRR and wind data](#point-awsm) | `config/awsm.ini` | `isnobal` | [Configuration file](configuration.md) |

(install-envs)=
## 1. Install helper environments
The conda environment yaml files are located in the `model_setup` repo folder:

```{code} bash
:class: shell-prompt
conda env create -f conda/hrrr.yaml
conda env create -f conda/katana.yaml
```

(download-hrrr)=
## 2. Download HRRR data

`download_hrrr.sh` fetches only the [variables](forcings/hrrr_variables.md) SMRF and Katana
need, cropped to the default western US bounds. [Customization](forcings/hrrr_area.md) of the download area is also possible.
Variables are downloaded for forecast hours 1 and 6 for every hour of the day.

Download daily forcings like so:

```{code} bash
:class: shell-prompt
conda activate hrrr
mkdir -p /path/to/HRRR && cd /path/to/HRRR

/path/to/model_setup/scripts/HRRR/download_hrrr.sh 20211001            # one day
/path/to/model_setup/scripts/HRRR/download_hrrr.sh 20211001,20211031   # date range
/path/to/model_setup/scripts/HRRR/download_hrrr.sh 2021 10             # whole month
/path/to/model_setup/scripts/HRRR/download_hrrr.sh 20211001 AWS        # choose archive
```

Each day gets a folder, `hrrr.YYYYMMDD/`, of 48 files (24 hours × 2 forecast hours). Plan
for about 170 MB per day, or 60 GB per water year, with the default download area.

- **Data archives:** Google is the default. Other archives can be specified or are used for attempted retrievals of missing files.
- **HPC:** `scripts/HRRR/HRRR.slurm` is a SLURM template for downloading several months.

```{note}
A "forecast hour" indicates how far in advance a specific forecast was made for the specified hour. For example, forecast
hour 6 (`f6`) for the 1400 hour (2PM) was made at 8AM (6 hours before hour 14).
```

(downscale-wind)=
## 3. Downscale wind with Katana

HRRR's 3 km winds miss sub-grid terrain effects that drive snow redistribution. Katana runs
[WindNinja](https://ninjastorm.firelab.org/windninja/) to downscale HRRR winds to basin
terrain, defaulting to a 200 m resolution. Set up `katana.ini` as described in
[Katana configuration](forcings/katana.md), then run:

```{code} bash
:class: shell-prompt
conda activate katana
run_katana katana.ini
```

Hourly outputs are stored in daily folders, `/path/to/katana/output/data<YYYYMMDD>/wind_ninja_data/` and contains information about wind speed (`topo_windninja_topo_MM-DD-YYYY_HH00_200m_vel.asc`) and direction (`topo_windninja_topo_MM-DD-YYYY_HH00_200m_ang.asc`).

(point-awsm)=
## 4. Point AWSM to the forcings
Now that forcings are prepared, update the directory paths to your forcing sources
(`/path/to/...`) in your [AWSM configuration](configuration.md):

```ini
[gridded]
data_type:                  hrrr_grib
hrrr_directory:             /path/to/HRRR
hrrr_sixth_hour_variables:  precip_int

[wind]
wind_model:                 wind_ninja
wind_ninja_dir:             /path/to/katana/output
wind_ninja_dxdy:            200
```

```{note}
- `hrrr_sixth_hour_variables: precip_int` takes precipitation from the 6-hour forecast
([see here](forcings/hrrr_variables.md#apcp) for more detail).
- `wind_ninja_dxdy` must match the Katana
`mesh_resolution`.
```
