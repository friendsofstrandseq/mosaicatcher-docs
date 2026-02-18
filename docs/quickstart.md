# 🚀 Quick Start

Get up and running with MosaiCatcher in minutes.

!!! note

    - Ensure Snakemake is installed (see [Installation](../installation.md))
    - Snakemake can be loaded via module system, conda, or Pixi
    - For containerized execution, Apptainer is used (replaces Singularity)

## Install Snakemake (if needed)

### Option 1: Using Pixi (recommended)

```bash
pixi install snakemake
pixi run snakemake --version
```

### Option 2: Using conda

```bash
conda install -c conda-forge -c bioconda snakemake==9
```

### Option 3: Using module system (EMBL)

```bash
module avail snakemake
module load snakemake/<version>
snakemake --version
```

## Activate Snakemake environment

Before running, ensure Snakemake is available:

```bash
which snakemake
```

## Run on example data

Test the pipeline with included example data:

```bash
snakemake \
    --cores 6 \
    --configfile .tests/config/simple_config_mosaicatcher.yaml \
    --sdm conda
```

For containerized execution (requires Apptainer):

```bash
snakemake \
    --cores 6 \
    --configfile .tests/config/simple_config_mosaicatcher.yaml \
    --sdm conda apptainer \
    --apptainer-args "-B /<disk>:/<disk>"
```

The container image (`ghcr.io/friendsofstrandseq/mosaicatcher-pipeline:hg38-v2.4.0` for hg38) is pulled automatically on first run.

## Run on your own data

Analyze your own Strand-Seq samples:

```bash
snakemake \
    --cores <N> \
    --config data_location=<PATH> \
    --sdm conda
```

Replace:
- `<N>` with number of cores to use
- `<PATH>` with path to your data folder

## Generate report (optional)

Create an interactive HTML report:

```bash
snakemake \
    --cores <N> \
    --config data_location=<PATH> \
    --report <report>.zip \
    --report-stylesheet workflow/report/custom-stylesheet.css
```

Next: See [Input Data Formats](input-data.md) to prepare your data correctly.
