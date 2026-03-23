# 🇪🇺 EMBL HPC Guide

This page covers EMBL-specific optimizations for running MosaiCatcher on the EMBL HPC cluster (seneca/raven). The dedicated profile at `workflow/snakemake_profiles/mosaicatcher-pipeline/v9/HPC/slurm_EMBL_apptainer/` pre-configures all of the settings below.

---

## Required software

| Package | Version | Notes |
|---|---|---|
| `snakemake` | **9.13.7** | Preferred version for SLURM + Apptainer; install via conda/pixi |
| `snakemake-executor-plugin-slurm` | latest | Required for SLURM job submission |
| `snakemake-storage-plugin-http` | latest | Required for downloading reference files over HTTP |

Install into a dedicated environment:

```bash
conda create -n snakemake-v9 -c conda-forge -c bioconda snakemake=9.13.7 snakemake-executor-plugin-slurm snakemake-storage-plugin-http
conda activate snakemake-v9
```

Or with pixi (project-level):

```bash
pixi add snakemake=9.13.7 snakemake-executor-plugin-slurm snakemake-storage-plugin-http
```

---

## Basic usage

```bash
snakemake \
    --profile workflow/snakemake_profiles/mosaicatcher-pipeline/v9/HPC/slurm_EMBL_apptainer/ \
    --config data_location=<DATA_FOLDER> reference=hg38 reference_base_dir=/scratch_cached/korbel/references
```

---

## Shared references

Reference genomes and BWA indexes are pre-built and shared across the Korbel group at:

```
/scratch_cached/korbel/references/
```

No manual download or index building is needed — the pipeline resolves the correct FASTA and BWA index files automatically.

`/scratch_cached/` is a fast-access cached filesystem backed by `/scratch/`, avoiding repeated transfers from the slow shared `/g/` filesystem.

!!! warning "Always pass `reference_base_dir` explicitly"

    Due to a [known Snakemake v9 bug (#3429)](https://github.com/snakemake/snakemake/issues/3429),
    the profile's `config:` section is silently dropped whenever any `--config` flag is passed on
    the CLI. Until this is fixed upstream, always include `reference_base_dir` in every run:

    ```bash
    --config ... reference_base_dir=/scratch_cached/korbel/references
    ```

---

## Shared container cache

Apptainer images are cached at a shared group location to avoid redundant pulls:

```
/scratch/korbel/shared/apptainer_cache/
```

This directory holds both the OCI **blob cache** (layers pulled from GHCR) and the final converted **`.simg` image files**. Once any group member pulls an image for a given assembly + version, it is available to all subsequent users at no extra cost.

Conda environments are similarly shared at:

```
/g/korbel2/shared/conda_envs/
```

!!! note "First-time setup (admin)"

    All shared directories must be group-writable with a setgid bit so new files inherit the group automatically. Run once by the group admin:

    ```bash
    bash workflow/scripts/toolbox/setup_shared_caches.sh
    ```

    This creates the directories, sets `chmod 2775`, and applies default ACLs via `setfacl`.

---

## Log management

SLURM logs are written to a per-user scratch directory:

```
/scratch/$USER/mosaicatcher_logs/slurm/
```

Logs for successful jobs are deleted automatically. Logs older than 30 days are pruned. This keeps scratch usage manageable across users.

A resource efficiency report is generated at:

```
/scratch/$USER/mosaicatcher_logs/slurm_efficiency_report.txt
```

Jobs using less than 80% of their requested memory or runtime are flagged, helping tune resource requests over time.

---

## Apptainer bind paths

The profile binds all relevant EMBL filesystems into the container automatically:

```
-B /g,/scratch,/scratch_cached,/tmp
```

No manual `--apptainer-args` override is needed when using the EMBL profile.

---

## Job grouping

The EMBL profile enables two Snakemake job-grouping strategies to reduce SLURM scheduler overhead and improve throughput when processing many cells.

### Horizontal grouping

Many independent short jobs (e.g., per-cell BAM indexing, QC) are bundled into a single SLURM allocation. Instead of flooding the scheduler with hundreds of 1-minute jobs, they are packed into a handful of multi-slot jobs and run sequentially within the allocation.

```
Without grouping:                With horizontal grouping:

cell_01/index → SLURM job 1     ┌── SLURM job 1  ──────────────┐
cell_02/index → SLURM job 2     │  cell_01/index               │
cell_03/index → SLURM job 3     │  cell_02/index               │
cell_04/index → SLURM job 4     │  cell_03/index               │
cell_05/index → SLURM job 5     │  cell_04/index               │
cell_06/index → SLURM job 6     │  cell_05/index               │
      ...                       │  cell_06/index  ...          │
                                └──────────────────────────────┘
```

Configured via `--group-components` in the profile. Reduces queue wait time and scheduler load significantly for large plates (96-well+).

### Vertical grouping

Tightly coupled sequential steps (mapping → sort → dedup → index) are chained within a single SLURM job, avoiding repeated queue submission for steps that would otherwise wait on each other.

```
Without grouping:             With vertical grouping:

mapping  → SLURM job 1        ┌── SLURM job 1  ────────┐
  ↓ (queue wait)              │  mapping               │
sort     → SLURM job 2        │     ↓                  │
  ↓ (queue wait)              │  sort                  │
dedup    → SLURM job 3        │     ↓                  │
  ↓ (queue wait)              │  dedup                 │
index    → SLURM job 4        │     ↓                  │
                              │  index                 │
                              └────────────────────────┘
```

Configured via `--groups` and `--group-components` in the profile. Eliminates intermediate queue waits for steps where the bottleneck is data dependency, not compute.
