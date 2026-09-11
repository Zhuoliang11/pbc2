# PBC2 non-spatial quantile joint-model analyses

This repository contains the non-spatial simulation study and supplementary PBC2 application for the dissertation. It compares three longitudinal-survival association structures:

1. shared random effects (`shared`);
2. current longitudinal value (`current`);
3. a composite association using current value, slope, and cumulative value (`jifen` or `composite`).

The materials are intended to be understandable without consulting the dissertation. No generated simulation data or fitted result objects are distributed; the scripts create those files when run.

## Contents

```text
jags/                    supplied JAGS model files
simulation/pbc2/         simulation scripts
application/pbc2/nofold/ full-data PBC2 fits
application/pbc2/5_fold/ five-fold prediction analyses
post_processing/pbc2/    result summaries and dissertation validation tables
reference_outputs/       verified aggregate values for result checking
scripts/                  reproducible launcher
data/                     data-access statement
environment/              software-version record
PARAMETER_SETTINGS.md     formal numerical settings
EXPECTED_OUTPUTS.md       output-to-script map
```

## Data

The PBC2 application uses the `pbc2` and `pbc2.id` objects distributed with the R package `JM`. The scripts construct:

- a survival record per participant from `pbc2.id`;
- longitudinal log-serum-bilirubin or log-albumin measurements from `pbc2`;
- standardised age and, for the composite association, centred follow-up time;
- stratified five-fold partitions for the prediction analyses.

No external patient-level data file is stored in this repository. See `data/README.md`.

## Software

R, JAGS, a working C++ compiler, and the packages listed in `environment/README.md` are required. The computational environment used to prepare these materials is recorded there. Because the exact original PBC2 run environment was not saved, the version record should be treated as a compatible reference environment rather than a claim about the historical run.

## Running scripts safely

Run commands from the repository root. The launcher places generated JAGS files and results under `results/`, rather than beside the source code:

```bash
Rscript scripts/run_in_output_dir.R <script-path> <output-directory>
```

Set the number of workers before a run if required. The original scripts read `SLURM_NTASKS`; on a local computer, for example:

```bash
export SLURM_NTASKS=4
```

The formal runs are computationally intensive. Parameter values are documented in `PARAMETER_SETTINGS.md`.

## Reproducing the simulation study

Run these commands independently:

```bash
Rscript scripts/run_in_output_dir.R simulation/pbc2/my_sim_shared.R results/pbc2/simulation/shared
Rscript scripts/run_in_output_dir.R simulation/pbc2/run_sim_all_current.R results/pbc2/simulation/current
Rscript scripts/run_in_output_dir.R simulation/pbc2/jifen_tau_025.R results/pbc2/simulation/jifen
Rscript scripts/run_in_output_dir.R simulation/pbc2/jifen_tau_050.R results/pbc2/simulation/jifen
Rscript scripts/run_in_output_dir.R simulation/pbc2/jifen_tau_075.R results/pbc2/simulation/jifen
```

After all five scripts finish, create the consolidated simulation summaries:

```bash
Rscript -e 'rmarkdown::render("post_processing/pbc2/simulation_summary.Rmd", knit_root_dir=getwd())'
```

The summary document reads `results/pbc2/simulation/{shared,current,jifen}`. It does not contain embedded simulation datasets.

## Reproducing the PBC2 application

### Full-data fits

The files without `foldal` use log-serum-bilirubin; files containing `foldal` use log-albumin.

```bash
Rscript scripts/run_in_output_dir.R application/pbc2/nofold/shared_nofold.r results/pbc2/application/nofold/shared
Rscript scripts/run_in_output_dir.R application/pbc2/nofold/shared_foldal.r results/pbc2/application/nofold/shared
Rscript scripts/run_in_output_dir.R application/pbc2/nofold/current_nofold.r results/pbc2/application/nofold/current
Rscript scripts/run_in_output_dir.R application/pbc2/nofold/current_foldal.r results/pbc2/application/nofold/current
Rscript scripts/run_in_output_dir.R application/pbc2/nofold/jifen_nofold.r results/pbc2/application/nofold/jifen
Rscript scripts/run_in_output_dir.R application/pbc2/nofold/jifen_foldal.r results/pbc2/application/nofold/jifen
```

Then render the full-data summaries:

```bash
Rscript -e 'rmarkdown::render("post_processing/pbc2/nofold_comparison.Rmd", knit_root_dir=getwd())'
```

### Five-fold analyses

Files ending `_al.R` analyse log-albumin; the other files analyse log-serum-bilirubin.

```bash
Rscript scripts/run_in_output_dir.R application/pbc2/5_fold/jm_shared_re_pbc2.R results/pbc2/application/cv5
Rscript scripts/run_in_output_dir.R application/pbc2/5_fold/jm_shared_re_pbc2_al.R results/pbc2/application/cv5
Rscript scripts/run_in_output_dir.R application/pbc2/5_fold/jm_current_re_pbc2.R results/pbc2/application/cv5
Rscript scripts/run_in_output_dir.R application/pbc2/5_fold/jm_current_re_pbc2_al.R results/pbc2/application/cv5
Rscript scripts/run_in_output_dir.R application/pbc2/5_fold/jm_jifen_re_pbc2.R results/pbc2/application/cv5
Rscript scripts/run_in_output_dir.R application/pbc2/5_fold/jm_jifen_re_pbc2_al.R results/pbc2/application/cv5
```

Then render the cross-model validation summaries:

```bash
Rscript -e 'rmarkdown::render("post_processing/pbc2/dissertation_validation.Rmd", knit_root_dir=getwd())'
```

## Interpreting completion

A model run is complete when its expected `.RData` file exists and the script reaches its final success message. The principal object and file names are listed in `EXPECTED_OUTPUTS.md`. Compare the rendered post-processing tables with the corresponding dissertation tables; small Monte Carlo differences can occur if software versions, parallel scheduling, or random-number streams differ.

`RESULTS_MAP.md` links each dissertation table to its generating script and output. The small CSV files under `reference_outputs/` contain only aggregate summaries whose displayed values were checked against the dissertation; they do not contain participant-level records or posterior draws.

## Data and privacy

Generated `.RData`, CSV, HTML, figures, and result directories are ignored by Git. Review any derived outputs before sharing them. This repository contains code only and does not contain the private Brazilian HIV data or any spatial Stan model.
