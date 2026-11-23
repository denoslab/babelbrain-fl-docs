# Settings

## site.env

`site.env` in the site folder holds the controller's own environment variables. The task code therefore reads the same names on a site as in a container. [Change Settings](../site/change-settings.md) shows how to edit the file.

| Setting | Meaning |
| --- | --- |
| `SITE_UID` | The site's ID on the router |
| `ROUTER_URL` | The router API |
| `ROUTER_TOKEN` | The site's token from enrolment, to keep secret |
| `REQUESTS_CA_BUNDLE` | The CA file that signed the router's certificate |
| `BABELBRAIN_FL_STORE` | The site's sample store, read-only |
| `BABELBRAIN_FL_EVAL_STORE` | Coordinator only, the held-out test set for the evaluation gate |
| `BABELBRAIN_FL_DEVICE` | `cuda`, `cuda:N`, `mps` or `cpu`, which wins over the automatic choice and the project's device |
| `STARFISH_DISABLE_AGENTS` | `1` keeps all LLM agent code of Starfish-FL off, set by enrolment |
| `PYTORCH_ENABLE_MPS_FALLBACK` | `1` runs operations that MPS lacks on the CPU, a torch setting for Macs |

## Environment

| Variable | Meaning |
| --- | --- |
| `STARFISH_SITE_HOME` | The site folder for the installer and every command without `--home` |

## The site folder

| Path | Holds |
| --- | --- |
| `site.env` | The settings, readable by the owner only |
| `router-ca.pem` | The router's CA file, when enrolment got one |
| `local/` | Run files, logs and artifacts per run, task and round |
| `logs/` | `starfish-site.log`, rotated at 10 MB with 5 old files kept |
| `starfish-site.pid` | The running process, locked while it runs |
| `venv/`, `python/`, `bin/` | The installed software |
