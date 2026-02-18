# ⚙️ Parameters

## MosaiCatcher arguments


!!! note "How to pass configuration arguments?"

    All these arguments can be specified in two ways:

    1. In the config/config.yaml file, by replacing existing values
    2. Using the `--config` snakemake argument (`--config` must be called only one time with all the arguments behind it, e.g: `--config input_bam_location=<INPUT> output_location=<OUTPUT> email=<EMAIL>`)


### General parameters

| Parameter | Comment                              | Default | Example |
| --------- | ------------------------------------ | ------- | ------- |
| `email`   | Email address for completion summary | None    | None    |

### Data location & Input/output options

<style>
table th:first-of-type {
    width: 30%;
}
table th:nth-of-type(2) {
    width: 30%;
}
table th:nth-of-type(3) {
    width: 10%;
}
table th:nth-of-type(4) {
    width: 20%;
}
</style>
| Parameter            | Comment                                                                                           | Parameter type | Default             |
| -------------------- | ------------------------------------------------------------------------------------------------- | -------------- | ------------------- |
| `data_location`      | Path to parent folder containing samples                                                          | String         | .tests/data_CHR17/  |
| `samples_to_process` | If multiple plates in the data_location parent folder, specify one or a comma-sep list of samples | None           | "[SampleA,SampleB]" |
| `publishdir`         | Path to backup location where important data is copied *(deprecated — will be removed in a future version)* | String | |

### Ashleys-QC upstream pipeline

| Parameter           | Comment                                                                                                                                                        | Parameter type | Default |
| ------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------- | ------- |
| `input_bam_legacy`  | Mutualy exclusive with ashleys_pipeline. Will use `selected` folder to identify high-quality libraries to process                                              | Boolean        | False   |
| `ashleys_pipeline`  | Allow to load and use [ashleys-qc-pipeline](/docs/usage.md#3c-fastq-input--preprocessing-module) snakemake preprocessing module and to start from FASTQ inputs | Boolean        | False   |
| `ashleys_threshold` | Threshold for Ashleys-qc binary classification                                                                                                                 | Float          | 0.5     |
| `bypass_ashleys`    | Set all cells as high-quality (labels to 1)                                                                                                                    | False          |         |  |
| `MultiQC`           | Enable or disable MultiQC analysis (includes FastQC, samtools flagstats & idxstats)                                                                            | Boolean        | False   |
| `hand_selection`    | Enable or disable hand selection through the Jupyter Notebook                                                                                                  | Boolean        | False   |
| `keep_ashleys_predictions` | Retain ashleys ML prediction files after pipeline completion                            | Boolean        | True    |
| `split_qc_plot`     | Enable or disable the split of QC plot into individual pages plots                                                                                             | Boolean        | False   |
| `paired_end`        | Enable or disable the use of paired-end data                                                                                                                   | Boolean        | False   |

### Reference data & Chromosomes

| Parameter                | Comment                             | Default (options)                            |
| ------------------------ | ----------------------------------- | -------------------------------------------- |
| `reference`              | Reference genome                    | hg38 (hg19, T2T, mm10, mm39)                 |
| `reference_base_dir`     | Base directory for reference files (allows multi-user sharing on HPC) | /g/korbel/shared/genomes |
| `chromosomes`            | List of chromosomes to be processed | Human: chr[1..22,X,Y], Mouse: chr[1..20,X,Y] |
| `chromosomes_to_exclude` | List of chromosomes to exclude      | []                                           |

!!! note "Reference genome support"

    - **Fully supported**: hg38, hg19, T2T, mm10, mm39
    - **Framework-ready** (normalization files pending): canfam3, canfam4
    - See [Reference Genomes](reference-genomes.md) for assembly-specific notes and caveats.

### BWA Index options

| Parameter                   | Comment                                                                                  | Default |
| --------------------------- | ---------------------------------------------------------------------------------------- | ------- |
| `download_prebuilt_indexes` | Download pre-built BWA indexes from AWS iGenomes instead of building them locally        | False   |

### Counts configuration

| Parameter           | Comment                                                                                             | Default |
| ------------------- | --------------------------------------------------------------------------------------------------- | ------- |
| `window`            | Window size used for binning by mosaic count (Can be of high importance regarding library coverage) | 100000  |
| `blacklist_regions` | Enable/Disable blacklisting                                                                         | True    |

### Optional modules


| Parameter                 | Comment                                                                                                                                       | Default |
| ------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------- | ------- |
| `multistep_normalisation` | Allow to perform [multistep normalisation](/docs/usage.md#multistep-normalisation) including GC correction for visualization (Marco Cosenza). | False   |
| `ploidy`                  | Enable ploidy estimation module to detect haploid chromosomes / segments                                                                      | True    |
| `breakpointR`             | Enable breakpointR module to compute breakpoints on Strand-Seq data (David Porubsky).                                                         | False   |

### Targeted execution

| Parameter               | Comment                                                                                        | Default |
| ----------------------- | ---------------------------------------------------------------------------------------------- | ------- |
| `ashleys_pipeline_only` | Stop the execution after ashleys-qc-pipeline submodule. Requires `ashleys_pipeline` to be True | Boolean | False |
| `breakpointR_only`      | Stop the execution after breakpointR submodule. Requires `breakpointR` to be True              | False   |
| `whatshap_only`         | Stop the execution after whatshap submodule (haplotagging of BAM files).                       | False   |



### SV calling parameters

| Parameter                                | Comment                                                                            | Default |
| ---------------------------------------- | ---------------------------------------------------------------------------------- | ------- |
| `multistep_normalisation_for_SV_calling` | Allow to use multistep normalisation count file during SV calling (Marco Cosenza). | False   |
| `hgsvc_based_normalized_counts`          | Use HGSVC based normalisation .                                                    | True    |

### SV calling algorithm processing options

| Parameter               | Comment                                                                                                    | Default  |
| ----------------------- | ---------------------------------------------------------------------------------------------------------- | -------- |
| `min_diff_jointseg`     | Minimum difference in error term to include another breakpoint in the joint segmentation (default=0.5)     | 0.1      |
| `min_diff_singleseg`    | Minimum difference in error term to include another breakpoint in the single-cell segmentation (default=1) | 0.5      |
| `additional_sce_cutoff` | Minimum gain in mismatch distance needed to add an additional SCE                                          | 20000000 |
| `sce_min_distance`      | Minimum distance of an SCE to a break in the joint segmentation                                            | 500000   |
| `llr`                   | Likelihood ratio used to detect SV calls                                                                   | 4        |

### Downstream analysis

| Parameter           | Comment                                                                                                                        | Default |
| ------------------- | ------------------------------------------------------------------------------------------------------------------------------ | ------- |
| `arbigent`          | Enable [ArbiGent](/docs/usage.md#arbigent-mode-of-execution) mode of execution to genotype SV based on arbitrary segments      | False   |
| `arbigent_bed_file` | Allow to specify custom ArbiGent BED file                                                                                      | ""      |
| `scNOVA`            | Enable [scNOVA](/docs/usage.md#scnova-mode-of-execution) mode of execution to compute Nucleosome Occupancy (NO) of detected SV | False   |
| `scTRIP_multiplot`  | Enable [scTRIP multiplot](/docs/usage.md#sctrip-multiplot) generation for all chromosomes of all cells                         | False   |

### EMBL specific options

| Parameter                 | Comment                                                                                               | Default                                     |
| ------------------------- | ----------------------------------------------------------------------------------------------------- | ------------------------------------------- |
| `genecore`                | Enable/disable genecore mode to give as input the genecore shared folder in /g/korbel/shared/genecore | False                                       |
| `genecore_date_folder`    | Specify folder to be processed                                                                        |                                             |
| `genecore_prefix`         | Specify genecore prefix folder                                                                        | /g/korbel/STOCKS/Data/Assay/sequencing/2023 |
| `genecore_regex_elements` | Specify genecore regex element to be used to distinguish sample from well number                      | PE20                                        |

If `genecore` and `genecore_date_folder` are correctly specified, each plate will be processed independently by creating a specific folder in the `data_location` folder.

### Execution profile

_Location_: workflow/snakemake_profiles/

| Parameter                               | Comment | Conda | Singularity | HPC | Local |
| --------------------------------------- | ------- | ----- | ----------- | --- | ----- |
| local/conda                             | /       | X     |             |     | X     |
| local/conda_singularity                 | /       | X     | X           |     | X     |
| HPC/slurm_generic (to modify)           | /       | X     |             | X   |       |
| HPC/slurm_EMBL (optimised for EMBL HPC) | /       | X     |             | X   |       |

