# StructRecon Code and Data Supplement

This anonymous package contains the implementation, configurations, environment specification, evaluation utilities, and dataset-format documentation supplied for review.

The full raw ICPARK-StructRecon sensor logs are not included because they exceed the submission system's archive limit. `Datasets/README.md` documents the expected dataset structure and modalities.

Quick checks:

```bash
conda env create -f environment.yml
conda activate structrecon
python -m compileall .
python run.py --help
```

No external repository is required to inspect the files in this archive.
