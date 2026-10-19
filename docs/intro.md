# Overview
This guide details installation, initial model runs, setting up your own basin and working with model outputs.

## Modeling background

**iSnobal** is a physically based, spatially distributed snow model. It solves the energy and mass balance of the snowpack on every cell of a grid (**i** stands for image), producing maps of snow depth, snow water equivalent (SWE), melt, and the energy fluxes that drive them{cite:p}`marks1999`.

The iSnobal community model is a stack of Python packages, maintained under the
[Community iSnobal GitHub organization](https://github.com/iSnobal):

```{mermaid}
flowchart LR
    A(Station or gridded weather data) --> B["`**SMRF**`"<br/>distribute forcing]
    T("`**topo.nc**`"<br/>DEM, mask, vegetation) --> B
    B --> C["`**iSnobal**`"<br/>snow energy balance per cell]
    T --> C
    C --> D("`**model outputs**`"<br/>e.g., snow.nc, em.nc)
    subgraph "`**AWSM**`"
      B
      C
    end
```

- **SMRF** (Spatial Modeling for Resources Framework){cite:p}`havens2017` transforms input weather data into gridded,
  hourly [forcings](forcings.md) based on model domain information provided by the `topo.nc`. These domain-distributed forcings are then fed to iSnobal.
-  At the core of **iSnobal** is the one-dimensional, two-layer snowpack model: **Snobal**{cite:p}`marks1992`. iSnobal runs Snobal on every cell of the model domain grid without any interaction between neighboring cells. Both the 1D and 2D models are wrapped for Python by
  [pysnobal](https://github.com/iSnobal/pysnobal).
- **AWSM** (Automated Water Supply Model){cite:p}`havens2020` runs SMRF and iSnobal together from a single
   [configuration file](configuration.md) (`.ini`). The `awsm` command is used for
  distributed runs.

## Get started
| If you want to… | Go to |
|---|---|
| Install the model | [Installation](install.md) |
| Understand the physics at a single point, in a notebook | [Run Snobal at a point](run_snobal.ipynb) |
| Run a small basin, end to end | [Run the RME test basin](run_rme.md) |
