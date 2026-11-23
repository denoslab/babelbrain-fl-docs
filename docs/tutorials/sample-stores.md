# Sample Stores from BabelBrain 0.8.8

!!! note "Planned"
    This tutorial comes with the exporter that turns BabelBrain 0.8.8 outputs into a sample store. Until then, runs use synthetic stores as in [A Federated Run on Synthetic Stores](synthetic-run.md).

## What it will cover

- Running BabelBrain 0.8.8 on a lab's subjects and keeping the outputs the exporter reads.
- Exporting those outputs into the lab's sample store.
- Checking the new store with `starfish-site check`.

## The sample store

A sample store is a folder that a site reads in place and never writes. It holds a manifest, `v1/manifest.jsonl`, with one line per sample, and one HDF5 file per sample under `v1/<bucket>/`, where the bucket is the frequency in Hz, 250000, 500000 or 750000. The site checks each file's SHA-256 against the manifest and leaves out samples that fail, and `starfish-site check` counts them with the reason.

Every site of the testbed runs BabelBrain 0.8.8. Fixes in 0.8.8 can change the fields BabelBrain computes for a subject, and samples from earlier versions therefore never share a training set with samples from 0.8.8.
