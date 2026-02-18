
# 📦 Installation & Update


!!! note "Ashleys-QC integration"

    From version 2.4.0 onwards, ashleys-qc preprocessing is directly integrated into MosaiCatcher. You only need to clone [mosaicatcher-pipeline](https://github.com/friendsofstrandseq/mosaicatcher-pipeline). The separate [ashleys-qc-pipeline repository](https://github.com/friendsofstrandseq/ashleys-qc-pipeline) is now legacy.


## System requirements

This workflow is meant to be run in a Unix-based operating system when using the local execution profiles (tested on Ubuntu 18.04 & CentOS 7).

Minimum system requirements vary based on the use case. We highly recommend running it in a server environment with 32+GB RAM and 12+ cores.

- [Conda install instructions](https://conda.io/miniconda.html)
- [Singularity install instructions](https://sylabs.io/guides/3.0/user-guide/quick_start.html#quick-installation-steps)

Aside local execution, the pipeline can be run on a HPC cluster. The pipeline has been tested on SLURM-based clusters.

## Installation

### 0. [Optional] Install [Apptainer](https://apptainer.org/docs/admin/main/installation.html)

In order to run the pipeline, you can use the Apptainer containerization tool. This will allow you to run the pipeline in a controlled environment, with all the dependencies installed and ready to use.


MosaiCatcher provides pre-built container images on GitHub Container Registry (GHCR). Each container is assembly-specific because it embeds the corresponding BSgenome R package required for haplotype analysis.

**Available container tags:**

| Reference Genome | Container URI |
| ---------------- | ------------- |
| hg38 (GRCh38)   | `ghcr.io/friendsofstrandseq/mosaicatcher-pipeline:hg38-v2.4.0` |
| hg19 (GRCh37)   | `ghcr.io/friendsofstrandseq/mosaicatcher-pipeline:hg19-v2.4.0` |
| T2T (CHM13)     | `ghcr.io/friendsofstrandseq/mosaicatcher-pipeline:T2T-v2.4.0` |
| mm10             | `ghcr.io/friendsofstrandseq/mosaicatcher-pipeline:mm10-v2.4.0` |
| mm39             | `ghcr.io/friendsofstrandseq/mosaicatcher-pipeline:mm39-v2.4.0` |

!!! note "Why assembly-specific containers?"

    Each container embeds the BSgenome R package for its reference genome. Splitting by assembly keeps image sizes manageable while ensuring StrandPhaseR has the correct genomic data available at runtime.

Use `--sdm conda apptainer` in your Snakemake command to activate container execution. See [Usage](usage.md) for examples.

### 1. Install snakemake through conda or pixi

!!! warning "Snakemake version compatibility"

    - **Until v2.3.5**: Compatible with Snakemake v7
    - **From v2.4.0+**: Compatible with Snakemake v9+

#### Option A: Conda

```bash
conda create -n snakemake-env \
    -c conda-forge -c bioconda snakemake==9
conda activate snakemake-env
```

Or with pip in a virtual environment:

```bash
python3 -m venv snakemake-env
source snakemake-env/bin/activate
pip install snakemake==9
```

#### Option B: Pixi (recommended)

[Pixi](https://pixi.sh/) automatically manages dependencies and environments, eliminating version conflicts:

```bash
pixi run snakemake --version
```

#### Snakemake v9 plugins

Snakemake v9 uses a plugin system for storage backends and cluster executors. Install the required and recommended plugins after installing Snakemake:

```bash
# Required — used to download reference files over HTTP
pip install snakemake-storage-plugin-http

# Recommended for HPC — SLURM executor
pip install snakemake-executor-plugin-slurm
```

With Pixi, these are managed automatically via `pixi.toml`.

!!! note "EMBL users"

    For module-based systems, check available Snakemake modules:

    ```
    module avail snakemake
    module load snakemake/<version>
    ```

### 2. Clone the repository and its submodules



```bash
# Clone the repository and its submodules
git clone --recurse-submodules https://github.com/friendsofstrandseq/mosaicatcher-pipeline.git && cd mosaicatcher-pipeline

# In each submodule, initialize and pull
git submodule update --init --remote --force --recursive
``` 



## Pipeline update procedure

If you already use a previous version of mosaicatcher-pipeline, follow these steps:

1. Update all origin/<branch> refs to latest:

```bash
git fetch --all
```

2. Jump to a new version & pull code:

```bash
git checkout <VERSION> && git pull
```

3. Update git submodules:

```bash
git submodule update --init --remote --force --recursive
```

4. **Check Snakemake compatibility** for the version you're updating to (see warning above)