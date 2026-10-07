# Run iSnobal over the Reynolds Mountain East test basin

Reynolds Mountain East (RME) is located within the Reynolds Creek Experimental Watershed, Idaho. This small test basin ships
with AWSM (an 18 × 19 grid at 50 m resolution) and forced by two weather stations for one day of a snowstorm in February 1986. 

The [`isnobal`](install.md) environment is required.

## Get the test basin

The RME files live in the AWSM repository. Clone it and copy the basin to a working
folder so the original stays clean:

```bash
git clone --depth 1 https://github.com/iSnobal/awsm.git
cp -r awsm/awsm/tests/basins/RME rme_testrun
cd rme_testrun
```

The folder contains the following:

| Path | Contents |
|---|---|
| `config.ini` | Model configuration, read by AWSM |
| `topo/topo.nc` | Elevation, basin mask, and vegetation grids |
| `topo/maxus_100window.nc` | Terrain exposure used to adjust wind |
| `station_data/*.csv` | Hourly station data and station locations (`metadata.csv`) |
| `gold/` | Reference outputs used by the AWSM tests |


## Set the start time
<!-- TODO: figure out whether config.ini needs to change to 00:00 start -->

`awsm` runs entire days, from the start time forward 24 hours. The test config starts
at 01:00, which would run past the end of the station data. Open `config.ini` and set
the start to midnight in the `[time]` section:

```ini
start_date:                    1986-02-17 00:00:00
```

The `end_date` is not used for a one-day run.

## Run the model

Create the `output` folder within `rme_testrun`, then run AWSM:

```bash
mkdir output
awsm -c config.ini --no_previous
```

`--no_previous` tells AWSM this is the first day, so there is no previous day's snow
state to start from. The run starts with no snow on the ground.

<!-- TODO: check on this exit status issue -->
```{warning}
If the configuration has errors, AWSM prints a report and stops, but still exits with
status 0. Check the terminal output rather than relying on the exit code.
```

## Find the outputs

AWSM writes one folder per day, named from the `[paths]` section of the config:

```text
output/rme/wy1986/rme_test/run19860217_19860217/
├── snow.nc          snow state: depth, density, SWE, temperatures, liquid water
├── em.nc            energy and mass fluxes: net radiation, turbulent fluxes, melt, SWI
├── air_temp.nc …    distributed forcing from SMRF, one file per variable
├── input_backup/    copy of the inputs used
└── logs/            run log
```

## Look at the results

With Jupyter and matplotlib installed (see [Installation](install.md#add-jupyter-optional)),
map the snow depth at the end of the day and plot the basin mean SWE:

```python
import matplotlib.pyplot as plt
import xarray as xr

run = "output/rme/wy1986/rme_test/run19860217_19860217"
snow = xr.open_dataset(f"{run}/snow.nc")

fig, (ax1, ax2) = plt.subplots(1, 2, figsize=(10, 4), constrained_layout=True)
snow["thickness"].isel(time=-1).plot(ax=ax1, cmap="Blues", cbar_kwargs={"label": "Snow depth (m)"})
ax1.set(aspect="equal", xlabel="Easting (m)", ylabel="Northing (m)", title="Snow depth at 23:00")
snow["specific_mass"].mean(["x", "y"]).plot(ax=ax2, color="#2a78d6")
ax2.set(xlabel="", ylabel="SWE (mm)", title="Basin mean SWE")
```

```{figure} images/rme_quickstart.png
:alt: Left, a map of snow depth across the RME grid, from about 0.15 m in the northwest to 0.35 m in the southeast. Right, basin mean SWE rising from 0 to about 56 mm over the day.

Snow depth at the end of the day (left) and basin mean SWE (right). The storm builds
about 56 mm of SWE, with more snow on the southeast side of the basin.
```

## Verify your install (optional)

The AWSM test suite checks the model against reference outputs in `gold/`, made from
the original config (01:00 to 08:00). To reproduce them, run AWSM from Python on a fresh,
unmodified copy of the basin and compare:

```bash
cp -r awsm/awsm/tests/basins/RME rme_verify
cd rme_verify
mkdir output
python -c "from awsm.framework.framework import run_awsm; run_awsm('config.ini')"
```

```python
import numpy as np
import xarray as xr

run = "output/rme/wy1986/rme_test/run19860217_19860217"
for f in ["snow.nc", "em.nc"]:
    gold, test = xr.open_dataset(f"gold/{f}"), xr.open_dataset(f"{run}/{f}")
    bad = [v for v in gold.data_vars if gold[v].ndim == 3
           and not np.allclose(gold[v], test[v], atol=1e-4, equal_nan=True)]
    print(f, "matches gold" if not bad else f"differs: {bad}")
```

Both files should print "matches gold". Unlike the `awsm` command, `run_awsm` runs the exact period in the config.

## Next steps

- Set up [your own basin](https://github.com/iSnobal/model_setup/tree/basin-setup-workflow) (guide in progress).
- Understand the physics behind a single grid cell in [Run Snobal at a point](run_snobal.ipynb).
