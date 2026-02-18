# Input Data Formats

Prepare your data according to these specifications before running MosaiCatcher.

## FASTQ input (with preprocessing)

If using the integrated ashleys-qc preprocessing (`ashleys_pipeline=True`):

### Directory structure

```bash
Parent_folder
|-- Sample_1
|   `-- fastq
|       |-- Cell_01.1.fastq.gz
|       |-- Cell_01.2.fastq.gz
|       |-- Cell_02.1.fastq.gz
|       |-- Cell_02.2.fastq.gz
|       |-- Cell_03.1.fastq.gz
|       |-- Cell_03.2.fastq.gz
|       |-- Cell_04.1.fastq.gz
|       `-- Cell_04.2.fastq.gz
|
`-- Sample_2
    `-- fastq
        |-- Cell_21.1.fastq.gz
        |-- Cell_21.2.fastq.gz
        |-- Cell_22.1.fastq.gz
        |-- Cell_22.2.fastq.gz
        |-- Cell_23.1.fastq.gz
        |-- Cell_23.2.fastq.gz
        |-- Cell_24.1.fastq.gz
        `-- Cell_24.2.fastq.gz
```

Create a subdirectory `Parent_Folder/sampleName/` for each sample. FASTQ files must be placed in a `fastq` subfolder with naming syntax: `<CELL>.<1|2>.fastq.gz` where `1|2` denotes pair identifiers (paired-end).

!!! warning "Sample naming"

    Limit underscores (`_`) in filenames to 2 maximum. The ashleys-qc ML tool has issues with 3+ underscores in sample names.

## BAM input (without preprocessing)

For direct analysis of aligned BAM files:

### BAM file requirements

Follow these strict requirements for Strand-Seq single-cell BAM data:

- **Filename suffix**: Must end with `.sort.mdup.bam`
- **One file per cell**: Each BAM file represents one cell
- **Sorted and indexed**: BAM files must be sorted and have `.bai` index files
  - If BAM files lack indices, use a writable folder so the pipeline can generate them
  - Index file timestamps must be newer than BAM file timestamps
- **Read groups**: Each BAM file must contain a read group (`@RG`) with a sample name (`SM`) matching the folder name

### Low-quality cell filtering

From v1.6.1+, MosaiCatcher automatically filters out low-quality cells based on coverage using the `filter_bad_cells` checkpoint rule.

Cells flagged as low-quality are listed in: `<OUTPUT>/cell_selection/<SAMPLE>/labels.tsv`

The quality threshold is determined by the `pass1` column in the mosaic count info file and varies with window size (larger windows require higher read counts per bin).

### Directory structure - Current (recommended)

```bash
Parent_folder
|-- Sample_1
|   `-- bam
|       |-- Cell_01.sort.mdup.bam
|       |-- Cell_02.sort.mdup.bam
|       |-- Cell_03.sort.mdup.bam
|       `-- Cell_04.sort.mdup.bam
|
`-- Sample_2
    `-- bam
        |-- Cell_21.sort.mdup.bam
        |-- Cell_22.sort.mdup.bam
        |-- Cell_23.sort.mdup.bam
        `-- Cell_24.sort.mdup.bam
```

For each sample, create a `Parent_Folder/sampleName/bam/` subdirectory. Low-quality cells are filtered automatically based on coverage.

### Directory structure - Legacy (`input_bam_legacy=True`)

For older workflows (MosaiCatcher ≤ 1.5), a `selected` folder could specify pre-filtered cells:

```bash
Parent_folder
|-- Sample_1
|   |-- bam
|   |   |-- Cell_01.sort.mdup.bam
|   |   |-- Cell_02.sort.mdup.bam
|   |   |-- Cell_03.sort.mdup.bam
|   |   `-- Cell_04.sort.mdup.bam
|   `-- selected
|       |-- Cell_01.sort.mdup.bam
|       `-- Cell_04.sort.mdup.bam
|
`-- Sample_2
    |-- bam
    |   |-- Cell_01.sort.mdup.bam
    |   |-- Cell_02.sort.mdup.bam
    |   |-- Cell_03.sort.mdup.bam
    |   `-- Cell_04.sort.mdup.bam
    `-- selected
        |-- Cell_03.sort.mdup.bam
        `-- Cell_04.sort.mdup.bam
```

In legacy mode, only cells present in both the `selected` folder AND passing coverage checks are processed.

!!! warning "Legacy mode constraints"

    - Only the **intersection** of cells in `selected` and passing coverage are used
    - Example: A cell in `selected` but with low coverage will be filtered out
    - `ashleys_pipeline=True` and `input_bam_legacy=True` are **mutually exclusive**
    - Sample names should not contain more than 2 underscores
