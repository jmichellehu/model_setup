# Katana configuration

Copy `config/katana.ini` and edit the paths, dates, and UTM zone:

```ini
[topo]
filename:                  /path/to/topo.nc
zone_number:               11
zone_letter:               N

[time]
start_date:                2021-10-01 00:00 UTC
end_date:                  2021-10-02 00:00 UTC

[input]
data_type:                 hrrr
hrrr_directory:            /path/to/HRRR
hrrr_num_wgrib_threads:    8

[output]
out_location:              /path/to/katana
wn_cfg:                    /path/to/katana/katana_wind_ninja.ini
make_new_gribs:            true

[logging]
log_level:                 error
log_file:                  /path/to/katana/katana.log

[wind_ninja]
mesh_resolution:           200
num_threads:               8
```

`zone_number` is the UTM zone of `topo.nc`: the last two digits of `BASIN_EPSG` in the
`basin.env` file written when you [built topo.nc](../basin_preparation.md) (EPSG 32611 is
zone 11 N). `mesh_resolution` must match `wind_ninja_dxdy` in the AWSM configuration.

## Run time

The WindNinja downscaling is the bottleneck. Expect about 20 minutes per month of data on 24 cores, with about 0.5 GB of memory per core. `scripts/katana/katana.slurm` runs a water year month by month on a cluster.
<!-- TODO: check that this estimate is correct, as this will vary by basin size? -->

## Check the output

Output goes to one folder per day, `/path/to/katana/data<YYYYMMDD>/wind_ninja_data/`. Each
day's folder should be about the same size (`du -hs /path/to/katana/data*`). Rerun any day
that is much smaller.
<!-- TODO: this last bit about size checks doesn't make a ton of sense, should add the helper scripts with wind_downscaling_check.sh and wind_incomplete_check.sh to the model_setup repo -->