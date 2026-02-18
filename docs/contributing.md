# Contributing

Bug reports, feature requests, and pull requests are welcome on the [GitHub repository](https://github.com/friendsofstrandseq/mosaicatcher-pipeline).

---

## Adding a new reference genome

This section walks through every step required to add a new reference genome to MosaiCatcher. The pipeline uses a **centralized genome registry** in `config/config.yaml` — once your genome is registered there and its data files are generated, the pipeline handles everything else automatically.

### Overview of required steps

| Step | What | Where |
|------|------|--------|
| 1 | Register genome metadata | `config/config.yaml` |
| 2 | Add FASTA download rule | `workflow/rules/external_data.smk` |
| 3 | Generate bin BED file | one-time script |
| 4 | Generate GC matrix | one-time script |
| 5 | Generate normalization file | one-time, requires Strand-seq data |
| 6 | [Optional] Add segmental duplications | UCSC download |
| 7 | Test with dry-run | `snakemake --dry-run` |

---

### Step 1 — Register the genome in `config/config.yaml`

Add a new entry to the `references_data` dictionary. Every field is required.

```yaml
references_data:
  "myGenome":
    # FASTA and R package
    reference_fasta: "workflow/data/ref_genomes/myGenome.fa"
    R_reference: "BSgenome.Species.UCSC.myGenome"   # BioConductor package name
    custom_bsgenome_tarball: null                    # set path if not on BioConductor

    # Download source
    reference_file_location: "https://hgdownload.soe.ucsc.edu/goldenPath/myGenome/bigZips/myGenome.fa.gz"
    igenomes_base: null                              # AWS iGenomes URL or null

    # Segmental duplications
    segdups: "workflow/data/segdups/segDups_myGenome_UCSCtrack.bed.gz"
    snv_sites_to_genotype: []

    # Species metadata
    species: "Hsapiens"          # BSgenome species token (Hsapiens, Mmusculus, Cfamiliaris...)
    species_id: 9606             # NCBI Taxonomy ID
    common_name: "human"
    chromosome_count: 24

    # Chromosome list — must match chromosome_count
    chromosomes:
      - chr1
      - chr2
      # ...
      - chr22
      - chrX
      - chrY

    # Pre-computed data files (generated in steps 3-5)
    bin_bed_file: "workflow/data/myGenome.bin_200kb_all.bed"
    gc_matrix_file: "workflow/data/GC/myGenome.GC_matrix.txt.gz"

    # Module compatibility — set false if data/tools are unavailable
    supports_scnova: false               # true for human genomes only (hg38/hg19/T2T)
    supports_hgsvc_normalization: false  # true for hg38 only
    supports_arbigent: true
    supports_multistep_normalization: true
```

!!! note "Module flags"

    Set `supports_scnova: true` only for human genomes — scNOVA requires human gene annotations.
    Set `supports_hgsvc_normalization: true` only if you provide HGSVC normalization files for all three window sizes (50 kb, 100 kb, 200 kb).

---

### Step 2 — Add a FASTA download rule in `workflow/rules/external_data.smk`

```python
rule download_myGenome_reference:
    localrule: True
    input:
        storage.http(
            "https://hgdownload.soe.ucsc.edu/goldenPath/myGenome/bigZips/myGenome.fa.gz",
            keep_local=True,
        ),
    output:
        f"{REF_BASE_DIR}/myGenome.fa",
    log:
        f"{REF_BASE_DIR}/log/myGenome.ok",
    conda:
        "../envs/mc_base.yaml"
    shell:
        f"""
        mkdir -p "{REF_BASE_DIR}" "{REF_BASE_DIR}/log"
        mv {{input}} {REF_BASE_DIR}/myGenome.fa.gz
        gunzip {REF_BASE_DIR}/myGenome.fa.gz
        """
```

---

### Step 3 — Generate the bin BED file

The bin BED file can be derived directly from the normalization file (Step 5), since both share the same bin coordinates. Generate the normalization file first, then strip its last two columns:

```bash
# Normalization file format: chrom  start  end  scalar  class
# Bin BED format:            chrom  start  end  bin_name

tail -n +2 workflow/data/normalization/myGenome/HGSVC.200000.txt | \
  awk 'BEGIN{OFS="\t"} {print $1, $2, $3, "bin_200kb_"NR}' \
  > workflow/data/myGenome.bin_200kb_all.bed
```

Alternatively, if you need to generate the bin BED independently (before running `mosaicatcher count`), use the bundled script:

```bash
bash workflow/scripts/utils/generate_bin_bed.sh \
    workflow/data/ref_genomes/myGenome.fa \
    workflow/data/myGenome.bin_200kb_all.bed \
    200000
```

---

### Step 4 — Generate the GC content matrix

```bash
BED=workflow/data/myGenome.bin_200kb_all.bed
FASTA=workflow/data/ref_genomes/myGenome.fa

# Extract sequences per window
bedtools getfasta -fi $FASTA -bed $BED > myGenome.win.fa

# Download faCount (UCSC tool) if not available
wget https://hgdownload.cse.ucsc.edu/admin/exe/linux.x86_64/faCount
chmod +x faCount

# Count nucleotides per window
./faCount myGenome.win.fa | grep -v "^total" > myGenome_nt_content.txt

# Compute %GC and reformat
mkdir -p workflow/data/GC

(grep "^#" myGenome_nt_content.txt | \
   sed 's/#seq\tlen/chrom\tstart\tend/;s/$/\t%GC/';
 grep -v "^#" myGenome_nt_content.txt | awk '
   BEGIN { FS=OFS="\t" }
   { GC=$4+$5; tot=$3+$4+$5+$6; perc=tot ? 100*GC/tot : "NA"
     split($1, a, "[:-]") }
   { print a[1], a[2], a[3], $3, $4, $5, $6, $7, $8, perc }') | \
  bgzip > workflow/data/GC/myGenome.GC_matrix.txt.gz
```

---

### Step 5 — Generate the normalization / blacklist file

The normalization file is derived from the raw `mosaicatcher count` output of a control sample. The file format is identical to the count output **with the last column (normalization values) removed**.

**Requires:** one high-quality diploid Strand-seq sample with no clonal SVs.

```bash
mkdir -p workflow/data/normalization/myGenome

# 1. Run mosaicatcher count on a control sample
#    --do-not-blacklist-hmm keeps all bins in the output
mosaicatcher count --verbose --do-not-blacklist-hmm \
  -o control.txt.raw.gz -i control.info_raw \
  -x chroms_to_exclude.txt -w 200000 control_sample/*.bam

# 2. Strip the last column to produce the normalization file
zcat control.txt.raw.gz | \
  rev | cut -f2- | rev | \
  gzip > workflow/data/normalization/myGenome/HGSVC.200000.txt
```

!!! warning "Use an XX / diploid control for chrX"

    Use a **XX** (female) karyotype to avoid systematic chrX under-coverage artifacts — the same issue documented for mm39 (see [Reference Genomes](reference-genomes.md)).

Repeat for each window size you want to support (`50000`, `100000`, `200000`). Place files at:

```
workflow/data/normalization/myGenome/HGSVC.50000.txt
workflow/data/normalization/myGenome/HGSVC.100000.txt
workflow/data/normalization/myGenome/HGSVC.200000.txt
```

Then set `supports_hgsvc_normalization: true` in `config.yaml`.

---

### Step 6 — [Optional] Add segmental duplications

Segmental duplications are used to filter low-confidence SV calls. Download the UCSC genomicSuperDups track if available:

```bash
wget -O - "https://hgdownload.soe.ucsc.edu/goldenPath/myGenome/database/genomicSuperDups.txt.gz" | \
  zcat | tail -n +2 | \
  cut -f 2-4 | \
  gzip > workflow/data/segdups/segDups_myGenome_UCSCtrack.bed.gz
```

If no segmental duplications track exists for your genome, set `segdups: ""` in `config.yaml`.

---

### Step 7 — Test with dry-run

```bash
snakemake \
    --configfile .tests/config/simple_config_mosaicatcher.yaml \
    --config reference=myGenome data_location=<TEST_DATA> \
    --sdm conda \
    --dry-run
```

Check that the pipeline resolves the correct data files and that no "missing input" errors appear for your genome's required files.

---

### BSgenome R package

StrandPhaseR requires a BSgenome package matching your reference. For standard UCSC assemblies, the package is installed automatically from BioConductor using the `R_reference` field.

For assemblies not on BioConductor (e.g., T2T CHM13), build a custom package and point to it:

```yaml
R_reference: "BSgenome.Hsapiens.T2T.CHM13v2"
custom_bsgenome_tarball: "workflow/data/ref_genomes/BSgenome.T2T.CHM13.V2_1.0.0.tar.gz"
```

The pipeline installs it automatically from the tarball at runtime.
