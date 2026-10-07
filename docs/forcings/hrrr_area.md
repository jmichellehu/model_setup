# Change the HRRR download area

Each file is cropped to a longitude and latitude box set by `GRIB_AREA` near the top of
`download_hrrr.sh`. The default covers the western US:

```bash
# Western United States from Denver West
export GRIB_AREA="-122.00:-105.00 32.00:49.00"
```

The format is `"WEST:EAST SOUTH:NORTH"` in decimal degrees, with negative west longitudes. `GRIB_AREA` is passed to `wgrib2 -small_grib`.

A smaller area will require less download space. To fit your
basin, you can use `BASIN_BBOX` in the `basin.env` file written when you
[built topo.nc](../basin_preparation.md), which is ordered `west,south,east,north`, and add padding to every side. This margin should be large enough that SMRF and Katana have
surrounding HRRR cells to interpolate from at the basin edges. A box fitted to a single
basin can make each file much smaller than the western US default.
<!-- TODO add specifics about what margin is "large enough" -->

```{warning}
The script skips files that already exist on disk. After changing `GRIB_AREA`, download into
a new folder; files already downloaded keep their old extent.
```
