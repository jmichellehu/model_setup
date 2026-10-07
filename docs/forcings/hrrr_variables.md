# HRRR variables

`download_hrrr.sh` keeps 12 variables from each HRRR surface file (set by `HRRR_VARS` in the
script). Because cloud cover comes at two levels and the 6-hour forecast holds two
precipitation totals, the files have 13 fields (f01) or 14 fields (f06):

| GRIB name | Level | Variable | Used for |
|---|---|---|---|
| `TMP` | 2 m | Air temperature | Air temperature; Katana wind downscaling |
| `RH` | 2 m | Relative humidity | Vapor pressure (with air temperature) |
| `DPT` | 2 m | Dew point temperature | Not currently read by SMRF or Katana |
| `UGRD`, `VGRD` | 10 m | East and north wind components | Katana wind downscaling; SMRF wind when WindNinja is not used |
| `APCP` | surface | Accumulated precipitation ([see below](#apcp)) | Precipitation |
| `TCDC` | whole atmosphere and boundary layer | Total cloud cover (%) | Katana; optional HRRR cloud factor |
| `DSWRF` | surface | Downward shortwave radiation | Cloud factor, by comparison with modeled clear-sky radiation |
| `VBDSF`, `VDDSF` | surface | Visible direct-beam and diffuse downward solar | Split of solar radiation into beam and diffuse, when HRRR solar is used |
| `DLWRF` | surface | Downward longwave radiation | Incoming thermal radiation, when HRRR thermal is used |
| `HGT` | surface | HRRR model terrain height | Elevation of each HRRR cell, for elevation adjustments of the forcing |

(apcp)=
## Precipitation accumulation periods

`APCP` is precipitation accumulated over a stated window. The 1-hour forecast has one
window; the 6-hour forecast has two, the total since the forecast started and the last hour
alone:

| File | `APCP` fields | Used by SMRF |
|---|---|---|
| f01 (`wrfsfcf01`) | `APCP:surface:0-1 hour acc fcst` | The 1-hour total |
| f06 (`wrfsfcf06`) | `APCP:surface:0-6 hour acc fcst` and `APCP:surface:5-6 hour acc fcst` | Only `5-6 hour`, the precipitation in the last hour; the 6-hour total is not used |

Either way SMRF gets one hour of precipitation for each time step. With
`hrrr_sixth_hour_variables: precip_int` (the template default), precipitation comes from
the f06 files.

## HRRR radiation and cloud

By default, SMRF computes shortwave and longwave radiation from the cloud factor and
terrain. To directly use the HRRR radiation or cloud fields instead, add `hrrr_solar`,
`hrrr_thermal` or `hrrr_cloud` to `variables` in the `[output]` section of the
[configuration](../configuration.md).

<!-- TODO: ^^^ double-check that's how you do it? -->
