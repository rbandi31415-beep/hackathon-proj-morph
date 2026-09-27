# Pi-side: forward Nano events to your machine

`ingest.py` reads the Nano's JSON status stream (same protocol as
`controller.py` and `status.html` — see
[nano_status/nano_status.ino](../nano_status/nano_status.ino)) over USB
serial and forwards each *changed* event over the LAN to
[server/receive.py](../server/receive.py), which is what actually stores
the data. The database lives on your machine, not the Pi, so Claude Code
can query it directly without touching the Pi.

No firmware changes needed: the Nano doesn't know or care what's on the
other end of the USB cable.

## Setup

```sh
python3 -m venv .venv && .venv/bin/pip install pyserial
ls /dev/ttyACM* /dev/ttyUSB*        # find the Nano's port
.venv/bin/python ingest.py --port /dev/ttyACM0 --server http://<your-machine-ip>:8765
```

Get `<your-machine-ip>` from `ipconfig` on your machine (an address like
`192.168.1.50`), and start [server/receive.py](../server/receive.py) there
*before* running this. Both devices need to be on the same LAN.

Only one program can hold the Nano's serial port at a time, same as on the
Mac: close `status.html` / `controller.py` first if they're pointed at this
port.

If the server is unreachable (network hiccup, server not started yet),
`ingest.py` prints a warning and keeps retrying — it does not buffer events
to disk, so anything that changes while it can't reach the server is lost.
