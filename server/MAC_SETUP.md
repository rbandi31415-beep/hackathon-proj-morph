# Running the receiver server on a Mac (direct Ethernet link)

This is an alternative to running `receive.py` on Windows: the same server,
just on a Mac, connected to the Pi over a direct Ethernet cable instead of
the campus WiFi (which blocks device-to-device TCP connections — see the
project history for why a direct link is needed).

Static IPs are used since there's no DHCP server on a direct link:
`192.168.50.1` for this machine, `192.168.50.2` for the Pi.

## On the Mac

**1. Get the code:**
```sh
git clone https://github.com/rbandi31415-beep/hackathon-proj-morph.git
cd hackathon-proj-morph/server
```

**2. Plug in the Ethernet cable** (Mac to Pi).

**3. Set a static IP on the Mac's Ethernet interface:**
- System Settings -> Network -> click the Ethernet adapter -> **Details** -> **TCP/IP**
- Set "Configure IPv4" to **Manually**
- IP Address: `192.168.50.1`
- Subnet Mask: `255.255.255.0`
- Leave Router blank
- Click OK / Apply

**4. Allow incoming connections through the Mac firewall** (if it's on): System Settings -> Network -> Firewall -> either turn it off for testing, or allow incoming connections for Python when prompted the first time you run the server.

**5. Start the receiver** (no dependencies needed - stdlib only):
```sh
python3 receive.py
```
You should see `Listening on 0.0.0.0:8765, logging to events.db.`

## Back on the Pi

Nothing changes except which machine it's pointed at - assuming the Pi
already has a static IP of `192.168.50.2` on its Ethernet port:
```sh
ping -c 4 192.168.50.1
.venv/bin/python pi/ingest.py --port /dev/ttyUSB0 --server http://192.168.50.1:8765
```

If the Pi doesn't have that static IP set yet:
```sh
sudo ip addr add 192.168.50.2/24 dev eth0
sudo ip link set eth0 up
```

## Verify on the Mac
```sh
sqlite3 events.db "select * from events order by id desc limit 5;"
```

Mac Ethernet adapters (especially USB-to-Ethernet dongles) sometimes show up
under a different name than "Ethernet" in Network Settings, e.g.
"Thunderbolt Ethernet" or "USB 10/100/1000 LAN" - use whichever one shows
"Cable Connected" once the Pi is plugged in.
