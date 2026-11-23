# Start and Stop

## Start

```bash
starfish-site start
```

The site polls the router for runs every 5 seconds and reports itself alive every 5 seconds. It works on its runs one step at a time and logs to the terminal and to `~/.starfish-site/logs/starfish-site.log`. The first lines name the site, the router, the store and the folders it uses.

```text
2026-11-23 11:56:52,098 INFO    starfish_site: starfish-site 0.1.0, site 43f7cf1a-7b04-4b89-8d4d-654eed33274a, router http://127.0.0.1:18100/starfish/api/v1
2026-11-23 11:56:52,098 INFO    starfish_site: Store /home/lab/bbfl-practice/stores/lab-b, device auto, run files in /home/lab/bbfl-practice/lab-b/local, log in /home/lab/bbfl-practice/lab-b/logs/starfish-site.log
2026-11-23 11:56:52,102 INFO    starfish_site: Polling the router. Press Ctrl-C to stop.
```

`--verbose` adds debug messages. starfish-site 0.1.0 has no background service, and the site runs as long as its terminal window stays open.

## Stop

| Way | Effect |
| --- | --- |
| Ctrl-C in the site's terminal | Stops after the step in progress |
| Ctrl-C a second time | Stops at once and cuts the step short |
| `starfish-site stop` in another terminal | Stops after the step in progress and waits up to 30 seconds for it |
| `starfish-site stop --now` | Cuts the step in progress short |

`stop` reports `Stopped process` with the process number. `--wait SECONDS` changes how long it waits, and when the step outlasts the wait, the command says so and the site stops once the step ends.

A stopped site shows as Disconnected on the router. When it starts again, it picks up its active runs where they stand.

## One process per site

Only one process can run a site folder. A second `start` on the same folder exits with the first one's process number.

```text
This site runs already, as process 1464637. Stop it first with starfish-site stop.
```

## Logs and run files

`logs/starfish-site.log` in the site folder holds the site's log, rotated at 10 MB with 5 old files kept. Each run keeps its files under `local/<run>/<task>/<round>/`, with the round's log in `logs.txt`. The coordinator's round folders also hold `eval_report.json` with the gate's scores. [Settings](../references/settings.md#the-site-folder) lists the whole folder.
