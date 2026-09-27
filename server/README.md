# Receiver server (runs on your machine)

`receive.py` listens on the LAN for events pushed by
[pi/ingest.py](../pi/ingest.py) and logs them into a local SQLite database,
`events.db`, right next to this script. This is the machine Claude Code
should query — the database never lives on the Pi.

No third-party packages needed, just the standard library.

## Setup

```sh
python3 receive.py                 # listens on 0.0.0.0:8765, db at ./events.db
```

Then find this machine's LAN IP (`ipconfig` on Windows) and pass
`http://<that-ip>:8765` as `--server` to `pi/ingest.py` on the Pi. Make sure
Windows Firewall allows inbound connections on the port you pick, or the Pi
won't be able to reach it.

Sanity-check it's up: `curl http://localhost:8765/health` should return `ok`.

## Schema

One `events` table, one row per event the Pi forwarded (the Pi already
filters out joystick jitter and unchanged ticks, so this is a meaningful
log, not a 20-rows/second dump):

```
id, ts, b1, b1n, b2, b2n, sw, swn, eb, ebn, enc, x, y, raw
```

`ts` is when *this machine* received the event (Unix timestamp, seconds).
`*n` columns are cumulative counts since the Nano last powered on. `raw` is
the full JSON line, in case a field isn't broken out into its own column
yet.

## Querying (e.g. from Claude Code)

```sh
sqlite3 events.db "select * from events order by id desc limit 20;"
sqlite3 events.db "select count(*) from events where b1n > 0;"
```
