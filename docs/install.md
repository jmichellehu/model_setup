# Installation

Running iSnobal requires a single `isnobal` conda environment containing AWSM, SMRF, pysnobal and their dependencies. The same environment works on a laptop and on an HPC cluster.

## Requirements

- Linux (see [macOS](#macos) below)
- A conda package manager. We recommend [Miniforge](https://github.com/conda-forge/miniforge)
  (which includes `mamba`).
- `git`

## Create the environment

**Linux**

Clone this repository and create the environment from the Linux environment file:

```{code} bash
:class: shell-prompt
git clone https://github.com/iSnobal/model_setup.git
cd model_setup/
conda env create -f conda/isnobal_linux.yaml
conda activate isnobal
```

The environment pins a specific AWSM release that matches a model release, see [NEWS.md](https://github.com/iSnobal/model_setup/blob/main/NEWS.md) for release history.

## Check the install

```{code} bash
:class: shell-prompt
awsm --help
```

If the install worked, a usage message should pop up.
```text
usage: awsm [-h] [--version] -c CONFIG_FILE [-sd START_DATE] [-np] [-t]
            [-mt MEDIUM_THRESHOLD]

Run AWSM with given config file.

optional arguments:
  -h, --help            show this help message and exit
  --version             show program's version number and exit
  -c CONFIG_FILE, --config_file CONFIG_FILE
                        Path to .ini config file.
  -sd START_DATE, --start_date START_DATE
                        Overwrite the start date in the .ini and force a single day run.
                            Format: YYYYMMDD or YYYY-MM-DD
  -np, --no_previous    Skip finding a previous snow state and storm day file.
                            Usually used when running the first day.
  -t, --threshold       Run iSnobal with different mass threshold
  -mt MEDIUM_THRESHOLD, --medium_threshold MEDIUM_THRESHOLD
                        Set the medium mass threshold. Default: 25
```

## Add Jupyter (optional)

Jupyter and matplotlib are needed to run the plotting examples in the [point Snobal notebook](run_snobal.ipynb). These are not part of the model environment and need to be installed separately:

```{code} bash
:class: shell-prompt
conda install -n isnobal -c conda-forge jupyterlab matplotlib
```

## Running on macOS

A macOS environment is available for Intel and Apple Silicon machines, intended
mainly for point runs with Snobal or small model domains with iSnobal. See the
[macOS section](https://github.com/iSnobal/model_setup/blob/main/conda/README.md#for-macos) in the conda README for the install script.

## Developer install

To edit the model source code, see the [development setup section](https://github.com/iSnobal/model_setup/blob/main/conda/README.md#model-development-environment) in the
conda README.
