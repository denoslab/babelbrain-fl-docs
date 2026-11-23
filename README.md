# BabelBrain FL docs

Installation guides and tutorials for the BabelBrain FL testbed on [Starfish-FL](https://github.com/denoslab/starfish-fl), built with MkDocs and the Material theme.

## Build locally

The build needs Python 3.11 or later.

```bash
python3 -m venv .venv
.venv/bin/pip install -r requirements.txt
.venv/bin/mkdocs serve
```

The site then runs at http://127.0.0.1:8000 and reloads on every change. `.venv/bin/mkdocs build --strict` writes it to `site/` and fails on any warning.

## Publish

`.github/workflows/docs.yml` builds the site with `--strict` on every push to `main` and publishes it on GitHub Pages.
