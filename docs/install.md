<!-- TODO: model_setup/conda/README.md - migrate information -->
Running iSnobal requires a single `isnobal` conda environment containing AWSM, SMRF, pysnobal and their dependencies. The same environment works on a laptop and on an HPC cluster.

## Requirements

- Linux (see [macOS](#macos) below)
- A conda package manager. We recommend [Miniforge](https://github.com/conda-forge/miniforge)
  (which includes `mamba`).
- `git`

## Create the environment

Clone this repository and create the environment from the Linux environment file:

```bash
git clone https://github.com/iSnobal/model_setup.git
cd model_setup
conda env create -f conda/isnobal_linux.yaml
conda activate isnobal
```

The environment pins a specific AWSM release that matches a model release; see [NEWS.md](https://github.com/iSnobal/model_setup/blob/main/NEWS.md).

## Check the install

```bash
awsm --help
```

A usage message listing the `-c/--config_file` option means the install worked.

## Add Jupyter (optional)

The [point Snobal notebook](run_snobal.ipynb) and the plotting examples need Jupyter and
matplotlib, which are not part of the model environment:

```bash
conda install -n isnobal -c conda-forge jupyterlab matplotlib
```

<!-- ## Laptop or HPC?

| | Laptop | HPC cluster |
|---|---|---|
| Good for | Point runs, test basins, small domains or short periods | Full basins and water years |
| Install | As above | As above, in your home or project space |
| Running | Directly from a terminal | Inside a batch job (e.g., SLURM); set threads with `ithreads` in the `[awsm system]` section of the `.ini` |

Start on a laptop with the [RME test basin](run_rme.md). It runs in seconds. -->

(macos)=
## macOS

A macOS environment is available for Intel and Apple Silicon machines, intended mainly
for point runs and small domains. See the
[conda README](https://github.com/iSnobal/model_setup/blob/main/conda/README.md#for-macos)
for the install script. This guide is tested on Linux only.

## Developer install

To edit the model source code, see the development setup in the
[conda README](https://github.com/iSnobal/model_setup/blob/main/conda/README.md#model-development-environment).
