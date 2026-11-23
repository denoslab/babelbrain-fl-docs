# Reading the Gate and Approving a Model

The coordinator scores every new model before the run keeps it, and a kept model waits in the model registry until someone signs it off. This page reads the gate's decisions from the [synthetic tutorial](synthetic-run.md) and describes the sign-off.

## The gate

After averaging the sites' updates, the coordinator runs both the current global model and the averaged candidate on all samples of its held-out test set. It compares them on six metrics.

| Metric | Name in the report |
| --- | --- |
| Relative l2 error of the field, in percent | `rel_l2_pct` |
| SSIM | `ssim` |
| PSNR, in dB | `psnr_db` |
| Distance between the true and the predicted pressure peak inside the brain, in mm | `peak_distance_mm` |
| Error of the peak amplitude, in percent | `peak_amplitude_error_pct` |
| Distance between the centroids of the true and the predicted -3 dB focal regions, in mm | `focal_centroid_distance_mm` |

Relative l2 error decides. A candidate whose mean relative l2 error rises by more than `max_rel_l2_increase`, 0 by default, is rejected, and the current model stays for the next round. The other five metrics are reported and never block, except that a project may set `max_peak_distance_increase_mm` to reject a candidate whose mean peak distance rises by more than that margin.

The coordinator's log sums up each decision in one line, here for the first round of the synthetic tutorial.

```text
Gate: eval rel l2 133.342% to 101.223%, peak distance 8.495 to 7.835 mm on 12 samples; accepted
```

## The report

Every round writes `eval_report.json` into the coordinator's round folder and uploads it to the router with the logs. The report of round 3, shortened, reads as follows.

```json
{
 "round": 3,
 "bucket_hz": 250000,
 "eval_samples": 12,
 "store_digest": "06baf78df5296620b8ee7fbefacb8005735ffab93d1a0b6e9b927964a01c003a",
 "current_model": "standin-250k-p1-b1-t1-r2",
 "margins": {"max_rel_l2_increase": 0.0, "max_peak_distance_increase_mm": null},
 "current": {"n": 12, "rel_l2_pct": {"mean": 100.0088, "std": 0.0105, "median": 100.0063}, "...": "..."},
 "candidate": {"n": 12, "rel_l2_pct": {"mean": 100.0072, "std": 0.0100, "median": 100.0068}, "...": "..."},
 "accepted": true,
 "reasons": [],
 "region_breakdown": {"P7": {"current": "...", "candidate": "..."}, "...": "..."}
}
```

`current` and `candidate` hold the mean, standard deviation and median of each metric. `reasons` names the rule a rejected candidate broke. When the test set's samples carry region labels, `region_breakdown` repeats the scores per target region, P7, P8, PO7, TP7, TP8 and other, while the decision rests on the overall means. `store_digest` fingerprints the test set and shows whether two reports scored the same samples.

## The model registry

A kept model enters the router's registry as the next version of its frequency bucket, such as `250k-v0003`, with the version before it as its parent. It stays pending until a reviewer approves or rejects it, and labs download approved versions only. Reviewers are the router's admin and the coordinator of the version's project. Every decision is final, and the router records who made it and when.

!!! note "Planned"
    The steps for signing off a model come with the admin command for projects and runs. Until then the models of a run stay pending, as in the synthetic tutorial.
