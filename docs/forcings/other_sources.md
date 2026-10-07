# Other distributed forcing sources

SMRF can also build forcings from sources besides HRRR.

## Weather stations

Supply hourly CSV files, one per variable with one column per station, plus a station
metadata file. SMRF interpolates between stations. The [RME test basin](../run_rme.md) uses
this setup; see its `config.ini` for a complete example:

```ini
[csv]
metadata:        ./station_data/metadata.csv
air_temp:        ./station_data/air_temp.csv
vapor_pressure:  ./station_data/vapor_pressure.csv
precip:          ./station_data/precip.csv
wind_speed:      ./station_data/wind_speed.csv
wind_direction:  ./station_data/wind_direction.csv
cloud_factor:    ./station_data/cloud_factor.csv
```

## WRF or other gridded netCDF
These are currently experimental.
<!-- TODO: CONUS404 and maybe AORC for CIROH? Joe is working on CONUS404 -->

Set `data_type: wrf` with `wrf_file`, or `data_type: netcdf` with `netcdf_file`, in the
`[gridded]` section.
