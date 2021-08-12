<h1 id="dataset-curation" align="center">
💾 Dataset curation
</h1>

These are the tools for curating public Nextclade datasets.

> ⚠️ This functionality is meant for the maintainers of Nextclade. We do not support using this outside of the team, but you can try anyways (you will need your own server infrastructure for that).


> ⚠️ Note that the data is shared among all users of Nextclade, including CLI and Web users. Data corruption or incorrect format or content of JSON files will break Nextclade for everyone on the planet.



<h2 id="dataset-curators-guide" align="center">
🧑💾 Dataset curator's guide
</h2>

Dataset curation process is automated using GitHub Actions. The GitHub Actions workflow is described
in [.github/workflows/data-curation.yml](.github/workflows/data-curation.yml). It runs on every push to the repository.

The result of the curation process produces complete fresh data repository, including new index file, and uploads it to remote server, so that the data becomes available for Nextclade users.


### Data source

View the contents of the `data/` directory. It contains source data for creating datasets repository.
During rebuild, some files copied to datasets repository as is, others are modified of generated on the fly. 

 ```
 data
 ├── flu_vic_ha
 │   ├── dataset.json
 │   └── versions
 │       └── 2021-08-11T19:47:59Z
 │           └── files
 │               ├── genemap.gff
 │               ├── metadata.json
 │               ├── primers.csv
 │               ├── qc.json
 │               ├── reference.fasta
 │               ├── sequences.fasta
 │               └── tree.json
 ├── sars-cov-2
 │   ├── dataset.json
 │   └── versions
 │       ├── 2021-06-20T00:00:00Z
 │       │   └── files
 │       │       ├── genemap.gff
 │       │       ├── metadata.json
 │       │       ├── primers.csv
 │       │       ├── qc.json
 │       │       ├── reference.fasta
 │       │       ├── sequences.fasta
 │       │       └── tree.json
 │       └── 2021-06-25T00:00:00Z
 │           └── files
 │               ├── genemap.gff
 │               ├── metadata.json
 │               ├── primers.csv
 │               ├── qc.json
 │               ├── reference.fasta
 │               ├── sequences.fasta
 │               └── tree.json
 └── settings.json
 ```


 - The file `settings.json` contains global settings, for all datasets.
 - Each directory contains a dataset
 - Each dataset is described by a `dataset.json` file. The `name` property of the `dataset.json` should match the directory name of the dataset.
 - Each dataset contains multiple versions, each under `versions/` subdirectory
 - Each dataset version is identified by a tag and described by `metadata.json` file.
 - Each dataset version contains a set of files in `files/` subdirectory. The `files` property in `metadata.json` should list these files.
 - A dataset version is uniquely identified by a tag (`datetime` currently)
 - A dataset version directory name should match the tag


### Adding new dataset

- Add a directory `data/<dataset_name>`, where `<dataset_name>` is the unique dataset name.
- Add `dataset.json` file. Use `dataset.json` files in existing datasets as a reference.
- Proceed to adding a dataset version as described in [Adding a version of the dataset](#adding-a-version-of-the-dataset).

### Adding a version of the dataset
- Add a directory `data/<dataset_name>/<version_tag>`, where `<version_tag>` is the current datetime. Datetime precision is not required, write manually or try this and tweak slightly to match existing tags:
     ```bash
     echo "$(date --iso-8601=seconds)Z"
     echo "$(date +"%Y-%m-%dT%H:%M:%S")Z"
     echo "python3 -c "import datetime; print(datetime.datetime.now().isoformat())"
   2021-08-13T01:03:28.052350"
     ```
- Add `metadata.json` describing the new version tag. If there are breaking changes, adjust `compatibility` fields accordingly.
- Add files under `files/` in this subdirectory. File list should match the `files` entry in `metadata.json`.
- Rebuild locally by running `./scripts/rebuild`
- If the result is satisfactory, commit files to git. This will launch the automated rebuild and will upload the resulting fresh data repo to the remote server.
- Wait up to 5-10 minutes for Cloudfront cache to be updated
- Verify that the modified data is available in Nextclade Web and Nextclade CLI

### Removing a version
 - Remove the corresponding version directory and commit to git. On rebuild, the version will be removed from the index. The scripts do not actually remove the files from S3 bucket, to avoid accidental data loss. We might clean up the bucket manually periodically. 

### Removing a dataset
 - Remove `dataset.json` corresponding to this dataset (optionally, the dataset directory too) and commit to git.

> ⚠️ Once committed and pushed to GitHub, all the files, source and generated become public. If any sensitive information was exposed, or if something needs to be deleted, ask someone with direct access to AWS to delete manually.
