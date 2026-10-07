# Forcing data

iSnobal requires the following hourly weather forcing for every grid cell:
- air temperature,
- humidity,
- wind,
- precipitation,
- net solar radiation,
- and incoming longwave radiation.

SMRF builds these grids at run time.

The recommended input is [HRRR](https://rapidrefresh.noaa.gov/hrrr/), NOAA's hourly
3 km weather model (archived from 2014), with wind downscaled by Katana.

Not using HRRR? See [Other forcing sources](forcings/other_sources.md) for weather
stations and WRF.

The HRRR forcings workflow is detailed below.

```{mermaid}
flowchart LR
    A[HRRR archive<br/>Google, AWS, Azure, UofU] -->|download_hrrr.sh| B[HRRR GRIB2 files<br/>hourly, 3 km]
    B -->|Katana + WindNinja| C[Downscaled wind<br/>200 m]
    T[topo.nc] --> C
    B --> D[SMRF, run by awsm]
    C --> D
```

| Step | Tool | Environment | Details |
|---|---|---|---|
| 1. Install envs | `conda/hrrr.yaml`, `conda/katana.yaml` | `hrrr`, `katana` |
| [2. Download HRRR](#download-hrrr) | `scripts/HRRR/download_hrrr.sh` | `hrrr` | [Variables](forcings/hrrr_variables.md) · [Download area](forcings/hrrr_area.md) |
| [3. Downscale wind](#downscale-wind) | Katana (runs WindNinja) | `katana` | [Katana configuration](forcings/katana.md) |
| [4. Direct AWSM to HRRR and wind data](#point-awsm) | `config/awsm.ini` | `isnobal` | [Configuration](configuration.md) |

## 1. Install the two helper environments from the `model_setup` repo folder:

```bash
conda env create -f conda/hrrr.yaml
conda env create -f conda/katana.yaml
```

(download-hrrr)=
## 2. Download HRRR

`download_hrrr.sh` fetches only the [variables](forcings/hrrr_variables.md) SMRF and Katana
need, cropped to the western US ([you can also customize the area to download](forcings/hrrr_area.md)), for forecast
hours 1 and 6 of every hour of the day. Download daily forcings like so:

```bash
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
<!-- TODO: what's our current management of missing variables/download errors? -->
- **HPC:** `scripts/HRRR/HRRR.slurm` is a SLURM template for downloading several months.

(downscale-wind)=
## 3. Downscale wind with Katana

HRRR's 3 km winds miss sub-grid terrain effects that drive snow redistribution. Katana runs
[WindNinja](https://ninjastorm.firelab.org/windninja/) to downscale HRRR winds to the basin
terrain, defaulting to a 200 m resolution. Set up `katana.ini` as described in
[Katana configuration](forcings/katana.md), then run:

```bash
conda activate katana
run_katana katana.ini
```

Hourly outputs are stored in daily folders, `/path/to/katana/data<YYYYMMDD>/wind_ninja_data/` and contains information about wind speed (`topo_windninja_topo_MM-DD-YYYY_HH00_200m_vel.asc`) and direction (`topo_windninja_topo_MM-DD-YYYY_HH00_200m_ang.asc`).

(point-awsm)=
## 4. Point AWSM to the forcings

Set the forcing sources (`/path/to/...`) in your [AWSM configuration](configuration.md):

```ini
[gridded]
data_type:                  hrrr_grib
hrrr_directory:             /path/to/HRRR
hrrr_sixth_hour_variables:  precip_int

[wind]
wind_model:                 wind_ninja
wind_ninja_dir:             /path/to/katana
wind_ninja_dxdy:            200
```

#### *Notes*
- `hrrr_sixth_hour_variables: precip_int` takes precipitation from the 6-hour forecast
([see here](forcings/hrrr_variables.md#apcp) for more detail).
- `wind_ninja_dxdy` must match the Katana
`mesh_resolution`.
