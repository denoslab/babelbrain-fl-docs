# First Run

A site joins runs once it has enrolled with the router. This page goes from the details the admin hands out to the first round of a run, and each step links to the page with more.

## 1. Get the details from the admin

The admin of the main device gives each site three things.

- The router's URL, such as `https://192.168.1.20:8443`.
- A single-use enrolment code, valid for 72 hours unless the admin sets another time.
- The CA file `router-ca.pem`, when the router's certificate comes from the lab's own certificate authority.

The practitioner brings the path of the lab's sample store. The coordinator's practitioner also brings the path of the held-out test set.

## 2. Enrol

```bash
starfish-site enrol --router https://192.168.1.20:8443 --code <code> --name "Lab B" \
    --store /path/to/BabelBrainFL/samples --ca-cert router-ca.pem
```

The coordinator adds `--eval-store` with the test set. [Enrol](../site/enrol.md) explains every option and what enrolment writes.

## 3. Check again

```bash
starfish-site check
```

With the settings in place, the check also asks the router who this site is and counts the usable samples in each store. Every line should read `ok` before the first run, and [Check](../site/check.md) shows a full report.

## 4. Start the site

```bash
starfish-site start
```

The site polls the router every 5 seconds and waits for a run. It keeps its terminal busy, and the window stays open while the site takes part in runs.

## 5. Follow the first round

When the admin starts a run, the site's log follows it through each state. A participant of the [synthetic tutorial](../tutorials/synthetic-run.md) logged this first round.

```text
2026-11-23 11:57:02,131 INFO    starfish_site: Run 2: Standby
2026-11-23 11:57:07,123 INFO    starfish_site: Run 2: Preparing
2026-11-23 11:57:07,193 INFO    logger-2-1-1: Sample store at 250000 Hz: 16 train, 4 val
2026-11-23 11:57:08,313 INFO    logger-2-1-1: Round 1 starts from seed model init:0
2026-11-23 11:57:12,136 INFO    starfish_site: Run 2: Running
2026-11-23 11:57:13,983 INFO    logger-2-1-1: Epoch 1 of 4: loss 1.27488, 0.9 s
2026-11-23 11:57:14,140 INFO    logger-2-1-1: Epoch 2 of 4: loss 1.16769, 0.2 s
2026-11-23 11:57:14,295 INFO    logger-2-1-1: Epoch 3 of 4: loss 1.08438, 0.2 s
2026-11-23 11:57:14,442 INFO    logger-2-1-1: Epoch 4 of 4: loss 1.02607, 0.1 s
2026-11-23 11:57:14,575 INFO    logger-2-1-1: Round 1: 16 train samples, loss 1.02607, val rel l2 1.0123, [0.85, 0.16, 0.15, 0.15] s per epoch, peak memory 20 MB on cuda
2026-11-23 11:57:17,126 INFO    starfish_site: Run 2: Pending Success
2026-11-23 11:57:22,136 INFO    starfish_site: Run 2: Pending Aggregating
2026-11-23 11:57:27,119 INFO    starfish_site: Run 2: Aggregating
2026-11-23 11:57:32,137 INFO    starfish_site: Run 2: Standby
2026-11-23 11:57:32,204 INFO    logger-2-1-2: Round 2 starts from global model standin-250k-p1-b1-t1-r1
```

The run number belongs to the site, and every site of a run has its own. The coordinator's log adds the averaging, the gate's decision and the model it published, as [Reading the Gate and Approving a Model](../tutorials/gate-and-registry.md) shows.

## 6. Stop the site

Ctrl-C in the site's terminal stops it after the step in progress, and `starfish-site stop` does the same from another terminal. The admin says when a run is over, and [Start and Stop](../site/start-and-stop.md) covers the details.
