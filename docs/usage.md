# Usage


!!! note 

    - Please be careful of your conda/mamba setup, if you applied specific constraints/modifications to your system, this could lead to some versions discrepancies.
    - mamba is usually preferred but might not be installed by default on a shared cluster environment
    - You will need to verify that this conda environment is activated and provide the right snakemake before each execution (`which snakemake` command)



## Get Started

!!! note "Note for 🇪🇺 EMBL users"

    Use the following command for singularity-args parameter: `--singularity-args "-B /g,/scratch"`

### Run pipeline on example data

Run on example data on only one small chromosome (`<disk>` must be replaced by your disk letter/name, `/g` or `/scratch` at EMBL for example)

```bash
snakemake \
    --cores 6 \
    --configfile .tests/config/simple_config.yaml \
    --profile workflow/snakemake_profiles/mosaicatcher-pipeline/v7/local/conda_singularity/ \
    --singularity-args "-B /<disk>:/<disk>"
```

Then, generate a report on example data.

```bash
snakemake \
    --cores 1 \
    --configfile .tests/config/simple_config.yaml \
    --profile workflow/snakemake_profiles/mosaicatcher-pipeline/v7/local/conda_singularity/ \
    --singularity-args "-B /disk:/disk" \
    --report report.zip \
    --report-stylesheet workflow/report/custom-stylesheet.css
```


### Run on your own data locally

1. Run your own analysis **locally** (`<disk>` must be replaced by your disk letter/name, `/g` or `/scratch` at EMBL for example)

```bash
snakemake \
    --cores <N> \
    --config \
        data_location=<PATH> \
    --profile workflow/snakemake_profiles/mosaicatcher-pipeline/v7/local/conda_singularity/ \
    --singularity-args "-B /<disk>:/<disk>"
```


## Detailed instructions

### MosaiCatcher full execution (with ashleys-qc preprocessing module enabled)



#### Directory structure

The ashleys-qc-pipeline takes as input Strand-Seq fastq file following the structure below:

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

Thus, in a `Parent_Folder`, create a subdirectory `Parent_Folder/sampleName/` for each `sample`. Each Strand-seq FASTQ files of this sample need to go into the `fastq` folder and respect the following syntax: `<CELL>.<1|2>.fastq.gz`, `1|2` corresponding to the pair identifier.


!!! warning "Samples naming"

    We advice to limit to the number of 2 the "\_" characters in your file names (ashleys-qc tool will react weirdly with 3 or more "_" in the sample name)

### MosaiCatcher execution (without preprocessing)

#### BAM input requirements


It is important to follow these rules for Strand-Seq single-cell BAM data

- **BAM file name ending by suffix: `.sort.mdup.bam`**
- One BAM file per cell
- Sorted and indexed
  - If BAM files are not indexed, please use a writable folder in order that the pipeline generate itself the index `.bai` files
- Timestamp of index files must be newer than of the BAM files
- Each BAM file must contain a read group (`@RG`) with a common sample name (`SM`), which must match the folder name (`sampleName` above)

!!! note "Filtering of low-quality cells impossible to process"

    From Mosaicatcher version ≥ 1.6.1, a snakemake checkpoint rule was defined ([filter_bad_cells](https://github.com/friendsofstrandseq/mosaicatcher-pipeline/blob/master/workflow/rules/count.smk#L91)) to automatically remove from the analysis cells flagged as low-quality.

    The ground state to define this status (usable/not usable) is determined from column pass1 (enough coverage to process the cell) in mosaic count info file (see example below). The value of pass1 is not static and can change according the window size used (the larger the window, the higher the number of reads to retrieve in a given bin).

    Cells flagged as low-quality cells are listed in the following TSV file: `<OUTPUT>/cell_selection/<SAMPLE>/labels.tsv`.

#### Directory organisation - Current behavior

In its current flavour, MosaiCatcher requires that input data must be formatted the following way :

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

In a `Parent_Folder`, create a subdirectory `Parent_Folder/sampleName/` for each `sample`. Your Strand-seq BAM files of this sample go into the following folder:

- `bam` for the total set of BAM files

> Using the classic behavior, cells flagged as low-quality will only be determined based on coverage [see Note here](#note:-filtering-of-low-quality-cells-impossible-to-process).

#### Directory organisation - Old behavior

Previous version of MosaiCatcher (version ≤ 1.5) needed not only a `all` directory as described above, but also a `selected` folder (now renamed `bam`), presenting only high-quality selected libraries wished to be processed for the rest of the analysis.

You can still use this behavior by enabling the config parameter either by the command line: `=True` or by modifying the corresponding entry in the config/config.yaml file.

Thus, in a `Parent_Folder`, create a subdirectory `Parent_Folder/sampleName/` for each `sample`. Your Strand-seq BAM files of this sample go into the following folder:

- `all` for the total set of BAM files
- `selected` for the subset of BAM files that were identified as high-quality (either by copy or symlink)

Your `<INPUT>` directory should look like this:

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

> Using the `input_bam_legacy` parameter, cells flagged as low-quality will be determined both based on their presence in the `selected` folder presented above and on coverage [see Note here](#note:-filtering-of-low-quality-cells-impossible-to-process).


!!! warning 

    - Using the `input_bam_legacy` parameter, only **intersection** between cells present in the selected folder and with enough coverage will be kept. Example: if a library is present in the selected folder but present a low coverage [see Note here](#note:-filtering-of-low-quality-cells-impossible-to-process), this will not be processed.
    - `ashleys_pipeline=True` and `input_bam_legacy=True` are mutually exclusive
    - We advice to limit to the number of 2 the "_" characters in your file names (Ashleys-qc ML tool will react weidly with more than 3 "_")





The `--profile` argument will define whether you want to execute the workflow locally on your laptop/desktop (`workflow/snakemake_profiles/mosaicatcher-pipeline/v7/local/conda_singularity//`), or if you want to run it on HPC. A generic slurm profile is available (`workflow/snakemake_profiles/mosaicatcher-pipeline/v7/HPC/slurm_generic/`).

### Local execution (not HPC or cloud):

```bash
snakemake \
    --cores <N> \
    --config \
        data_location=<DATA_FOLDER> \
    --profile workflow/snakemake_profiles/mosaicatcher-pipeline/v7/local/conda_singularity/ \
    --singularity-args "-B /<disk>:/<disk>"
```


### HPC execution (SLURM)

HPC execution (require first that you modify the workflow/snakemake_profiles/mosaicatcher-pipeline/HPC/slurm_generic/config.yaml for your own cluster)

```bash
snakemake \
    --config \
        data_location=<DATA_FOLDER> \
    --profile workflow/snakemake_profiles/mosaicatcher-pipeline/v7/HPC/slurm_generic/ \
    --singularity-args "-B /<disk>:/<disk>"
```

!!! note "Current strategy to solve HPC job Out-of-Memory (OOM)"

    Workflow HPC execution usually needs to deal with out of memory (OOM) errors, out of disk space, abnormal paths or missing parameters for the scheduler. To deal with OOM, we are currently using snakemake restart feature that allows to automatically double allocated memory to the job at each attempt (limited to 6 for the moment). Then, if a job fails to run with the default 1GB of memory allocated, it will be automatically restarted tith 2GB at the 2nd attempt, 4GB at the 3rd, etc.

!!! note "EMBL HPC execution"

    In the EMBL HPC execution profile, disks binding points already set in the snakemake profile
    `--profile workflow/snakemake_profiles/mosaicatcher-pipeline/v7/HPC/slurm_EMBL/`

### Generate report (optional)

Optionally, it's also possible to generate a static interactive HTML report to explore the different plots produced by MosaiCatcher, using the same command as before and just adding at the end the following:
`--report <report>.zip --report-stylesheet workflow/report/custom-stylesheet.css`, which correspond to the following complete command:

```bash
snakemake \
    --config \
        data_location=<INPUT_FOLDER> \
    --singularity-args "-B /<mounting_point>:/<mounting_point>" \
    --profile workflow/snakemake_profiles/slurm_generic/ \
    --report <report>.zip --report-stylesheet workflow/report/custom-stylesheet.css
```

!!! note
    The zip file produced can be heavy (~1GB for 24 HGSVC samples ; 2000 cells) if multiple samples are processed in parallel in the same output folder.

## Modes of execution


## ArbiGent mode of execution

From 1.9.0, it's now possible to run MosaiCatcher in order to genotype a given list of positions specified in a bed file. To do so, `arbigent` config parameter need to be set to `True`.
Thus, an alternative branch of the pipeline will be executed instead of the classic one, targetting results produced by ArbiGent.

```bash
snakemake \
    --cores <N> \
    --config \
        data_location=<INPUT_DATA_FOLDER> \
        arbigent=True \
    --profile workflow/snakemake_profiles/local/conda_singularity/

```

A generic BED file is provided in `workflow/data/arbigent/manual_segmentation.bed`. A custom BED file can be specified by modifying `arbigent_bed_file` config parameter to the path of your choice.

!!! note
    If you modify the chromosome list to be processed (remove chrX & chrY for instance), a dedicated rule will extract only from the BED file, the rows matching the list of chromosomes to be analysed.

### scNOVA mode of execution

!!! warning
    Following Anaconda licensing issues with EMBL, scNOVA can only be executed using profiles based on singularity where the conda environment were already built.

From 1.9.2, it's now possible to run [scNOVA](https://github.com/jeongdo801/scNOVA/) directly from MosaiCatcher in order to determine the nucleosome occupancy associated to the SV calls provided during MosaiCatcher execution.

To do so, mosaicatcher must be executed during a first step in order to generate the SV calls and associated plots/tables required to determine subclonality.

Once the subclonality was determined, a table respecting the following template must be provided at this path `<DATA_LOCATION>/<SAMPLE>/scNOVA_input_user/input_subclonality.txt` where the different clones are named `clone<N>`:

| Filename             | Subclonality |
| -------------------- | ------------ |
| TALL3x1_DEA5_PE20406 | clone2       |
| TALL3x1_DEA5_PE20414 | clone2       |
| TALL3x1_DEA5_PE20415 | clone1       |
| TALL3x1_DEA5_PE20416 | clone1       |
| TALL3x1_DEA5_PE20417 | clone1       |
| TALL3x1_DEA5_PE20418 | clone1       |
| TALL3x1_DEA5_PE20419 | clone1       |
| TALL3x1_DEA5_PE20421 | clone1       |
| TALL3x1_DEA5_PE20422 | clone1       |

Once done, you can run the exact same command as previously and set `scNOVA` config parameter to `True` like the following:

```bash
snakemake \
    --cores <N> \
    --config \
        data_location=<INPUT_DATA_FOLDER> \
        scNOVA=True \
    --profile workflow/snakemake_profiles/local/conda_singularity/

```


### Multistep normalisation

From 2.1.0, it's now possible to use multistep normalisation (library size normalisation, GC correction, Variance Stabilising Transformation) resulting counts file, not only for visualisation purpose, but also to rely on it during the SV calling framework. To do so

```bash
snakemake \
    --cores <N> \
    --config \
        data_location=<INPUT_DATA_FOLDER> \
        multistep_normalisation=True \
        multistep_normalisation_for_SV_calling=True \
    --profile workflow/snakemake_profiles/local/conda_singularity/

```

### scTRIP multiplot

From 2.2.2, scTRIP multiplot (from Marco Cosenza) is now compatible with MosaiCatcher. The single requirement is to clone scTRIP multiplot repository (please reach out Marco if you want to access the repository, currently private) inside `workflow/scripts/plotting/scTRIP_multiplot`.

By default, scTRIP multiplot is set to false, to enable it, please update the `config/config.yaml` file or use `scTRIP_multiplot=True` in the config section of the command line. An example of a scTRIP multiplot is available [here](/docs/output.md#sctrip-multiplot-marco-cosenza)


!!! warning
    `multistep_normalisation_for_SV_calling` is mutually exclusive with `hgsvc_based_normalisation`

### IGV tracks upload

To upload IGV tracks generated by MosaiCatcher to the genome browser, you can follow the aforementionned steps:

* Start IGV
* Go to `File` > `Open Session`
* Navigate to the mosaicatcher output folder and then in `plots/IGV`
* Select the **<SAMPLE>_IGV_session.xml** file

The reference genome set during Mosaicatcher execution should automatically be loaded in IGV, as well as Watson/Crick and SV tracks for each cell.