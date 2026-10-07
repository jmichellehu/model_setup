# Quickstart

iSnobal is a physically based, spatially distributed snow model.
It solves the energy and mass balance of the snowpack on every cell of a grid (hence, *i* for image), producing maps of snow depth, snow water equivalent (SWE), melt, and the energy fluxes that drive them.

This guide details installation, initial model runs, setting up your own basin (the all-important topo.nc file) and working with model outputs.

## Overview

The iSnobal community model is a stack of Python packages, maintained under the
[Community iSnobal GitHub organization](https://github.com/iSnobal):

```{mermaid}
flowchart LR
    A(Station or gridded weather data) --> B["`**SMRF**`"<br/>distribute forcing]
    T("`**topo.nc**`"<br/>DEM, mask, vegetation) --> B
    B --> C["`**iSnobal**`"<br/>snow energy balance per cell]
    T --> C
    C --> D(snow.nc, em.nc<br/>model outputs)
    subgraph "`**AWSM**`"
      B
      C
    end
```

- **Snobal** is the one-dimensional, two-layer snowpack model at the core
  {cite}`marks1992`. **iSnobal** runs Snobal on every cell of a grid
  {cite}`marks1999`. Both are wrapped for Python by
  [pysnobal](https://github.com/iSnobal/pysnobal).
- **SMRF** (Spatial Modeling for Resources Framework) transforms input weather data into gridded,
  hourly [forcings](forcings.md) {cite}`havens2017` based on model domain information in the `topo.nc`.
- **AWSM** (Automated Water Supply Model) runs SMRF and iSnobal together from a single
  `.ini` [configuration file](configuration.md) {cite}`havens2020`. The `awsm` command is used for
  distributed runs.

## Next steps

| If you want to… | Go to |
|---|---|
| Install the model | [Installation](install.md) |
| Understand the physics at a single point, in a notebook | [Run Snobal at a point](run_snobal.ipynb) |
| Run a small basin, end to end | [Run the RME test basin](run_rme.md) |
