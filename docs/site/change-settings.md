# Change Settings

A site keeps its settings in `site.env` in the site folder. Enrolment writes the first ones, and the file is safe to edit while the site is stopped. A change takes effect at the next start.

## The file

Lab C of the [synthetic tutorial](../tutorials/synthetic-run.md) has this `site.env`, shown here with its token left out.

```bash
# starfish-site settings. Written by starfish-site enrol, safe to edit.

# This site, as the router knows it
SITE_UID='e1ed21a9-1fff-4351-8b5b-f7361cd50677'
# The router API
ROUTER_URL='http://127.0.0.1:18100/starfish/api/v1'
# This site's token from enrolment, SF-09. Keep it secret
ROUTER_TOKEN='...'
# This site's BabelBrain sample store, read-only
BABELBRAIN_FL_STORE='/home/lab/bbfl-practice/stores/lab-c'
# cuda, mps or cpu to override the automatic choice
BABELBRAIN_FL_DEVICE='cpu'
# 1 keeps all LLM agent code off on this site
STARFISH_DISABLE_AGENTS='1'
```

Each setting takes one line of the form `NAME='value'`. A value cannot hold a quote or a line break. [Settings](../references/settings.md) lists every setting.

## Train on another device

The automatic choice takes CUDA, then MPS, then the CPU. `BABELBRAIN_FL_DEVICE` wins over it and over the project's device setting.

```bash
BABELBRAIN_FL_DEVICE='cpu'
```

`cuda:1` picks the second GPU of a workstation with two. On a Mac where MPS lacks an operation of the model, `PYTORCH_ENABLE_MPS_FALLBACK='1'` keeps MPS and runs that operation on the CPU.

## Move the sample store

`BABELBRAIN_FL_STORE` names the store, and `BABELBRAIN_FL_EVAL_STORE` names the coordinator's test set. After a change `starfish-site check` counts the samples it finds there.

## Run two sites on one computer

Each site needs its own folder and its own enrolment. `--home` before the command picks the folder, and `STARFISH_SITE_HOME` picks it for every command of a shell.

```bash
starfish-site --home ~/site-two enrol --router https://192.168.1.20:8443 --code <code> --name "Lab B2" \
    --store /path/to/other/samples --ca-cert router-ca.pem
starfish-site --home ~/site-two start
```
