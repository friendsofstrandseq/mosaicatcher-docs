# ▶️ Running the Pipeline

Execute MosaiCatcher on your data using different compute environments.

!!! note

    - Verify your conda/mamba environment is properly configured
    - Check Snakemake is installed: `which snakemake`
    - Ensure your preferred profile is available in `workflow/snakemake_profiles/`

## Local execution

Run the pipeline on your local machine or server:

### Conda environments (recommended)

```bash
snakemake \
    --cores <N> \
    --config data_location=<DATA_FOLDER> \
    --sdm conda
```

### Containerized execution (Apptainer)

```bash
snakemake \
    --cores <N> \
    --config \
        data_location=<DATA_FOLDER> \
        reference=hg38 \
    --sdm conda apptainer \
    --apptainer-args "-B /<disk>:/<disk>"
```

The container image is automatically selected based on the `reference` config parameter (e.g., `hg38-v2.4.0`). Images are pulled from GHCR on first run.

Replace:
- `<N>` with the number of cores available
- `<DATA_FOLDER>` with your data location
- `<disk>` with your disk path (e.g., `/g` or `/scratch` at EMBL)

## HPC execution (SLURM)

Run on HPC clusters using SLURM scheduling:

```bash
snakemake \
    --config \
        data_location=<DATA_FOLDER> \
    --profile workflow/snakemake_profiles/mosaicatcher-pipeline/v9/HPC/slurm_generic/ \
    --apptainer-args "-B /<disk>:/<disk>"
```

### Configuration

Before first execution, modify the SLURM profile configuration for your cluster:

`workflow/snakemake_profiles/mosaicatcher-pipeline/HPC/slurm_generic/config.yaml`

Adjust memory, job time limits, and other cluster-specific parameters.

### Out-of-Memory (OOM) handling

MosaiCatcher uses Snakemake's restart feature to automatically handle OOM errors. Memory allocation scales on each retry, with starting values and step sizes tuned per rule type:

- **Lightweight rules** (indexing, QC): start at 1 GB, double on each retry
- **Medium rules** (alignment, sorting): start at 4 GB, scale up to 32 GB
- **Heavy rules** (segmentation, StrandPhaseR): start at 8 GB, scale up to 64 GB

Jobs are retried up to 5 times before failing. No manual intervention is needed in most cases.

### EMBL HPC

EMBL users have a dedicated pre-configured profile. See the [EMBL HPC guide](embl.md) for details on shared references, container cache, and job grouping.

```bash
--profile workflow/snakemake_profiles/mosaicatcher-pipeline/v9/HPC/slurm_EMBL_apptainer/
```

## Next steps

- See [Quick Start](quickstart.md) for minimal examples
- See [Input Data Formats](input-data.md) to prepare your data
- See [Advanced Modes](advanced-modes.md) for specialized analyses
- See [Parameters](parameters.md) for configuration options
