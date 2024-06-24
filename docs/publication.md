
## Publication full example

If you wish to reproduce results generated in the MosaiCatcher v2 publication, you can follow these different steps:

### Download and prepare data

As the complete data was above 50GB, data was stored into 2 separated zenodo repositories: https://zenodo.org/record/7696695 & https://zenodo.org/record/7697329. You can then first create a directory in a location with at least 300GB of free space, download the 3 tar.gz and extract them:

```
# Specify your location
YOUR_LOCATION=DEFINE_YOUR_PATH_HERE
# Create directory
mkdir -p $YOUR_LOCATION/mosaicatcher_publication_data/COMPRESSED
# Download tar.gz parts
wget https://zenodo.org/record/7696695/files/MOSAICATCHER_DATA_PAPER.tar.gz.part1?download=1 -P $YOUR_LOCATION/mosaicatcher_publication_data/COMPRESSED
wget https://zenodo.org/record/7697329/files/MOSAICATCHER_DATA_PAPER.tar.gz.part2?download=1 -P $YOUR_LOCATION/mosaicatcher_publication_data/COMPRESSED
wget https://zenodo.org/record/7697329/files/MOSAICATCHER_DATA_PAPER.tar.gz.part3?download=1 -P $YOUR_LOCATION/mosaicatcher_publication_data/COMPRESSED
# Extract 
for f in *part*; do tar -xvf "$f" -C $YOUR_LOCATION/mosaicatcher_publication_data; done
```

You can then verify checksum to make sure files were downloaded properly.

```
# Verify checksum
for f in C7 RPE-BM510 RPE1-WT LCL; do
    cd $YOUR_LOCATION/"$f"/fastq && md5sum -c *.md5 && cd $YOUR_LOCATION
done
```

If all the checks were successfull, you can now remove this directory before running the pipeline.


```
rm -r $YOUR_LOCATION/mosaicatcher_publication_data/COMPRESSED
```

### Run the pipeline

A generic slurm profile is provided in the the following directory: mosaicatcher-pipeline/workflow/snakemake_profiles/HPC/slurm_generic. Feel free to edit, create another profile specify to your cluster institute. 

```
snakemake \
    --config \
    data_location=$YOUR_LOCATION/mosaicatcher_publication_data \
    reference=T2T \
    split_qc_plot=True \
    ashleys_pipeline=True \
    multistep_normalisation=True \
    multistep_normalisation_for_SV_calling=False hgsvc_based_normalized_counts=True \
    --profile workflow/snakemake_profiles/HPC/slurm_generic/
```

### Generate the report 

```
snakemake \
    --config \
    data_location=$YOUR_LOCATION/mosaicatcher_publication_data \
    reference=T2T \
    split_qc_plot=True \
    ashleys_pipeline=True \
    multistep_normalisation=True \
    multistep_normalisation_for_SV_calling=False hgsvc_based_normalized_counts=True \
    --profile workflow/snakemake_profiles/HPC/slurm_generic/
    --report $YOUR_LOCATION/MosaiCatcher_V2_publication_data.report.zip --report-stylesheet workflow/report/custom-stylesheet.css
```

You can then extract the zip archive and open the `report.html` file to access the HTML report.