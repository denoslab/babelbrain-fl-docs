# BabelBrain FL

BabelBrain FL fine-tunes the tFUS-FNO model on the samples of several labs at once. tFUS-FNO is a Fourier neural operator that predicts the transcranial ultrasound fields BabelBrain simulates. Every lab keeps its samples on its own computers, and the model learns from all of them through federated learning on [Starfish-FL](https://github.com/denoslab/starfish-fl). These pages set up the testbed on the university network and walk through a federated run, from installing a site to reading the evaluation gate.

<figure class="bbfl-figure">
<svg class="bbfl-diagram" viewBox="0 0 720 336" role="img" aria-labelledby="bbfl-testbed-title bbfl-testbed-desc">
<title id="bbfl-testbed-title">The BabelBrain FL testbed</title>
<desc id="bbfl-testbed-desc">A main device runs the router and PostgreSQL in Docker behind HTTPS. Three sites run starfish-site with their own sample stores and exchange weight updates with the router over HTTPS. Site A is the coordinator and also holds the held-out test set.</desc>
<defs>
<marker id="bbfl-arrow" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse"><path class="arrowhead" d="M0,0 L10,5 L0,10 z"/></marker>
</defs>
<rect class="box main" x="210" y="14" width="300" height="90" rx="10"/>
<text class="title" x="360" y="44">Main device</text>
<text x="360" y="68">Router and PostgreSQL in Docker</text>
<text x="360" y="89">HTTPS with the lab's own CA</text>
<line class="link" x1="120" y1="214" x2="298" y2="108" marker-start="url(#bbfl-arrow)" marker-end="url(#bbfl-arrow)"/>
<line class="link" x1="360" y1="214" x2="360" y2="108" marker-start="url(#bbfl-arrow)" marker-end="url(#bbfl-arrow)"/>
<line class="link" x1="600" y1="214" x2="422" y2="108" marker-start="url(#bbfl-arrow)" marker-end="url(#bbfl-arrow)"/>
<rect class="pill" x="260" y="148" width="200" height="26" rx="13"/>
<text class="note" x="360" y="165.5">weight updates over HTTPS</text>
<rect class="box coordinator" x="20" y="218" width="200" height="104" rx="10"/>
<text class="title" x="120" y="246">Site A, coordinator</text>
<text x="120" y="270">starfish-site</text>
<text x="120" y="291">sample store</text>
<text x="120" y="311">held-out test set</text>
<rect class="box" x="260" y="218" width="200" height="104" rx="10"/>
<text class="title" x="360" y="246">Site B</text>
<text x="360" y="270">starfish-site</text>
<text x="360" y="291">sample store</text>
<rect class="box" x="500" y="218" width="200" height="104" rx="10"/>
<text class="title" x="600" y="246">Site C</text>
<text x="600" y="270">starfish-site</text>
<text x="600" y="291">sample store</text>
</svg>
<figcaption>Samples and the test set stay on their site. Sites send the router weight updates, run status and logs, and the coordinator adds the gate's scores.</figcaption>
</figure>

## Components

### Main device

One workstation runs the Starfish-FL router with its PostgreSQL database. The router holds the projects, runs and models, moves the weight updates between the sites and keeps the model registry. [Main Device](main-device/introduction.md) describes its setup.

### Sites

Every lab computer that trains runs starfish-site, the native Starfish-FL site for Ubuntu and macOS. A site reads its lab's sample store in place, trains the current model on it and sends its weight update to the router. [Quick Installation](getting-started/quick-installation.md) sets one up.

### Coordinator

One site also holds a held-out test set. It averages the updates of all sites, scores the result on the test set and keeps a new model only when its error does not rise. Each kept model enters the registry and waits for a person to sign it off, as [Reading the Gate and Approving a Model](tutorials/gate-and-registry.md) explains.

## Getting Started

1. [Quick Installation](getting-started/quick-installation.md) installs a site and tests the computer, with no router involved.
2. [First Run](getting-started/first-run.md) enrols the site with a code from the admin and joins a run.
3. [A Federated Run on Synthetic Stores](tutorials/synthetic-run.md) runs a test router and three sites on one computer, for admins and anyone who wants to see a whole run.

## Status

!!! info "Pilot"
    The pilot runs on the university's LAN and wifi, with BabelBrain 0.8.8 at every site. starfish-site 0.1.0 has run on Ubuntu with an NVIDIA RTX 4070 Ti, and its first runs on a Mac and on an RTX PRO 6000 belong to the pilot. Runs use synthetic sample stores and a small stand-in model until the exporter for BabelBrain 0.8.8 outputs and the `tfus_fno` package are ready.
