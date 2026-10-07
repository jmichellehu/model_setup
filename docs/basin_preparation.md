# Prepare your basin

Every iSnobal run starts from a `topo.nc` file that defines the model grid: elevation, the
basin mask, and vegetation properties. This page builds the `topo.nc` for any US watershed with the scripts in
[`scripts/topo`](https://github.com/iSnobal/model_setup/tree/main/scripts/topo).

## `topo.nc` contents

| Variable | Contents |
|---|---|
| `dem` | Elevation (m), from USGS 3DEP |
| `mask` | 1 inside the basin, 0 outside, from the USGS Watershed Boundary Dataset |
| `veg_type` | LANDFIRE 1.4.0 existing vegetation type class |
| `veg_height` | Canopy height (m) |
| `veg_tau` | Canopy transmissivity (0–1) |
| `veg_k` | Canopy extinction coefficient |

SMRF uses elevation to distribute weather across the terrain and the vegetation layers
to adjust wind and radiation under canopy.

The grid is projected to the basin's UTM zone, 100 m by default.

## Set up

The topo scripts use their own conda environment, `basin_setup`, separate from `isnobal`:

```bash
cd model_setup
conda env create -f conda/basin_setup.yaml
conda activate basin_setup
cd scripts/topo
```

### Get the vegetation data

While basin boundary and elevation data are downloaded semi-automatically,  vegetation layers come from
LANDFIRE 1.4.0. As this version is no longer distributed by LANDFIRE, the iSnobal maintainers
host a copy:

- **Dataset:** LANDFIRE 1.4.0 EVT and EVH for CONUS plus the vegetation parameter table,
  on Zenodo (DOI: *to be added*). About 6 GB zipped.

Download and unzip it into one folder, keeping this layout:

```text
<landfire-dir>/
├── US_140EVT_20180618/Grid/us_140evt/
├── US_140EVH_20180618/Grid/us_140evh/
└── landfire_veg_params_assigned.csv
```

```{tip}
**CHPC users (University of Utah):** the data are already staged at
`/uufs/chpc.utah.edu/common/home/skiles-group3/LANDFIRE/`, which is the scripts' default.
You can leave out `--landfire-dir` and `--veg-params-csv` below.
```

## Build `topo.nc`

`generate_topo.py` runs the whole pipeline. Choose the basin by name, by HUC ID, or with your
own boundary polygon:

```bash
# By name: prints a table of candidates if more than one basin matches
python generate_topo.py -n "reynolds creek" -level 10 -o ./reynolds_creek \
    --landfire-dir /path/to/landfire \
    --veg-params-csv /path/to/landfire/landfire_veg_params_assigned.csv

# By HUC ID (2, 4, 6, 8, 10, or 12 digits, including leading zeros)
python generate_topo.py -huc 1705010306 -o ./reynolds_creek \
    --landfire-dir /path/to/landfire \
    --veg-params-csv /path/to/landfire/landfire_veg_params_assigned.csv

# From your own boundary (shapefile or GeoPackage, in UTM)
python generate_topo.py -s /path/to/basin.gpkg -o ./my_basin \
    --landfire-dir /path/to/landfire \
    --veg-params-csv /path/to/landfire/landfire_veg_params_assigned.csv
```

A name search defaults to HUC 8 basins; use `-level` to search other sizes. When several
basins match, the script prints their HUC IDs and stops. Rerun with `-huc`.

The HUC 10 Reynolds Creek example (about 360 km²) builds in under a minute:

```text
[1/3] Fetching basin boundary...
  EPSG      : 32611
[2/3] Building DEM...
  Warping to EPSG:32611 at 100m...
[3/3] Building topo.nc...
  Projection OK, EPSG:32611
Done! topo.nc at ./reynolds_creek/output_100m/topo.nc
```

### Useful options

| Option | Effect |
|---|---|
| `-res METERS` | Grid resolution (default 100). Output goes to `output_<res>m/` |
| `-e EPSG` | Override the auto-detected UTM zone |
| `--download-dem-tiles` | Save DEM tiles to disk before warping, instead of streaming them. Use on compute nodes without reliable internet |
| `--veg-dir DIR` | Use your own `veg_type.tif`, `veg_height.tif`, `veg_tau.tif`, `veg_k.tif` instead of LANDFIRE (**experimental**) |

Run `python generate_topo.py -h` for the full list. Each step can also be run on its own
(`fetch_basin.py`, `fetch_dem.py`, `build_topo_nc.py`); see the
[scripts README](https://github.com/iSnobal/model_setup/blob/main/scripts/topo/README.md).

## Check the result

The output folder holds the boundary, the DEM, a `basin.env` file recording the paths and
UTM zone, and `output_<res>m/topo.nc`. Plot it to check that the mask and layers line up:

```python
import matplotlib.pyplot as plt
import xarray as xr

topo = xr.open_dataset("reynolds_creek/output_100m/topo.nc")

fig, (ax1, ax2) = plt.subplots(1, 2, figsize=(9, 6), constrained_layout=True)
topo["dem"].plot(ax=ax1, cmap="terrain", cbar_kwargs={"label": "Elevation (m)"})
topo["veg_height"].plot(ax=ax2, cmap="Greens", cbar_kwargs={"label": "Vegetation height (m)"})
for ax, title in [(ax1, "dem"), (ax2, "veg_height")]:
    topo["mask"].plot.contour(ax=ax, levels=[0.5], colors="k", linewidths=1)
    ax.set(aspect="equal", xlabel="Easting (m)", ylabel="Northing (m)", title=title)
```

```{figure} images/topo_reynolds_creek.png
:alt: Two maps of the Reynolds Creek HUC 10 grid with the basin outline in black. Left, elevation from about 700 m in the north to 2200 m in the southwest. Right, vegetation height, mostly low shrubs with taller canopy in the southern uplands.

Elevation (left) and vegetation height (right) for Reynolds Creek, Idaho, with the basin
mask outlined. The grid extends past the basin so that terrain around it is available.
```

## Next step

With `topo.nc` in hand, prepare the weather data that drives the model:
[Forcing data](forcings.md).
