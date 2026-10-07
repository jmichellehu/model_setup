# Model outputs

Each AWSM run writes one folder per day, under
`<path_dr>/<basin>/wy<year>/<project_name>/run<YYYYMMDD>/` (see
[Configuration](configuration.md)). All grids are netCDF on the `topo.nc` grid, with
dimensions `(time, y, x)` and hourly time steps in UTC.

```text
run<YYYYMMDD>/
├── snow.nc              iSnobal snow state
├── em.nc                iSnobal energy and mass fluxes
├── air_temp.nc …        SMRF forcing grids, one file per variable in [output] variables
├── storm_days.nc        days since last storm, used to start the next day
├── config.ini           configuration used for this day
├── awsm_config_backup.ini
├── input_backup/        copy of the input files (if input_backup: True)
└── logs/                run log
```

## Snow state variables within `snow.nc`

Values at the end of each hour.

| Variable | Units | Description |
|---|---|---|
| `thickness` | m | Snow depth |
| `specific_mass` | kg m⁻² | Snow water equivalent (SWE); 1 kg m⁻² = 1 mm |
| `snow_density` | kg m⁻³ | Mean snowpack density |
| `liquid_water` | kg m⁻² | Liquid water held in the snowpack |
| `water_saturation` | % | Liquid water as a percentage of what the pack can hold |
| `temp_surf` | °C | Active (surface) layer temperature |
| `temp_lower` | °C | Lower layer temperature |
| `temp_snowcover` | °C | Mean snowpack temperature |
| `thickness_lower` | m | Lower layer thickness |

## Energy and mass flux variables within `em.nc`

Energy terms are averages over the hour; mass terms are totals for the hour. Energy fluxes into the snowpack are positive in sign.

| Variable | Units | Description |
|---|---|---|
| `net_rad` | W m⁻² | Net all-wave radiation |
| `sensible_heat` | W m⁻² | Sensible heat flux |
| `latent_heat` | W m⁻² | Latent heat flux |
| `snow_soil` | W m⁻² | Ground heat flux |
| `precip_advected` | W m⁻² | Heat advected by precipitation |
| `sum_EB` | W m⁻² | Sum of the energy balance terms |
| `cold_content` | J m⁻² | Energy needed to bring the pack to 0 °C |
| `evaporation` | kg m⁻² | Evaporation and sublimation |
| `snowmelt` | kg m⁻² | Melt |
| `SWI` | kg m⁻² | Surface water input: water leaving the bottom of the pack (melt plus rain) |

## Forcing grids

The SMRF forcing grids, such as `air_temp.nc`, `precip.nc` and `net_solar.nc`, are the
hourly inputs iSnobal received. Which ones are saved is set by `variables` in `[output]`.

```{note}
The forcing files include the first hour of the day (00:00). Whether `snow.nc` and `em.nc`
do too depends on how the day starts:
- **First day of a run started with `--no_previous`:** there is no prior snow state, so 00:00
  is used only to initialize the model and is not saved. The first saved step is 01:00.
- **Any other day** (continuing a multi-day run, or a single day initialized from the
  previous day's output): the 00:00 step runs from the previous day's state and is saved.

The last hour of every day (23:00) is always saved. See also `output_frequency` in
[Configuration](configuration.md#defaults-and-checks), which controls how many hours in
between are kept.
```

## Open one day

```python
import xarray as xr

run = "output/rme/wy1986/rme_test/run19860217_19860217"
snow = xr.open_dataset(f"{run}/snow.nc")
em = xr.open_dataset(f"{run}/em.nc")

snow["specific_mass"].isel(time=-1).plot()   # SWE map at the last hour
```

```{warning}
In current AWSM outputs, the `long_name` attributes of `x` and `y` in `snow.nc` and
`em.nc` are swapped. The coordinate values are correct; set axis labels yourself when
plotting.
```
