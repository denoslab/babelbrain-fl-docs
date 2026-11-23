# Quick Installation

These steps install starfish-site on one lab computer and test it. They need neither admin rights nor Docker, and the test at the end runs without a router.

## 1. Check the computer

| | Ubuntu | macOS |
| --- | --- | --- |
| System | Ubuntu 22.04 or later on x86_64 | macOS on Apple silicon, such as an M3 Pro or M4 |
| GPU | NVIDIA with driver 525 or later, and 570 or later for a Blackwell card such as the RTX PRO 6000 | The Apple GPU through MPS |
| Disk | About 8 GB | Less than on Ubuntu |
| Tools | `curl` and `git` | `curl` and `git` |

`nvidia-smi` shows the driver version in its header. A computer without an NVIDIA GPU trains on the CPU.

## 2. Clone the Starfish-FL repository

```bash
git clone -b feature/babelbrain-fl https://github.com/denoslab/starfish-fl.git
cd starfish-fl
```

The checkout holds every part of Starfish-FL.

```text
starfish-fl
├── controller       training and task code, which every site runs
├── router           the router, which runs on the main device
├── starfish-site    the installer and the starfish-site command
└── workbench        Docker setups for development
```

The site runs the code of this checkout. Moving or deleting the folder breaks the site.

## 3. Run the installer

```bash
starfish-site/install.sh
```

The installer puts everything into `~/.starfish-site`. It downloads uv, a Python installer, and sets up Python 3.11 in that folder, leaving the system Python untouched. Then it installs torch, as a build for CUDA 12.8 on Ubuntu with an NVIDIA GPU and as the standard build with MPS on a Mac, followed by the controller and starfish-site from the checkout. The download of torch takes most of the time.

The last lines sum up the install. On an Ubuntu workstation with an RTX 4070 Ti they read as follows.

```text
== Installed
torch 2.11.0+cu128 | CUDA build 12.8 | CUDA available True | MPS available False
starfish-site 0.1.0

Next steps
  1. starfish-site enrol --router <router URL> --code <code> --name "<site name>" --store <sample store>
  2. starfish-site check
  3. starfish-site start
```

On a Mac the second line ends with `MPS available True`. [Commands](../references/commands.md#installsh) lists the installer's options, such as `--cpu` for the smaller CPU-only build of torch.

## 4. Put the command on the PATH

The installer links `starfish-site` into `~/.local/bin` and says so when that folder is missing from `PATH`. One line in the shell's startup file adds it.

=== "Ubuntu"

    ```bash
    echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.bashrc
    source ~/.bashrc
    ```

=== "macOS"

    ```bash
    echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.zshrc
    source ~/.zshrc
    ```

Afterwards the command answers with its version.

```bash
starfish-site --version
```

```text
starfish-site 0.1.0
```

## 5. Test the computer

```bash
starfish-site check
```

Before enrolment the check covers torch and training only. It trains a small model for one epoch on the device the site will use and compares the model's output there with the CPU's. On the RTX 4070 Ti workstation it reported this.

```text
starfish-site 0.1.0 checks /home/lab/.starfish-site/site.env

No settings found. The check covers torch and training only until this site is enrolled, see starfish-site enrol --help.

Torch and device
  ok    torch 2.11.0+cu128, built for CUDA 12.8
  ok    cuda:0 NVIDIA GeForce RTX 4070 Ti, sm_89, 12 GB
  ok    This site trains on cuda

Training on this device
  ok    Output on cuda matches the CPU to 1.6e-04 of its peak
  ok    One epoch on 4 synthetic samples took 1.18 s, loss 1.3196

0 failed, 0 warnings
```

Each line reads `ok`, `warn` or `FAIL`, with a hint under each problem, and the last line counts both kinds of problem. On a Mac the check shows whether MPS can train the model, and [Check](../site/check.md#on-a-mac) explains what to do when it cannot. During the pilot, send the report to the admin.

## Next

[First Run](first-run.md) enrols the site with a code from the admin and joins a run.
