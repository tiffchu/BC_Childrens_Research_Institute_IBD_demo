# BCCHR IBD Public Demo

This repository is a **public demo copy** of the original project.

## Important note

The data used in this demo is **generated fake data**. Any files produced from it, including:

- dashboard outputs
- figures
- statistical results
- rendered Quarto pages

are **not real patient data** and should not be interpreted as real clinical findings.

## What this repo is for

This repo is meant to show the project's:

- code structure
- data-processing pipeline
- dashboard implementation
- Quarto/statistical reporting workflow

## What was removed

- private raw, intermediate, and processed data
- live credentials
- deployment/account metadata
- private generated outputs
- local machine/cache artifacts

## Main folders

- `src/` - analysis and pipeline code
- `dashboard/` - Shiny dashboard source
- `stats/` - Quarto analysis site source
- `data/` - demo input/output folders
- `figures/` - generated figures and static diagrams

## Run the demo

From the repository root:

```sh
make setup
make mycobiome
make dietary
make characteristics
make merge
make stats
make app
```

## Publish this as a new public repo

If you want this folder to become a brand-new GitHub repository with fresh history:

```sh
git init
git add .
git commit -m "Initial public demo release"
git branch -M main
git remote add origin <your-public-repo-url>
git push -u origin main
```
