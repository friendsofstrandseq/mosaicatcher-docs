# 🧬 Reference Genomes

MosaiCatcher supports multiple reference genome assemblies. Each assembly requires specific normalization files, blacklist regions, and a matching BSgenome R package for StrandPhaseR.

## Supported Assemblies

| Assembly | Organism | Status | Container tag |
| -------- | -------- | ------ | ------------- |
| hg38 (GRCh38) | Human | Fully supported | `hg38-v2.4.0` |
| hg19 (GRCh37) | Human | Fully supported | `hg19-v2.4.0` |
| T2T (CHM13v2) | Human | Fully supported | `T2T-v2.4.0` |
| mm10 (GRCm38)  | Mouse  | Fully supported | `mm10-v2.4.0` |
| mm39 (GRCm39)  | Mouse  | Fully supported (see caveats) | `mm39-v2.4.0` |
| canfam3        | Dog    | Framework-ready (normalization files pending) | — |
| canfam4        | Dog    | Framework-ready (normalization files pending) | — |

## Selecting a Reference

Set the `reference` parameter in your config or via `--config`:

```bash
snakemake --cores <N> --config data_location=<PATH> reference=mm39 --sdm conda
```

For containerized runs, the container image is automatically selected to match the `reference` value:

```bash
snakemake --cores <N> \
    --config data_location=<PATH> reference=hg19 \
    --sdm conda apptainer \
    --apptainer-args "-B /g:/g"
# Uses ghcr.io/friendsofstrandseq/mosaicatcher-pipeline:hg19-v2.4.0
```

## Assembly-Specific Notes

### mm39 (GRCm39) Caveats

mm39 is fully supported but has two known limitations to be aware of:

#### Missing 50 kb / 100 kb normalization files

Normalization files for 50 kb and 100 kb bin sizes are not yet available for mm39. Use `window: 200000` (200 kb bins) when analyzing mm39 data:

```yaml
# config/config.yaml
reference: mm39
window: 200000
```

#### HGSVC blacklist — chrX validation needed for XX samples

The blacklist regions file for mm39 was generated using an XO (male) control cell line. As a result, chrX coverage normalization may be biased for XX (female) samples. We recommend:

- Excluding chrX from analysis for XX samples: `chromosomes_to_exclude: [chrX]`
- Or interpreting chrX SV calls from XX mm39 samples with extra caution

!!! warning "mm39 chrX results"

    For XX (female) mouse samples on mm39: chrX blacklist regions were derived from a male (XO) control. SV calls on chrX should be validated independently.

### canfam3 / canfam4 (Dog)

The reference genome infrastructure (genome download, chromosome lists) is in place for canfam3 and canfam4. However, normalization files and blacklist regions are not yet generated. These assemblies are **framework-ready** and will be fully supported in a future release.

## Pre-built BWA Indexes

To skip building BWA indexes locally, enable the `download_prebuilt_indexes` parameter:

```yaml
download_prebuilt_indexes: True
```

This downloads pre-built indexes from AWS iGenomes. Useful on HPC systems where index building is time-consuming or storage-constrained.

## Shared Reference Directory (HPC)

On HPC systems with multiple users, configure `reference_base_dir` to point to a shared location:

```yaml
reference_base_dir: /g/korbel/shared/genomes
```

All users sharing this directory avoid redundant downloads and index building.
