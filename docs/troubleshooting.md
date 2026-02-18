# 🛠️ Troubleshooting & Limitations

## Current limitations

- Do not change the structure of your input folder after running the pipeline. The first execution builds a config dataframe file (`OUTPUT_DIRECTORY/config/config.tsv`) that contains the list of cells and their associated paths. Changes to the folder structure will cause issues on subsequent runs.

- Do not change the list of chromosomes between executions. For example, if you first run with `chr17` only, do not run the next execution on all chromosomes in the same output directory.
