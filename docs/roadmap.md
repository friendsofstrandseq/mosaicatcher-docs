# Development Roadmap

## Technical-related features

- [x] Zenodo automatic download of external files + indexes ([1.2.1](https://github.com/friendsofstrandseq/mosaicatcher-pipeline/releases/tag/1.2.1))
- [x] Multiple samples in the parent folder ([1.2.2](https://github.com/friendsofstrandseq/mosaicatcher-pipeline/releases/tag/1.2.2))
- [x] Automatic testing of BAM SM tag compared to sample folder name ([1.2.3](https://github.com/friendsofstrandseq/mosaicatcher-pipeline/releases/tag/1.2.3))
- [x] On-error/success e-mail ([1.3](https://github.com/friendsofstrandseq/mosaicatcher-pipeline/releases/tag/1.3))
- [x] HPC execution (slurm profile for the moment) ([1.3](https://github.com/friendsofstrandseq/mosaicatcher-pipeline/releases/tag/1.3))
- [x] Full singularity image with preinstalled conda envs ([1.5.1](https://github.com/friendsofstrandseq/mosaicatcher-pipeline/releases/tag/1.5.1))
- [x] Single BAM folder with side config file ([1.6.1](https://github.com/friendsofstrandseq/mosaicatcher-pipeline/releases/tag/1.6.1))
- [x] (EMBL) GeneCore mode of execution: allow selection and execution directly by specifying genecore run folder (2022-11-02-H372MAFX5 for instance) ([1.8.2](https://github.com/friendsofstrandseq/mosaicatcher-pipeline/releases/tag/1.8.2))
- [x] Version synchronisation between ashleys-qc-pipeline and mosaicatcher-pipeline ([1.8.3](https://github.com/friendsofstrandseq/mosaicatcher-pipeline/releases/tag/1.8.3))
- [x] Report captions update ([1.8.5](https://github.com/friendsofstrandseq/mosaicatcher-pipeline/releases/tag/1.8.5))
- [x] Clustering plot (heatmap) & SV calls plot update ([1.8.6](https://github.com/friendsofstrandseq/mosaicatcher-pipeline/releases/tag/1.8.6))
- [x] [`ashleys_pipeline_only` parameter](usage.md#ashleys-pipeline-only): using mosaicatcher-pipeline, trigger ashleys-qc-pipeline only and will stop after the generation of the counts, ashleys predictions & plots to allow the user manual reviewing/selection of the cells to be processed ([2.2.0](https://github.com/friendsofstrandseq/mosaicatcher-pipeline/releases/tag/2.2.0))
- [x] Target alternative execution ending: `breakpointr_only` parameter to stop the execution after breakpointR ; `whatshap_only` parameter to stop the execution after whatshap ([2.3.3](https://github.com/friendsofstrandseq/mosaicatcher-pipeline/releases/tag/2.3.3))
- [x] Snakemake v9 + Pixi migration: unified package management and reproducible environments ([2.4.0](https://github.com/friendsofstrandseq/mosaicatcher-pipeline/releases/tag/2.4.0))
- [x] Assembly-specific containers on GHCR: one image per reference genome (hg38, hg19, T2T, mm10, mm39) embedding the matching BSgenome R package ([2.4.0](https://github.com/friendsofstrandseq/mosaicatcher-pipeline/releases/tag/2.4.0))
- [x] Centralized version management with automated bumping (`pixi run bump-patch|bump-minor|bump-major|bump-beta`) and changelog generation ([2.4.0](https://github.com/friendsofstrandseq/mosaicatcher-pipeline/releases/tag/2.4.0))
- [x] HPC storage optimization: configurable `reference_base_dir` for multi-user reference genome sharing ([2.4.0](https://github.com/friendsofstrandseq/mosaicatcher-pipeline/releases/tag/2.4.0))
- [x] New EMBL HPC Apptainer profile: `workflow/snakemake_profiles/mosaicatcher-pipeline/v9/HPC/slurm_EMBL_apptainer/` ([2.4.0](https://github.com/friendsofstrandseq/mosaicatcher-pipeline/releases/tag/2.4.0))
- [ ] Plotting options (enable/disable segmentation back colors)

## Bioinformatic-related features

- [x] Self-handling of low-coverage cells ([1.6.1](https://github.com/friendsofstrandseq/mosaicatcher-pipeline/releases/tag/1.6.1))
- [x] Upstream [ashleys-qc-pipeline](https://github.com/friendsofstrandseq/ashleys-qc-pipeline.git) and FASTQ handle ([1.6.1](https://github.com/friendsofstrandseq/mosaicatcher-pipeline/releases/tag/1.6.1))
- [x] Change of reference genome (currently only GRCh38) ([1.7.0](https://github.com/friendsofstrandseq/mosaicatcher-pipeline/releases/tag/1.7.0))
- [x] Ploidy detection at the segment and the chromosome level: used to bypass StrandPhaseR if more than half of a chromosome is haploid ([1.7.0](https://github.com/friendsofstrandseq/mosaicatcher-pipeline/releases/tag/1.7.0))
- [x] inpub_bam_legacy mode (bam/selected folders) ([1.8.4](https://github.com/friendsofstrandseq/mosaicatcher-pipeline/releases/tag/1.8.4))
- [x] Blacklist regions files for T2T & hg19 ([1.8.5](https://github.com/friendsofstrandseq/mosaicatcher-pipeline/releases/tag/1.8.5))
- [x] [ArbiGent](usage.md#arbigent-mode-of-execution) integration: Strand-Seq based genotyper to study SV containly at least 500bp of uniquely mappable sequence ([1.9.0](https://github.com/friendsofstrandseq/mosaicatcher-pipeline/releases/tag/1.9.0))
- [x] [scNOVA](usage.md#scnova-mode-of-execution) integration: Strand-Seq Single-Cell Nucleosome Occupancy and genetic Variation Analysis ([1.9.2](https://github.com/friendsofstrandseq/mosaicatcher-pipeline/releases/tag/1.9.2))
- [x] [`multistep_normalisation` and `multistep_normalisation_for_SV_calling` parameters](usage.md#multistep-normalisation) to replace GC analysis module (library size normalisation, GC correction, Variance Stabilising Transformation) ([2.1.1](https://github.com/friendsofstrandseq/mosaicatcher-pipeline/releases/tag/2.1.1))
- [x] Strand-Seq processing based on mm10 assembly ([2.1.2](https://github.com/friendsofstrandseq/mosaicatcher-pipeline/releases/tag/2.1.2))
- [x] UCSC ready to use file generation including counts & SV calls ([2.1.2](https://github.com/friendsofstrandseq/mosaicatcher-pipeline/releases/tag/2.1.2))
- [x] `blacklist_regions` parameter: ([2.2.0](https://github.com/friendsofstrandseq/mosaicatcher-pipeline/releases/tag/2.2.0))
- [x] IGV ready to use XML session generation: ([2.2.2](https://github.com/friendsofstrandseq/mosaicatcher-pipeline/releases/tag/2.2.2))
- [x] BreakpointR integration through `breakpointr` parameter ([2.3.3](https://github.com/friendsofstrandseq/mosaicatcher-pipeline/releases/tag/2.3.3))
- [x] ashleys-qc-pipeline fully integrated (no longer a git submodule): preprocessing lives directly in mosaicatcher-pipeline ([2.4.0](https://github.com/friendsofstrandseq/mosaicatcher-pipeline/releases/tag/2.4.0))
- [x] mm39 full support: normalization files, blacklist regions, and BSgenome package for mouse GRCm39 assembly ([2.4.0](https://github.com/friendsofstrandseq/mosaicatcher-pipeline/releases/tag/2.4.0))
- [x] CanFam (canfam3/canfam4) framework-ready: reference infrastructure in place, normalization files pending ([2.4.0](https://github.com/friendsofstrandseq/mosaicatcher-pipeline/releases/tag/2.4.0))
- [x] Pre-built iGenomes index download (`download_prebuilt_indexes` parameter): skip local BWA index building by downloading from AWS iGenomes ([2.4.0](https://github.com/friendsofstrandseq/mosaicatcher-pipeline/releases/tag/2.4.0))
- [x] Ploidy estimation module (`ploidy` parameter): optional detection of haploid chromosomes/segments to guide StrandPhaseR ([2.4.0](https://github.com/friendsofstrandseq/mosaicatcher-pipeline/releases/tag/2.4.0))
- [x] `keep_ashleys_predictions` parameter: control retention of ashleys ML prediction files ([2.4.0](https://github.com/friendsofstrandseq/mosaicatcher-pipeline/releases/tag/2.4.0))
- [ ] Pooled samples

## Small issues to fix

- [x] replace `input_bam_location` by `data_location` (harmonization with [ashleys-qc-pipeline](https://github.com/friendsofstrandseq/ashleys-qc-pipeline.git))
- [x] List of commands available through list_commands parameter ([1.8.6](https://github.com/friendsofstrandseq/mosaicatcher-pipeline/releases/tag/1.8.6))
- [x] Move pysam / SM tag comparison script to snakemake rule ([2.2.0](https://github.com/friendsofstrandseq/mosaicatcher-pipeline/releases/tag/2.2.0))
- [x] Reference properly reference genome in IGV session script generation ([2.3.5](https://github.com/friendsofstrandseq/mosaicatcher-pipeline/releases/tag/2.3.5))
