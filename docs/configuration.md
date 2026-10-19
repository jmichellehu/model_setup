# Model configuration and runs

One `.ini` file controls an AWSM run and dictates input location, run period, SMRF distribution rules for each forcing variable, and output location.

Start from the template file, [`config/awsm.ini`](https://github.com/iSnobal/model_setup/blob/main/config/awsm.ini),
which is set up for HRRR forcing with Katana winds (see [Forcing data](forcings.md)).

```{code} bash
:class: shell-prompt
cp config/awsm.ini my_basin.ini
```

## Required user input

Lines marked `** REPLACE ME **` or `** UPDATE **` in the template (mostly `/path/to/...`) must be changed. The
table lists everything that is specific to your basin or run:

| Section | Item | Set to |
|---|---|---|
| `[topo]` | `filename` | Your `topo.nc` ([Prepare your basin](basin_preparation.md)) |
| `[time]` | `start_date`, `end_date` | Period to run in UTC, begin at 00:00 |
| `[gridded]` | `hrrr_directory` | Folder containing the `hrrr.YYYYMMDD/` folders |
| `[wind]` | `wind_ninja_dir` | Katana output folder |
| `[albedo]` | `decay_start`, `decay_end` | Spring albedo decay window, in the year you run |
| `[paths]` | `path_dr` | Existing parent folder where outputs are written |
| `[paths]` | `basin`, `project_name` | Names used to build the output folder path |
| `[awsm system]` | `ithreads` | Number of CPU cores available |

The rest of the template holds the maintainers' recommended settings. These may be tuned as desired.

```{important}
`<path_dr>` must exist before the run starts! AWSM will only create the following:
`/<path_dr>/<basin>/` and `/<path_dr>/<basin>/<project_name>`
```

## Section overview

| Sections | Controls |
|---|---|
| `[topo]`, `[time]` | Model domain grid and period |
| `[gridded]` or `[csv]` | Forcing data source |
| `[air_temp]`, `[vapor_pressure]`, `[wind]`, `[precip]`, `[albedo]`, `[cloud_factor]`, `[solar]`, `[thermal]`, `[soil_temp]` | How SMRF builds each forcing variable |
| `[output]` | Which SMRF forcing grids to save to file |
| `[awsm master]`, `[awsm system]`, `[system]` | What to run (SMRF, iSnobal), threads, logging |
| `[paths]` | Output folder naming |
| `[grid]`, `[files]`, `[ipysnobal constants]` | iSnobal settings: time-step thresholds, initial state, measurement heights |

## Defaults, checks, and ini explanations
AWSM uses [inicheck](https://pypi.org/project/inicheck/) to read the file. For any
item you leave out, inicheck fills in a default, and it rejects items or values it does
not recognize. Check a file before running, and write out the complete configuration with
every default filled in:

```{code} bash
:class: shell-prompt
conda activate isnobal
inicheck -f my_basin.ini -m smrf awsm        # check only
inicheck -f my_basin.ini -m smrf awsm -w     # also write my_basin_full.ini
inicheck -f my_basin.ini -m smrf awsm -d wind distribution   # explain one item
```

```{warning}
- **Keep every section header**
    - Defaults are only filled in for sections present in the
  file. A missing section can crash the run, so keep all headers from the template even if
  a section is empty.
- **Some defaults turn things off**
    - `[awsm master]` needs `run_smrf: True` and `model_type: ipysnobal`. Without these settings,
    AWSM finishes without running anything.
- **`project_description` is required** in practice, even though it has no default and is not a part of the output path
- **`output_frequency` defaults to 24 hours**
    - `[awsm system]` `output_frequency` is not set in the template, so it falls back to inicheck's default of 24.
    - Note that only 00:00 and 23:00 are saved to `snow.nc`/`em.nc` each day (when saved, see [Model outputs](model_outputs.md#forcing-grids)).
    - To get hourly output, set `output_frequency: 1`.
```

## Running AWSM
AWSM runs one day at a time and starts each day from the previous day's snow state. If you are
running from the first day, there is no previous state, so passing `--no_previous` is required.
This runs every day in the config from `start_date` to `end_date`, 24 hours at a time.

```{code} bash
:class: shell-prompt
conda activate isnobal
awsm -c my_basin.ini --no_previous
```
Two other forms may also be useful. To run a single day, add the start_date (`-sd`) flag:
```{code} bash
:class: shell-prompt
awsm -c my_basin.ini -sd 2021-10-01 -np  # one day, starting with no snow
awsm -c my_basin.ini -sd 2021-11-15      # one day, starting from the day before's output
```

With `folder_date_style: day`, each day's output goes to
`<path_dr>/<basin>/wy<year>/<project_name>/run<YYYYMMDD>/`. See
[Model outputs](model_outputs.md) for the contents of each run day folder.

```{tip}
On an HPC cluster, run AWSM inside a batch job;
[`scripts/iSnobal/iSnobal.slurm`](https://github.com/iSnobal/model_setup/blob/main/scripts/iSnobal/iSnobal.slurm)
is a SLURM template. Set `ithreads` to the number of cores you request.
```
