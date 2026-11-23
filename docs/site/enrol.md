# Enrol

Enrolment joins a site to the router with a code from the admin. It runs once per site folder and writes the site's settings.

## The command

```bash
starfish-site enrol --router https://192.168.1.20:8443 --code <code> --name "Lab B" \
    --store /path/to/BabelBrainFL/samples --ca-cert router-ca.pem
```

| Option | Meaning |
| --- | --- |
| `--router URL` | The router's URL. A bare host such as `https://192.168.1.20:8443` gets the API path `/starfish/api/v1` added |
| `--code CODE` | The single-use enrolment code from the admin |
| `--name NAME` | The site's name on the router, such as `"Lab B"` |
| `--description TEXT` | A line about the site |
| `--store PATH` | The lab's BabelBrain sample store |
| `--eval-store PATH` | Coordinator only, the held-out test set for the evaluation gate |
| `--device DEVICE` | `cuda`, `mps` or `cpu`, which wins over the automatic choice |
| `--ca-cert FILE` | The CA file that signed the router's certificate |
| `--force` | Enrol again with a new code |

`--router`, `--code` and `--name` are required, and a site that trains also needs `--store`.

## What enrolment writes

The router answers with an ID and a token for the site. Enrolment writes both into `~/.starfish-site/site.env`, together with the router's URL and the paths given, and only the owner can read that file. A CA file is copied into the site folder as `router-ca.pem`. The command then names the site and the file.

```text
Enrolled as site "Lab A", 277d2fd1-b944-4883-8e1d-d5d59d2985a7, router id 1.
Settings are in /home/lab/.starfish-site/site.env. Next run starfish-site check.
```

A code that the admin bound to a project also adds the site to that project, and the output gains a line that names the project.

The token stands for the site in every request to the router. It belongs in `site.env` only, and the admin revokes a token that leaked.

## The coordinator

The coordinator's site holds the held-out test set, on which the evaluation gate scores every new model. Its enrolment adds `--eval-store`.

```bash
starfish-site enrol --router https://192.168.1.20:8443 --code <code> --name "Lab A" \
    --store /path/to/BabelBrainFL/samples --eval-store /path/to/test-set --ca-cert router-ca.pem
```

## Enrol again

An enrolled site refuses a second enrolment.

```text
This site is enrolled already, as 277d2fd1-b944-4883-8e1d-d5d59d2985a7. Pass --force to enrol it again with a new code.
```

`--force` with a new code from the admin gives the site a new token and keeps its ID. Settings left out of the new command keep their values in `site.env`. The old token stays valid until the admin revokes it.

## When enrolment fails

| Message | What to do |
| --- | --- |
| `The router refused the code. It may be mistyped, used or expired.` | Ask the admin for a new code |
| `The router's HTTPS certificate is not trusted` | Pass the CA file from the admin with `--ca-cert` |
| `The router at ... cannot be reached` | Check the URL and that the computer is on the lab network |
| `The router URL needs a scheme, for example https://host:8443` | Start the URL with `https://` |

[Troubleshooting](../references/troubleshooting.md#enrolment) lists the rest.
