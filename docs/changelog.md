# Changelog

---

## 2.5.0 — *unreleased*

### 🐳 Docker images

| Assembly | Image |
|----------|-------|
| hg38 | `ghcr.io/friendsofstrandseq/mosaicatcher-pipeline:hg38-2.5.0` |
| hg19 | `ghcr.io/friendsofstrandseq/mosaicatcher-pipeline:hg19-2.5.0` |
| T2T | `ghcr.io/friendsofstrandseq/mosaicatcher-pipeline:T2T-2.5.0` |
| mm10 | `ghcr.io/friendsofstrandseq/mosaicatcher-pipeline:mm10-2.5.0` |
| mm39 | `ghcr.io/friendsofstrandseq/mosaicatcher-pipeline:mm39-2.5.0` |

### 📋 Summary

This release focuses on pipeline robustness: corrupted BAMs are now caught immediately after alignment and merging, StrandPhaseR no longer crashes on chromosomes with insufficient SNPs, and a Snakemake v9 checkpoint deadlock in the StrandPhaseR step has been resolved. SLURM resource requests across memory, CPU threads, and runtimes have been right-sized based on empirical data from ~9,700 jobs.

### ✨ Features

- **AVITI watcher** — new Snakemake v9 Python API script (`workflow/scripts/toolbox/watcher/`) to monitor AVITI sequencing runs and trigger pipeline execution automatically
- **Shared cache setup** — `setup_shared_caches.sh` creates group-writable Apptainer image and conda environment directories on EMBL HPC with correct ACLs

### 🐛 Fixes

- **Assembly binning** — bin BED files and chromosome lists are now resolved per reference assembly; previously all assemblies fell back to the primary reference, causing incorrect binning for hg19, T2T, mm10, mm39
- **StrandPhaseR checkpoint deadlock** — resolved a Snakemake v9 deadlock where the `jobstep` checkpoint caused StrandPhaseR jobs to hang indefinitely
- **StrandPhaseR chromosome guard** — chromosomes with fewer SNPs than StrandPhaseR's minimum are now skipped with a logged warning instead of crashing with a cryptic R error
- **Corrupted BAM detection (alignment)** — `samtools quickcheck` added after alignment group; truncated BAMs from node failures are caught before downstream steps
- **Corrupted BAM detection (merge)** — `samtools quickcheck` added after `mergeBams` and `mergeSortBams`; prevents silent propagation of corrupt merged files on resume
- **SLURM profile runtime comment** — `default-resources: runtime=600` was documented as "minutes" but is in **seconds** (= 10 min); comment corrected

### ⚙️ Misc

- **Right-sized SLURM resources** (based on empirical data from 9,700+ jobs across 22 runs):
    - `create_haplotag_table` memory: 8 GB → 12 GB base (p95 observed: 9.7 GB, max: 17.4 GB)
    - `mosaiClassifier_calc_probs` memory: 8 GB → 2.5 GB base (max observed: 2.8 GB)
    - `run_strandphaser_per_chrom` memory: 8 GB → 4 GB base; runtime: 600 → 120 min
    - `call_SNVs_bcftools_chrom` memory: 8 GB → 1 GB base; runtime: 180 → 30 min
    - `mosaiClassifier_calc_probs` runtime: 180 → 60 min
    - `ashleys_generate_features` threads: 64 → 12; runtime: 3600 → 60 min
    - `ploidy_estimation` threads: 48 → 24

---

## 2.4.0 — February 2026

### 🐳 Docker images

Assembly-specific images introduced in this release. Each image embeds the matching BSgenome R package:

| Assembly | Image |
|----------|-------|
| hg38 | `ghcr.io/friendsofstrandseq/mosaicatcher-pipeline:hg38-2.4.0` |
| hg19 | `ghcr.io/friendsofstrandseq/mosaicatcher-pipeline:hg19-2.4.0` |
| T2T | `ghcr.io/friendsofstrandseq/mosaicatcher-pipeline:T2T-2.4.0` |
| mm10 | `ghcr.io/friendsofstrandseq/mosaicatcher-pipeline:mm10-2.4.0` |
| mm39 | `ghcr.io/friendsofstrandseq/mosaicatcher-pipeline:mm39-2.4.0` |

### 📋 Summary

2.4.0 is a major infrastructure release: the pipeline migrates to Snakemake v9 and Pixi for reproducible environment management, the ashleys-qc-pipeline is fully integrated (no longer a submodule), and all hardcoded genome references are replaced by a centralized genome registry. New assemblies (mm39, CanFam) and a new EMBL HPC Apptainer profile with shared container caches are included.

### ✨ Features

- **Snakemake v9 + Pixi** — unified package management via `pixi.toml`; `pixi run snakemake` replaces all conda-activate workflows
- **Assembly-specific containers** — one GHCR image per reference genome embedding the matching BSgenome R package; automated multi-assembly container builds in CI
- **Centralized genome registry** — all hardcoded genome paths replaced by a single `references_data` config block; adding a new assembly now requires a single config entry
- **ashleys-qc-pipeline integration** — preprocessing pipeline no longer a git submodule; lives directly in `workflow/rules/ashleys/`
- **mm39 full support** — normalization files, blacklist regions, BSgenome package for mouse GRCm39
- **CanFam framework** — reference infrastructure in place for canfam3/canfam4 (normalization files pending)
- **Pre-built iGenomes index download** — `download_prebuilt_indexes` parameter: skip local BWA index building by downloading from AWS iGenomes
- **Ploidy estimation module** — `ploidy` parameter: optional detection of haploid chromosomes/segments to guide StrandPhaseR
- **Centralized version management** — `pixi run bump-patch|bump-minor|bump-major|bump-beta|bump-release`; automated changelog generation for releases
- **EMBL HPC Apptainer profile** — new profile at `workflow/snakemake_profiles/mosaicatcher-pipeline/v9/HPC/slurm_EMBL_apptainer/` with shared container caches, SLURM job grouping, and resource efficiency reporting
- **Configurable `reference_base_dir`** — multi-user reference sharing on HPC without per-user downloads
- **SLURM job grouping** — horizontal (batch short jobs) and vertical (chain sequential steps) grouping to reduce scheduler overhead

### 🐛 Fixes

- Empty DataFrames in clustering plots no longer raise errors
- Assertion added for iGenomes base URL availability before attempting index download
- Redundant `index_input_bam` rule removed
- `regenotype_SNVs` handles empty site lists correctly
- Conda environments cleaned: anaconda/defaults channels removed across all envs

### ⚙️ Misc

- `keep_ashleys_predictions` parameter to control retention of ashleys ML prediction files
- `external_data_v8.smk` renamed to `external_data_v9.smk`
- snakemake_profiles submodule replaced with local versioned files
- CI matrix expanded: mm39 indexed and tested; T2T with latency-wait for storage

---

## 2.3.0 — July 2024

### 🐳 Docker images

```
ghcr.io/friendsofstrandseq/mosaicatcher-pipeline:2.3.x
```

### 📋 Summary

2.3.0 adds BreakpointR and WhatsHap as optional execution targets, enabling users to stop the pipeline at intermediate steps for manual inspection. Snakemake v7 and v8 are both tested in CI, and several conda environment and path issues are resolved.

### ✨ Features

- **BreakpointR integration** — `breakpointr: True` parameter activates BreakpointR for SV calling; `breakpointr_only: True` stops execution after BreakpointR
- **`whatshap_only` parameter** — stop execution after WhatsHap phasing for manual review before SV classification
- **Paired-end support improvements** — `paired_end` parameter handling stabilised
- **Data reorganisation script** — `workflow/scripts/toolbox/reorganise_data/` for reformatting data from external sequencing providers (DKFZ)

### 🐛 Fixes

- IGV session script: reference genome path correctly resolved
- scNOVA_DL conda environment fixed (note: only functional outside EMBL due to network restrictions)
- Missing `publishdir` function in `common.smk` restored
- Conda environments: removed anaconda/defaults channels, migrated to conda-forge/bioconda only
- SLURM job time formatting corrected
- Duplicate column in clustering plot fixed (previously silently ignored by some ggplot2 versions)
- Missing wildcards in `grouptrack` rule resolved

### ⚙️ Misc

- CI: Snakemake v7 and v8 parallel testing
- `.tests-mock` submodule added for faster CI without LFS
- StrandPhaseR updated to bioconda package (removed custom install)
