# Check

`starfish-site check` tells whether a site is ready to train. It takes a few seconds and changes nothing.

```bash
starfish-site check
```

## What it checks

| Section | Checks |
| --- | --- |
| Settings | The site's ID, the router's URL and token, the CA file, the store paths and the device setting in `site.env` |
| Router | That the router answers, how fast, and which site it takes this one for |
| Sample stores | The usable samples per frequency bucket and split in each store, and any samples the store rejected |
| Torch and device | The torch build, each GPU with whether the build has kernels for it, and the device the site trains on |
| Training on this device | One epoch of the stand-in model on 4 synthetic samples, and how far its output on the device lies from the CPU's |

Before enrolment the check covers the last two sections only. `--quick` skips the training. Its synthetic samples go to a temporary folder that the check deletes afterwards.

## A full report

The coordinator of the [synthetic tutorial](../tutorials/synthetic-run.md) reported this on an Ubuntu workstation with an RTX 4070 Ti.

```text
starfish-site 0.1.0 checks /home/lab/bbfl-practice/lab-a/site.env

Settings
  ok    SITE_UID 277d2fd1-b944-4883-8e1d-d5d59d2985a7
  ok    ROUTER_URL http://127.0.0.1:18100/starfish/api/v1, plain HTTP on this machine
  ok    ROUTER_TOKEN is set
  ok    BABELBRAIN_FL_STORE /home/lab/bbfl-practice/stores/lab-a
  ok    BABELBRAIN_FL_EVAL_STORE /home/lab/bbfl-practice/stores/test-set

Router
  ok    Reached in 11 ms, this is site "Lab A", id 1

Sample stores
  ok    BABELBRAIN_FL_STORE has 250 kHz 16 train, 4 val, checked in 0.0 s
  ok    BABELBRAIN_FL_EVAL_STORE has 250 kHz 12 train, checked in 0.0 s

Torch and device
  ok    torch 2.11.0+cu128, built for CUDA 12.8
  ok    cuda:0 NVIDIA GeForce RTX 4070 Ti, sm_89, 12 GB
  ok    This site trains on cuda

Training on this device
  ok    Output on cuda matches the CPU to 1.6e-04 of its peak
  ok    One epoch on 4 synthetic samples took 1.20 s, loss 1.3196

0 failed, 0 warnings
```

Each line reads `ok`, `warn` or `FAIL`. A problem gets a hint on the line below it, and the last line counts failures and warnings. The command exits with status 1 when any check failed and with 0 otherwise.

A router on plain HTTP passes on the same computer only. Elsewhere the token would travel unencrypted, and the check gives a warning.

## On a Mac

A Mac trains on Apple's MPS. The model's complex FFTs have not run on MPS so far, and the training section is the test for them. When it fails, the hint offers two ways out.

```text
Training on this device
  FAIL  Training the stand-in on mps failed: ...
        Put BABELBRAIN_FL_DEVICE='cpu' in site.env to train on the CPU, or PYTORCH_ENABLE_MPS_FALLBACK='1' to run what MPS lacks on the CPU.
```

The first setting moves all training to the CPU. The second keeps MPS and runs only the operations MPS lacks on the CPU. [Change Settings](change-settings.md) shows how to add either to `site.env`. Before enrolment no `site.env` exists, and the setting goes in front of the command instead.

```bash
BABELBRAIN_FL_DEVICE=cpu starfish-site check
PYTORCH_ENABLE_MPS_FALLBACK=1 starfish-site check
```

## When a check fails

[Troubleshooting](../references/troubleshooting.md#check) lists every failure with what to do.
