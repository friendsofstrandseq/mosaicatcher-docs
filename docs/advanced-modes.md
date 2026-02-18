# 🔬 Advanced Modes

MosaiCatcher supports several advanced execution modes for specialized analyses.

## ArbiGent mode

Genotype a given list of positions specified in a BED file.

Available from v1.9.0+.

### Setup

Set the `arbigent=True` config parameter. An alternative pipeline branch will execute with results targeted for ArbiGent.

### Command

```bash
snakemake \
    --cores <N> \
    --config \
        data_location=<INPUT_DATA_FOLDER> \
        arbigent=True \
    --profile workflow/snakemake_profiles/local/conda_singularity/
```

### Custom BED file

A default BED file is provided at: `workflow/data/arbigent/manual_segmentation.bed`

Specify a custom BED file by setting the `arbigent_bed_file` parameter:

```bash
--config arbigent_bed_file=<path/to/custom.bed>
```

!!! note

    If you modify the chromosome list (e.g., exclude chrX and chrY), MosaiCatcher automatically extracts only matching rows from the BED file.

## scNOVA mode

Determine nucleosome occupancy associated with SV calls.

Available from v1.9.2+.

!!! warning

    Following Anaconda licensing changes at EMBL, scNOVA can only run with singularity profiles that have pre-built conda environments.

### Two-step workflow

#### Step 1: Generate SV calls

Run MosaiCatcher normally to generate initial SV calls and subclonality data.

#### Step 2: Prepare subclonality file

Create a TSV file at: `<DATA_LOCATION>/<SAMPLE>/scNOVA_input_user/input_subclonality.txt`

Format (tab-separated):

```
Filename             Subclonality
TALL3x1_DEA5_PE20406 clone2
TALL3x1_DEA5_PE20414 clone2
TALL3x1_DEA5_PE20415 clone1
TALL3x1_DEA5_PE20416 clone1
TALL3x1_DEA5_PE20417 clone1
```

Clone identifiers must follow the format `clone<N>`.

#### Step 3: Run scNOVA analysis

```bash
snakemake \
    --cores <N> \
    --config \
        data_location=<INPUT_DATA_FOLDER> \
        scNOVA=True \
    --profile workflow/snakemake_profiles/local/conda_singularity/
```

## Multistep normalization

Apply advanced normalization (library size, GC correction, Variance Stabilizing Transformation) for both visualization and SV calling.

Available from v2.1.0+.

### For visualization only

```bash
snakemake \
    --cores <N> \
    --config \
        data_location=<INPUT_DATA_FOLDER> \
        multistep_normalisation=True \
    --profile workflow/snakemake_profiles/local/conda_singularity/
```

### For SV calling

```bash
snakemake \
    --cores <N> \
    --config \
        data_location=<INPUT_DATA_FOLDER> \
        multistep_normalisation=True \
        multistep_normalisation_for_SV_calling=True \
    --profile workflow/snakemake_profiles/local/conda_singularity/
```

!!! warning

    `multistep_normalisation_for_SV_calling` is mutually exclusive with `hgsvc_based_normalized_counts`.

## BreakpointR integration

Compute breakpoints on Strand-Seq data (David Porubsky).

Enable with the `breakpointr=True` parameter.

Stop after BreakpointR execution with `breakpointr_only=True`.

## scTRIP multiplot

Advanced visualization module by Marco Cosenza (requires separate repository access).

Available from v2.2.2+.

### Setup

Clone the scTRIP multiplot repository into: `workflow/scripts/plotting/scTRIP_multiplot`

(Contact Marco Cosenza for private repository access)

### Enable

```bash
--config scTRIP_multiplot=True
```

Or set `scTRIP_multiplot: true` in `config/config.yaml`.

### Output

Example multiplot visualization available in [Outputs](outputs.md#sctrip-multiplot-marco-cosenza).

## IGV tracks upload

Load MosaiCatcher results in the Integrative Genomics Viewer (IGV).

### Steps

1. **Start IGV** - Launch the IGV application
2. **Open session** - Go to `File` > `Open Session`
3. **Navigate** - Go to: `<OUTPUT>/plots/IGV/`
4. **Select** - Choose `<SAMPLE>_IGV_session.xml`

### Automatic setup

The reference genome set during MosaiCatcher execution automatically loads, along with Watson/Crick and SV tracks for each cell.

For report generation, see [Quick Start](quickstart.md#generate-report-optional).
