# Troubleshooting

Problems found so far are listed here with what to do. Anything else goes to the admin, together with the output of `starfish-site check` and the end of `~/.starfish-site/logs/starfish-site.log`.

## Installation

### starfish-site is not found

```text
starfish-site: command not found
```

The folder `~/.local/bin` is missing from `PATH`. Step 4 of [Quick Installation](../getting-started/quick-installation.md#4-put-the-command-on-the-path) adds it.

### The installer is missing

```text
bash: starfish-site/install.sh: No such file or directory
```

The checkout lacks starfish-site, which happens with a clone of another branch. Clone with `-b feature/babelbrain-fl` as in [Quick Installation](../getting-started/quick-installation.md#2-clone-the-starfish-fl-repository).

### The NVIDIA driver is too old

```text
install.sh: NVIDIA driver <version> is older than 525, which CUDA 12 builds need. Update the driver or pass --cpu.
```

Update the driver, to 570 or later for a Blackwell card such as the RTX PRO 6000. `--cpu` installs the CPU-only build of torch instead.

### The system is not supported

```text
install.sh: Linux on aarch64 is not supported, only x86_64
```

starfish-site runs on Ubuntu on x86_64 and on macOS. An Intel Mac installs and trains on the CPU, and the installer says so.

## Enrolment

### The code is refused

```text
The router refused the code. It may be mistyped, used or expired.
```

A code works once and expires after 72 hours by default. Ask the admin for a new one.

### The certificate is not trusted

```text
The router's HTTPS certificate is not trusted: ...
Pass the CA file from the router's admin with --ca-cert.
```

The router's certificate comes from the lab's own certificate authority. Enrol again with `--ca-cert router-ca.pem`, using the file from the admin.

### The router cannot be reached

```text
The router at https://192.168.1.20:8443/starfish/api/v1 cannot be reached: ...
```

Check the URL, that the computer is on the lab network and that the router runs. A computer on the wifi may sit on a different network from the main device, and the admin knows which networks reach it.

### The site is enrolled already

```text
This site is enrolled already, as 277d2fd1-b944-4883-8e1d-d5d59d2985a7. Pass --force to enrol it again with a new code.
```

`--force` with a new code enrols the site again and keeps its ID. A second site on the same computer needs its own folder, as [Change Settings](../site/change-settings.md#run-two-sites-on-one-computer) shows.

## Check

### The settings are incomplete

```text
  FAIL  No ROUTER_TOKEN
        Enrol first, see starfish-site enrol --help
```

The same hint follows a missing `SITE_UID` or `ROUTER_URL`. The site has not enrolled, or its `site.env` lost lines. Enrol with a new code, with `--force` when the site enrolled before.

### The router uses plain HTTP

```text
  warn  ROUTER_URL http://192.168.1.20:8000/starfish/api/v1 is plain HTTP
        The token travels unencrypted. Ask the router's admin for its https URL.
```

Only a router on the same computer may use plain HTTP. Ask the admin for the https URL and change `ROUTER_URL` in `site.env`.

### The router refuses the site

```text
  FAIL  The router refused this site's credentials (401)
        The token may be revoked. Ask the admin for a new enrolment code.
```

The admin revoked the token, or `site.env` holds a wrong one. Enrol again with a new code and `--force`. `The router does not know site` means the router has no site with this ID, for example after a reset of its database, and the same fix applies.

### A store path is wrong

```text
  FAIL  BABELBRAIN_FL_STORE /path/to/samples is not a folder
```

Correct the path in `site.env`. `warn  BABELBRAIN_FL_STORE is not set` means the path is missing, and a site without a store cannot train.

### A store holds no usable sample

```text
  FAIL  BABELBRAIN_FL_STORE holds no usable sample
```

The folder holds no store, or every sample failed its checks. A warning that starts with `rejected samples` counts the failed samples by reason, such as a SHA-256 that differs from the manifest.

### The GPU has no kernels

```text
  FAIL  cuda:0 <GPU name> is sm_XY, which this torch build has no kernels for
        Run install.sh again to get a torch build for CUDA 12.8.
```

The installed torch build predates the card. Running the installer again brings the build for CUDA 12.8.

### Training fails on a Mac

```text
  FAIL  Training the stand-in on mps failed: ...
        Put BABELBRAIN_FL_DEVICE='cpu' in site.env to train on the CPU, or PYTORCH_ENABLE_MPS_FALLBACK='1' to run what MPS lacks on the CPU.
```

MPS lacks an operation the model needs. [Check](../site/check.md#on-a-mac) explains both settings.

### The device's output differs from the CPU

```text
  FAIL  The model's output on mps differs from the CPU by ... of its peak
```

The device computes the model differently from the CPU, beyond a tolerance of 1e-3 of the output's peak. Train on the CPU with `BABELBRAIN_FL_DEVICE='cpu'` and send the report to the admin.

## Running

### The site runs already

```text
This site runs already, as process 1464637. Stop it first with starfish-site stop.
```

Another process runs this site folder, possibly in a terminal left open. `starfish-site stop` stops it.

### The site stops slowly

```text
Process 1464637 is still finishing the step in progress and stops when that ends. starfish-site stop --now cuts it short.
```

A training step can take minutes. Wait for it, or cut it short with `starfish-site stop --now`.

### A run fails in Preparing

The round's `logs.txt` and the site's log name the reason. `Only 16 train samples, fewer than min_samples 20` means the store holds fewer training samples than the project asks for, and `Sample store unavailable` means the store cannot be read. The admin can lower `min_samples` in the project, or the lab adds samples.

### The site shows as Disconnected

The router shows a site as Disconnected when the site stops or its heartbeat no longer arrives. `starfish-site check` on the site shows whether it reaches the router.
