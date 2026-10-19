# Katana configuration

Copy [`config/katana.ini`](https://github.com/iSnobal/model_setup/blob/main/config/katana.ini) and edit the paths (`/path/to/...`), dates, and UTM zone:

```ini
[topo]
filename:                  /path/to/topo.nc
zone_number:               13
zone_letter:               N

[time]
start_date:                2017-10-01 00:00 UTC
end_date:                  2017-10-02 00:00 UTC

[input]
data_type:                 hrrr
hrrr_directory:            /path/to/HRRR
hrrr_num_wgrib_threads:    8

[output]
out_location:              /path/to/katana/output
wn_cfg:                    /path/to/katana/output/katana_wind_ninja.ini
make_new_gribs:            true

[logging]
log_level:                 error
log_file:                  /path/to/katana/output/katana.log

[wind_ninja]
mesh_resolution:           200
num_threads:               8
```

`zone_number` is the UTM zone of `topo.nc`: the last two digits of `BASIN_EPSG` in the
`basin.env` file written when you built the [topo.nc](../basin_preparation.md). For example,
EPSG 32613 is zone 13 N. `mesh_resolution` must match `wind_ninja_dxdy` in the AWSM configuration.

## Run time

The WindNinja downscaling is the bottleneck of this workflow. `scripts/katana/katana.slurm` runs a water year month by month on a cluster.

## Check the output

Output goes to one folder per day, `/path/to/katana/output/data<YYYYMMDD>/wind_ninja_data/`. Each
day's folder should be about the same size (`du -hs /path/to/katana/output/data*`). Rerun any day
that is much smaller.