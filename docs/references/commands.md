# Commands

## install.sh

```bash
starfish-site/install.sh [--cpu] [--home DIR] [--python PATH] [--no-link]
```

| Option | Effect |
| --- | --- |
| `--cpu` | CPU-only torch on Ubuntu, a smaller download for a computer without a GPU |
| `--home DIR` | Another site folder in place of `~/.starfish-site` |
| `--python PATH` | An existing Python 3.10 or 3.11 with venv and pip, used in place of uv |
| `--no-link` | No link in `~/.local/bin` |

`STARFISH_SITE_HOME` in the environment also names the site folder. `--help` prints the installer's own description.

## starfish-site

```bash
starfish-site [--home DIR] COMMAND [OPTIONS]
starfish-site --version
```

`--home` picks the site folder for any command and comes before the command. Without it the folder is `STARFISH_SITE_HOME`, or `~/.starfish-site` when that is unset. `starfish-site COMMAND --help` lists a command's options.

### enrol

Joins the router with a code from its admin, as [Enrol](../site/enrol.md) explains.

| Option | Meaning |
| --- | --- |
| `--router URL` | The router's URL, required |
| `--code CODE` | The single-use enrolment code, required |
| `--name NAME` | The site's name on the router, required |
| `--description TEXT` | A line about the site |
| `--store PATH` | The lab's BabelBrain sample store |
| `--eval-store PATH` | Coordinator only, the held-out test set |
| `--device DEVICE` | `cuda`, `mps` or `cpu`, which wins over the automatic choice |
| `--ca-cert FILE` | The CA file that signed the router's certificate |
| `--force` | Enrol again with a new code |

### check

Checks the settings, the router, the stores, torch and training, as [Check](../site/check.md) explains. It exits with status 1 when any check failed.

| Option | Meaning |
| --- | --- |
| `--quick` | Skips the training check |

### start

Runs the site until Ctrl-C, as [Start and Stop](../site/start-and-stop.md) explains.

| Option | Meaning |
| --- | --- |
| `--verbose` | Logs debug messages too |

### stop

Stops the running site after its step in progress. It exits with status 1 when the site is still finishing that step after the wait.

| Option | Meaning |
| --- | --- |
| `--now` | Cuts the step in progress short |
| `--wait SECONDS` | How long to wait for the site to stop, 30 by default |

## Synthetic stores

```bash
~/.starfish-site/venv/bin/python -m starfish.controller.tasks.babel_brain_fno.synthetic STORE \
    [--groups N] [--per-group N] [--val-groups N] [--bucket HZ] [--seed N] [--regions]
```

Writes a store of random samples in the layout of real ones, for practice runs and tests.

| Option | Meaning |
| --- | --- |
| `--groups N` | Synthetic subjects, 5 by default |
| `--per-group N` | Samples per subject, 4 by default |
| `--val-groups N` | Subjects whose samples form the validation split, 1 by default, with the rest in train |
| `--bucket HZ` | The frequency bucket, 250000, 500000 or 750000, with 250000 by default |
| `--seed N` | The random seed, 0 by default |
| `--regions` | Adds region labels, as a test set has them |
