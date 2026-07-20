# BCCHR IBD Public Demo

This repository is a **public demo copy** of the original project.

## Important note

The data used in this demo is **generated fake data**. Any files produced from it, including:

- dashboard outputs
- figures
- statistical results
- rendered Quarto pages

are **not real patient data** and should not be interpreted as real clinical findings.

## What was removed

- private raw, intermediate, and processed data
- HANDOVER.md
- live credentials
- deployment/account metadata
- private generated outputs
- local machine/cache artifacts

# BCCHR_IBD_Capstone

2026 MDS Capstone project in partnership with BC Children's Hospital Research Institute. Team: Tiffany Chu, Victoria Farkas, Ian Gault, Derrick Jaskiel

## Overview

This repository supports the BCCHR IBD capstone project by turning raw clinical, dietary, biomarker, and mycobiome data into reproducible data products: cleaned processed files, exploratory visualizations, Quarto-based statistical reports, and a Shiny dashboard for participant- and cohort-level exploration.

Domain-specific notes live in the relevant sub-folders under `src/`.

## Structure

**Main Analysis Pipeline**

- `data/raw`: raw input files
- `data/intermediate`: pipeline intermediates
- `data/processed`: processed outputs
- `figures/`: generated figures from main pipeline and exploratory data analysis
- `src/mycobiome`: main omics pipeline
- `src/characteristics`, `src/diet`, `src/merge_files.R`: data cleaning and merge utilities

**Shiny Dashboard** (`dashboard/`)

- `dashboard/ui.R`, `dashboard/server.R`, `dashboard/global.R`: app entry point (legacy Shiny three-file layout - this is what Shiny actually loads, not `app.R`, which doesn't exist in this project)
- `dashboard/R/`: dashboard-specific helper modules (`auth.R` - shinymanager login credentials; `cards.R`, `plots.R`, `individual_*.R`, `population_*.R` - tab logic)
- `dashboard/data/`: **not committed** - copied from `data/processed/` and `data/intermediate/` automatically by `make app` or `make deploy`, so the dashboard folder is self-contained for deployment

**Quarto Statistical Analysis Site** (`stats/`)

- `stats/posts/`: one folder per analysis, each containing an `index.qmd`
- `stats/figures/`: figures saved by R chunks within stats posts (written via `here("figures", ...)`, anchored to `stats/` by `stats/.here`)
- `stats/R/`: shared R helper scripts sourced by posts
- `stats/_freeze/`: frozen execution results (committed; enables fast rebuilds)
- `stats/_site/`: rendered HTML output (not committed)

**Structural Diagrams**

![Pipeline overview](figures/diagrams/pipeline_overview.svg)\
_Figure 1: Pipeline overview_

![Data linkage diagram](figures/diagrams/data_linkage.svg)\
_Figure 2: Data linkage diagram_

## Raw data requirements {#raw-data-requirements}

Raw files are **not** committed to git (see `.gitignore`). After cloning, place every required source file in `data/raw/` using the **exact filenames** below.

| Domain                      | Filename                                  | Used by                |
| --------------------------- | ----------------------------------------- | ---------------------- |
| Mycobiome metadata          | `OPT_MBI sample IDs meta.xlsx`            | `make mycobiome`       |
| Mycobiome taxa              | `OPT_stool mycobiota relative abund.xlsx` | `make mycobiome`       |
| Inflammatory biomarkers     | `OPT_Inflammatory biomarkers.xlsx`        | `make mycobiome`       |
| Dietary intake              | `OPT_dietary data.xlsx`                   | `make dietary`         |
| Participant characteristics | `OPT_Participant Characteristics.xlsx`    | `make characteristics` |

Without these files, the processing pipelines and Quarto stats posts cannot run.

## Prerequisites

Install these **before** running any commands (all commands assume you are at the repository root):

| Tool                                                                                                        | Version / notes                                    |
| ----------------------------------------------------------------------------------------------------------- | -------------------------------------------------- |
| [git bash](https://git-scm.com/install/windows)                                                             | download the latest (2.54.0 or higher) x64 version |
| [R](https://cran.r-project.org/)                                                                            | **4.6.0** (pinned in `renv.lock`)                  |
| [Quarto](https://quarto.org/docs/get-started/)                                                              | CLI required for `make stats` and `make stats-*`   |
| [GNU Make](https://sourceforge.net/projects/ezwinports/files/make-4.4.1-without-guile-w32-bin.zip/download) | Provides the `make` targets documented below       |
| [VS Code](https://code.visualstudio.com/download)                                                           | Default code editor for git bash setup             |

> **Before running `make setup`:** Windows needs Rtools, and macOS needs Homebrew + OpenSSL, so that R can compile packages that don't yet have a prebuilt binary for R 4.6.0. See REDACTED and install those first, or `make setup` may fail partway through with packages you'd otherwise have to install by hand.

## Quick start

Run commands from the repository root. We recommend using `git bash`. Alternatives include the terminal in `RStudio`(located in the top right beside console or bottom half of screen), or the terminal in `VSCode` (launch using `ctrl + ~`).

To get the most recent updates/changes on your local machine:

```sh
git pull
```

To see the available commands:

```sh
make help
```

Install the project R environment (see the compiler prerequisites note above first - Rtools on Windows, Homebrew + OpenSSL on macOS)

```sh
make setup
```

Run the mycobiome processing pipeline

```sh
make mycobiome
```

Run the dietary cleaning and dietary EDA pipeline

```sh
make dietary
```

Run the participant characteristics cleaning script (R)

```sh
make characteristics
```

Build merged analysis files

```sh
make merge
```

**Launch the dashboard**

You may have to `Ctrl + -` (Control + Minus) the dashboard in your browser if the cards are being cut off at the borders

```sh
make app
```

The dashboard is gated behind a login screen ([shinymanager](https://datastorm-open.github.io/shinymanager/)):

Credentials are defined in `dashboard/R/auth.R`. Documented here in plaintext since this repository is private — do not move this dashboard, its credentials, or this README section to a public repository without changing the password first.

`make dashboard-data` is not part of the `make help` list of commands, but is incorporated in both `make app` and `make deploy` commands described below. This command copies the required processed data into `dashboard/data` so that the `dashboard` can be bundled easily for deployment to the Posit Connect Cloud account.

`make app` automatically refreshes `dashboard/data/` from the latest processed pipeline output before launching. To deploy the dashboard to Posit Connect Cloud, first run `make setup` with R 4.6.0 so the project packages are installed, then link your account once with `Rscript -e 'rsconnect::connectCloudUser(launch.browser = TRUE)'`, and finally run `make deploy`. See REDACTED for the one-time account setup details.

**Build the statistical analysis website**

```sh
make stats
```

Format and lint the R code

```sh
make r-check
```

Run unit tests for stats helper functions (`tests/testthat/`)

```sh
make test
```

On success you should see a summary like `[ FAIL 0 | WARN 0 | SKIP 0 | PASS 28 ]` followed by `Unit tests complete.` Any non-zero **FAIL** count means a test failed and `make` will exit with an error. The first run may install `testthat` via renv and print a dependency-discovery note from renv - that is normal and can be ignored.

Remove generated processed files, figures, and site output

```sh
make clean
```

### Alternative to make

If the above commands fail, or `make` is not downloaded, try these commands in bash:

```sh
# instead of make setup
Rscript -e "options(repos=c(CRAN='https://cran.rstudio.com/')); if (!requireNamespace('renv', quietly=TRUE)) install.packages('renv'); renv::restore(prompt = FALSE)"

# instead of make mycobiome
Rscript src/mycobiome/01_data_import.R
Rscript src/mycobiome/02_data_wrangling.R
Rscript src/mycobiome/04_abundance_analysis.R
Rscript src/mycobiome/05_heatmap_analysis.R
Rscript src/mycobiome/06_save_processed.R

# instead of make dietary
Rscript src/diet/dietary.R
Rscript src/diet/01_prepare_eda_data.R
Rscript src/diet/02_correlation_heatmap.R
Rscript src/diet/03_scaled_boxplots.R
Rscript src/diet/04_group_kde_plots.R
Rscript src/diet/05_deficiency_barplot.R

# instead of make characteristics
Rscript src/characteristics/characteristics.R

# instead of make merge
Rscript src/merge_files.R

# instead of make stats
quarto render stats --execute --cache-refresh

# instead of make app
Rscript -e "shiny::runApp('dashboard', launch.browser = TRUE)"

# instead of make deploy (requires `make setup` to have completed successfully with R 4.6.0 so `rsconnect` is installed, and `rsconnect::connectCloudUser()` to have been run once already)
Rscript -e "dir.create('dashboard/data', recursive = TRUE, showWarnings = FALSE); file.copy('data/processed/merged.csv', 'dashboard/data/merged.csv', overwrite = TRUE); file.copy('data/processed/participants_all.csv', 'dashboard/data/participants_all.csv', overwrite = TRUE); file.copy('data/intermediate/inflammatory_markers.rds', 'dashboard/data/inflammatory_markers.rds', overwrite = TRUE)"
Rscript -e "rsconnect::deployApp('dashboard', appName = 'bcchr-ibd-dashboard', appTitle = 'BCCHR IBD Dashboard', launch.browser = TRUE)"
```

## Statistical analysis site (Quarto blog)

The `stats/` directory is a Quarto website that documents every pre-specified and exploratory analysis: methods, tabulated results, interpretation, and figures. Follow the steps below from a fresh clone to reproduce all stats tests and render the blog locally.

All data pipelines (mycobiome, dietary, and characteristics) run in R via `make setup` and the locked `renv` library.

### Step 1 - Clone the repository

```sh
git clone https://github.com/tiffchu/BC_Childrens_Research_Institute_IBD_demo
cd BCCHR_IBD_Capstone
```

### Step 2 - Upload raw data

Copy the source files listed in [Raw data requirements](#raw-data-requirements) into `data/raw/`. Filenames must match exactly.

### Step 3 - Set up the R environment (renv)

The project uses [renv](https://rstudio.github.io/renv/) to lock R package versions. Restore the library from `renv.lock`:

```sh
make setup
```

This installs `renv` if needed and runs `renv::restore()` to match the locked package versions.

**Automatic activation:** Opening R or RStudio/Positron in the repo root sources `.Rprofile`, which activates renv. The stats site has its own `stats/.Rprofile` that sources the parent `renv/activate.R` when Quarto executes R chunks.

Useful renv maintenance commands:

```sh
make status      # check whether renv.lock and the local library are in sync
make snapshot    # update renv.lock after adding or upgrading packages
make renv-clean  # interactively remove packages not used by the project
```

### Step 4 - Run data pipelines (required before stats)

Stats posts read from `data/intermediate/` and `data/processed/`. Run the shared pipelines in order:

```sh
make mycobiome        # import, wrangle, heatmaps -> alpha_long.rds, genus.csv, etc.
make dietary          # clean diet data -> dietary_cleaned.xlsx + figures/diet/*
make characteristics  # R script: clean survey data -> cleaned_characteristics.csv
make merge            # merges files together on shared IDs to create subcohorts
```

`make stats` checks for `data/intermediate/alpha_long.rds` and exits with an error if the mycobiome pipeline has not been run.

**Outputs consumed by stats posts:**

| File                                                    | Produced by                         |
| ------------------------------------------------------- | ----------------------------------- |
| `data/intermediate/alpha_long.rds`                      | `make mycobiome`                    |
| `data/intermediate/taxa_long_list.rds`, `meta_data.rds` | `make mycobiome`                    |
| `data/processed/{phylum,family,genus,species}.csv`      | `make mycobiome`                    |
| `data/processed/dietary_cleaned.xlsx`                   | `make dietary`                      |
| `data/processed/cleaned_characteristics.csv`            | `make characteristics`              |
| `figures/mycobiome/heat_*.png`                          | `make mycobiome` (EDA heatmap post) |
| `figures/diet/*.png`                                    | `make dietary` (dietary EDA post)   |

### Step 5 - Build the stats site

Render the full Quarto website (all posts, navbar pages, and listings):

```sh
make stats
```

Output is written to `stats/_site/index.html`.

**Freeze behavior:** `_quarto.yml` sets `execute: freeze: auto`. Committed results in `stats/_freeze/` let `make stats` rebuild HTML quickly without re-running every R chunk. To **recompute** a test from fresh processed data, use the per-analysis `make stats-*` targets below (they pass `--no-freeze` where needed), then run `make stats` again to refresh the full site.

### Step 6 - Launch the blog locally

After `make stats`, the Makefile attempts to open the rendered site automatically via R. If you need to open it manually, use one of these:

```sh
open stats/_site/index.html          # macOS
xdg-open stats/_site/index.html      # Linux
start stats/_site/index.html         # Windows (Git Bash)
```

For live reload while editing `.qmd` files:

```sh
quarto preview stats
```

The preview server prints a local URL (typically `http://localhost:XXXX`).

### Step 7 - Re-run individual statistical tests

Use these targets when you need to re-execute analysis code for a single post. Each target re-renders one or more `.qmd` files; follow with `make stats` to rebuild the complete site.

| Command                | Analysis                                                    |
| ---------------------- | ----------------------------------------------------------- |
| `make stats-diversity` | Alpha diversity (Kruskal-Wallis)                            |
| `make stats-permanova` | Beta diversity (PERMANOVA x phylum, family, genus, species) |
| `make stats-symptoms`  | Symptom scores x fungal composition (Spearman + PERMANOVA)  |
| `make stats-nutrients` | Nutrients x diversity by disease group (Spearman)           |
| `make stats-domain`    | Domain relationship analysis (OLS)                          |
| `make stats-evidence`  | Evidence-ranking matrix (disease-group validation)          |
| `make stats-pca`       | Diet + mycobiome PCA / clustering                           |

Example - re-run all PERMANOVA posts, then rebuild the site:

```sh
make stats-permanova
make stats
```

Helper R code for posts lives under `stats/R/` (e.g. `permanova_helpers.R`, `symptom_association_helpers.R`).

### Step 8 - Add a new analysis post

1.  Copy `stats/_template.qmd` into a dated folder, e.g. `stats/posts/2026-06-04-my-test/index.qmd`.

2.  Fill in the YAML front matter (`title`, `date`, `categories`, `description`). Use category `pre-specified` or `eda` so the post appears on the correct navbar tab. Add in post directory name in `here::i_am("posts/<post-directory-name>/index.qmd")`, so code can run from make commands (terminal) as well as interactively in IDE (Rstudio, Positron, etc).

3.  Document research question -> data -> methods -> results -> figures in the template sections.

4.  **Paths within a stats post** are anchored to `stats/` via `stats/.here`:
    - Save post figures to `stats/figures/` by using `here::here("figures", "my-figure.png")`
    - Read main-repo data into a post: `here::here("..", "data", "intermediate", "data.rds")`
    - Embed a main-repo figure (via Markdown): `![caption](../../../figures/mycobiome/my-figure.png)`

    `here::here("..")` steps from `stats/` to the repo root. Markdown image paths are relative to the `.qmd` file itself (inside `stats/posts/YYYY-MM-DD-name/`), so `../../../` reaches the repo root.

5.  Render:

    ```sh
    quarto render stats/posts/2026-06-04-my-test/index.qmd
    make stats
    ```

### End-to-end checklist (fresh clone -> stats site)

```sh
git clone https://github.com/tiffchu/BC_Childrens_Research_Institute_IBD_demo
cd BCCHR_IBD_Capstone
# -> place raw files in data/raw/ (see table above)
make setup
make mycobiome
make dietary
make characteristics
make merge
make stats
```

### Troubleshooting

| Problem                                                          | Fix                                                                                                                                                                                 |
| ---------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `Missing data/intermediate/alpha_long.rds`                       | Run `make mycobiome` first                                                                                                                                                          |
| `Raw characteristics file not found`                             | Add `OPT_Participant Characteristics.xlsx` to `data/raw/`                                                                                                                           |
| `renv` package errors                                            | Run `make setup`; confirm R 4.6.0                                                                                                                                                   |
| `make setup` fails to compile a package (e.g. `openssl`, `curl`) | See REDACTED - Windows needs Rtools, macOS (Apple Silicon) needs `brew install openssl` + `PKG_CONFIG_PATH` |
| `quarto: command not found`                                      | Install Quarto CLI from [quarto.org](https://quarto.org/docs/get-started/)                                                                                                          |
| EDA posts show broken images                                     | Run `make mycobiome` and `make dietary` to regenerate `figures/`                                                                                                                    |
| Stale results after reprocessing data                            | Run the relevant `make stats-*` target, then `make stats`                                                                                                                           |
| Reset all generated artifacts                                    | `make clean`, then repeat Steps 4-5                                                                                                                                                 |

### Additional troubleshooting with make + rscript

#### Installing `make` on Windows and adding to path

We use `make` to automate the analysis pipeline.

1.  Download `make` from the Windows release page you are using.

2.  Open the downloaded `.zip` file in File Explorer.

3.  Click `Extract all`.

4.  Extract it to:

    `C:\Users\YOUR_USERNAME\make-4.4.1`

    Replace `YOUR_USERNAME` with your actual Windows username.

To use `make` from Git Bash, add its `bin` folder to your PATH. In bash, paste these commands:

```bash
code ~/.bash_profile
export PATH="/c/Users/${USERNAME}/make-4.4.1/bin:${R_DIR}:$PATH"
```

Then save the file, close the terminal