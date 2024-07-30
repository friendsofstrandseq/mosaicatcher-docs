
# Installation & Update


!!! note "ashleys-qc-pipeline & mosaicatcher-pipeline dependencies"

    From 2.2.0, you don't need to clone both [ashleys-qc-pipeline preprocessing module](https://github.com/friendsofstrandseq/ashleys-qc-pipeline) and [mosaicatcher-pipeline](https://github.com/friendsofstrandseq/mosaicatcher-pipeline). By using `ashleys_pipeline_only=True` combined with `ashleys_pipeline=True` in the configuration of MosaiCatcher, this will stop the execution after the generation of the files related to ashleys-qc-pipeline. This allow you to use a single pipeline, repository and container and limit the potential conflicts by processing the same folder (data_location) by different repositories and set of files (including the workflow/data/ref_genomes references files).


## System requirements

This workflow is meant to be run in a Unix-based operating system when using the local execution profiles (tested on Ubuntu 18.04 & CentOS 7).

Minimum system requirements vary based on the use case. We highly recommend running it in a server environment with 32+GB RAM and 12+ cores.

- [Conda install instructions](https://conda.io/miniconda.html)
- [Singularity install instructions](https://sylabs.io/guides/3.0/user-guide/quick_start.html#quick-installation-steps)

Aside local execution, the pipeline can be run on a HPC cluster. The pipeline has been tested on SLURM-based clusters.

## Installation

### 0. [Optional] Install [Apptainer](https://apptainer.org/docs/admin/main/installation.html)

In order to run the pipeline, you can use the Apptainer containerization tool. This will allow you to run the pipeline in a controlled environment, with all the dependencies installed and ready to use.


### 1. Install snakemake through conda in a dedicated env


!!! warning "Snakemake version compatibility"

    The pipeline has been tested with Snakemake version 7+. Some tests are currently ongoing to ensure compatibility with Snakemake 8+.


To install the required snakemake dependencies, you can use the following commands:

```bash
conda create -n snakemake-env \
    -c conda-forge -c bioconda snakemake==7.32.4
conda activate snakemake-env
```

If you prefer to use a python virtual environment, you can use the following commands:

```bash
python3 -m venv snakemake-env
source snakemake-env/bin/activate
pip install snakemake==7.32.4
```

!!! note EMBL users

    Instead of installing your own snakemake environment, you can load an existing snakemake module with the following command: 
    
    ```
    module load snakemake/7.32.4-foss-2022b
    ```

### 2. Clone the repository and its submodules



```bash
# Clone the repository and its submodules
git clone --recurse-submodules https://github.com/friendsofstrandseq/mosaicatcher-pipeline.git && cd mosaicatcher-pipeline

# In each submodule, initialize and pull
git submodule update --init --remote --force --recursive
``` 



## Pipeline update procedure

If you already use a previous version of mosaicatcher-pipeline, here is a short procedure to update it:

- First, update all origin/<branch> refs to latest:

`git fetch --all`

- Jump to a new version & pull code:

`git checkout <VERSION> && git pull`

Then, to initiate or update git submodules:

`git submodule update --init --remote --force --recursive`