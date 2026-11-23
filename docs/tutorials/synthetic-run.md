# A Federated Run on Synthetic Stores

This tutorial runs a whole federated run on one computer. A test router runs in Docker, and three sites named Lab A, Lab B and Lab C run from one starfish-site install, each with its own site folder and a synthetic sample store. Lab A coordinates and holds a synthetic test set. The run takes 3 rounds of 4 local epochs, which took about a minute and a half on an Ubuntu workstation with an RTX 4070 Ti.

Synthetic samples hold random numbers in the layout of real ones. The run therefore goes through every step of federated training, while its scores say nothing about the model.

## Before you start

- starfish-site installed as in [Quick Installation](../getting-started/quick-installation.md).
- Docker with Compose v2 for the test router.
- Four terminals, one for the commands and one for each site. The commands run in the `starfish-fl` checkout and keep all files in `~/bbfl-practice`.

## 1. Build the router image

The test router runs the `starfish-router` image, built from the checkout.

```bash
docker build -t starfish-router router
```

## 2. Start a test router

```bash
docker compose -f starfish-site/tests/e2e_router.yaml up -d --wait
```

Compose reports both containers healthy once the router answers.

```text
 Container starfish-site-e2e-postgres-1  Healthy
 Container starfish-site-e2e-router-1  Healthy
```

The router listens on `http://127.0.0.1:18100`, which only this computer reaches. Its admin account is `admin` with the password `1234`, a test setting for this router alone.

## 3. Write the synthetic stores

The site's own Python writes synthetic stores through a module of the controller. Each lab gets 20 samples from 5 synthetic subjects, with the 4 samples of one subject as its validation split. The test set gets 12 samples with region labels.

```bash
PY=~/.starfish-site/venv/bin/python
$PY -m starfish.controller.tasks.babel_brain_fno.synthetic ~/bbfl-practice/stores/lab-a \
    --groups 5 --per-group 4 --val-groups 1 --seed 1
$PY -m starfish.controller.tasks.babel_brain_fno.synthetic ~/bbfl-practice/stores/lab-b \
    --groups 5 --per-group 4 --val-groups 1 --seed 2
$PY -m starfish.controller.tasks.babel_brain_fno.synthetic ~/bbfl-practice/stores/lab-c \
    --groups 5 --per-group 4 --val-groups 1 --seed 3
$PY -m starfish.controller.tasks.babel_brain_fno.synthetic ~/bbfl-practice/stores/test-set \
    --groups 6 --per-group 2 --val-groups 0 --seed 4 --regions
```

Each command reports its count, `Wrote 20 synthetic samples` for a lab and `Wrote 12 synthetic samples` for the test set.

## 4. Enrol the three sites

A small shell function asks the test router for a code. Each site then enrols into its own folder through `--home`.

```bash
new_code() {
  curl -s -u admin:1234 -H 'Content-Type: application/json' -d "{\"note\": \"$1\"}" \
    http://127.0.0.1:18100/starfish/api/v1/enrolment-codes/ |
    python3 -c 'import json, sys; print(json.load(sys.stdin)["code"])'
}
starfish-site --home ~/bbfl-practice/lab-a enrol --router http://127.0.0.1:18100 --code "$(new_code 'Lab A')" \
    --name "Lab A" --store ~/bbfl-practice/stores/lab-a --eval-store ~/bbfl-practice/stores/test-set
starfish-site --home ~/bbfl-practice/lab-b enrol --router http://127.0.0.1:18100 --code "$(new_code 'Lab B')" \
    --name "Lab B" --store ~/bbfl-practice/stores/lab-b
starfish-site --home ~/bbfl-practice/lab-c enrol --router http://127.0.0.1:18100 --code "$(new_code 'Lab C')" \
    --name "Lab C" --store ~/bbfl-practice/stores/lab-c --device cpu
```

Lab A gets the test set as the coordinator, and `--device cpu` makes Lab C train on the CPU. Every enrolment confirms its site.

```text
Enrolled as site "Lab A", 277d2fd1-b944-4883-8e1d-d5d59d2985a7, router id 1.
Settings are in /home/lab/bbfl-practice/lab-a/site.env. Next run starfish-site check.
```

## 5. Check the coordinator

```bash
starfish-site --home ~/bbfl-practice/lab-a check
```

Every line should read `ok`, as in the full report on the [Check](../site/check.md#a-full-report) page. The synthetic test set lists its samples as train, and the gate scores all samples of a test set whatever their split.

## 6. Start the sites

Each site runs in its own terminal.

=== "Lab A"

    ```bash
    starfish-site --home ~/bbfl-practice/lab-a start
    ```

=== "Lab B"

    ```bash
    starfish-site --home ~/bbfl-practice/lab-b start
    ```

=== "Lab C"

    ```bash
    starfish-site --home ~/bbfl-practice/lab-c start
    ```

Each site logs `Polling the router. Press Ctrl-C to stop.` and waits for a run. A warning that the token travels over plain HTTP is expected with the test router.

## 7. Start a run

The test script `e2e_run.py` acts as the admin here. It creates a BabelBrainFno project with Lab A as the coordinator, adds Lab B and Lab C, starts a run of 3 rounds with 4 local epochs and follows it to the end.

```bash
python3 starfish-site/tests/e2e_run.py http://127.0.0.1:18100/starfish/api/v1 3 4 "Lab A" "Lab B" "Lab C"
```

```text
    0 s  [('Standby', 1)]
    3 s  [('Preparing', 1)]
    9 s  [('Running', 1)]
   16 s  [('Pending Success', 1)]
   19 s  [('Pending Aggregating', 1)]
   22 s  [('Aggregating', 1)]
   28 s  [('Standby', 2)]
   34 s  [('Preparing', 2)]
   37 s  [('Running', 2)]
   44 s  [('Pending Success', 2)]
   50 s  [('Pending Aggregating', 2)]
   53 s  [('Aggregating', 2)]
   59 s  [('Standby', 3)]
   62 s  [('Preparing', 3)]
   68 s  [('Running', 3)]
   75 s  [('Pending Success', 3)]
   78 s  [('Pending Aggregating', 3)]
   84 s  [('Aggregating', 3)]
   87 s  [('Success', 3)]
project 1 batch 1 succeeded after 87 s
```

Each line lists the states of the three sites' runs with their round. One entry means all three sites agree.

## 8. Read the results

The coordinator's log records the averaging, the gate's decision and the published model of every round. In `~/bbfl-practice/lab-a/logs/starfish-site.log` the first round ends with these lines.

```text
2026-11-23 11:57:22,272 INFO    logger-1-1-1: Downloaded 3 deltas
2026-11-23 11:57:27,119 INFO    starfish_site: Run 1: Aggregating
2026-11-23 11:57:27,561 INFO    logger-1-1-1: Gate: eval rel l2 133.342% to 101.223%, peak distance 8.495 to 7.835 mm on 12 samples; accepted
2026-11-23 11:57:27,563 INFO    logger-1-1-1: Round 1: aggregated 3 deltas, 48 samples; global model standin-250k-p1-b1-t1-r1 accepted
2026-11-23 11:57:27,623 INFO    logger-1-1-1: Published model 250k-v0001 for 250000 Hz, parent None, pending sign-off
```

The coordinator keeps each round's files under `~/bbfl-practice/lab-a/local/<run>/<task>/<round>/`, where `eval_report.json` holds the gate's full scores. [Reading the Gate and Approving a Model](gate-and-registry.md) explains them. The registry now lists three models that wait for sign-off.

```bash
curl -s -u admin:1234 "http://127.0.0.1:18100/starfish/api/v1/registry/?bucket_hz=250000&status=pending" |
    python3 -c 'import json, sys; [print(m["version"], m["parent"], m["status"]) for m in json.load(sys.stdin)]'
```

```text
250k-v0003 250k-v0002 pending
250k-v0002 250k-v0001 pending
250k-v0001 None pending
```

## 9. Clean up

Ctrl-C in each site's terminal stops the sites. The test router and all files of the tutorial then go.

```bash
docker compose -f starfish-site/tests/e2e_router.yaml down -v
rm -rf ~/bbfl-practice
```
