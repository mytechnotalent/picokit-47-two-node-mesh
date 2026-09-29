![picokit-47-two-node-mesh](https://raw.githubusercontent.com/mytechnotalent/picokit-47-two-node-mesh/main/picokit-47-two-node-mesh.png)

<br>

## FREE Reverse Engineering Self-Study Course [HERE](https://github.com/mytechnotalent/reverse-engineering)
## FREE Embedded Hacking Course [HERE](https://github.com/mytechnotalent/Embedded-Hacking)

<br>

# PICOKIT-47 TWO NODE MESH

### Two-Node Mesh Relay and Authenticated Heartbeat
#### Lesson 47 of the Picokit Series

<br>

***
**LEGAL DISCLAIMER:**
The information, tools, and code provided in this repository and course are strictly for educational, research, and defensive purposes only.

You are explicitly prohibited from using any materials contained herein to access, test, modify, or exploit any device, network, or system that you do not own 100% or for which you do not have explicit, documented, and legally binding authorization to interact with.

By using this repository and course, you acknowledge and agree that:

1. Any illegal, unauthorized, or malicious use of this information is solely your responsibility.
2. The author(s) and contributor(s) of this repository and course shall not be held liable for any damages, legal repercussions, criminal charges, or unauthorized actions resulting from the use, misuse, or abuse of the contents herein.
3. You will comply with all applicable local, state, national, and international laws regarding cybersecurity and computer fraud.

**IF YOU DO NOT AGREE WITH THESE TERMS, DO NOT USE THIS REPOSITORY AND COURSE.**
***

<br>
<br>

## Overview

The first capstone of the Picokit series. Two Pico 2 nodes share one LoRa
network: one is the origin node that measures and reports, and the other is a
relay and peer that hears the origin frame, authenticates it, re-seals it, and
forwards it to the hub. The relay still runs the standard LED chase and
authenticated heartbeat, and its heartbeat carries the running relay count so
the gateway can watch the mesh grow.

<br>

## What it teaches

- A two-node mesh: an origin node and a relay and peer on the same network.
- Authenticating an inbound peer envelope, then re-sealing and forwarding it.
- A heartbeat that reports the relayed count: `{"n":47,"s":<seq>,"r":<relayed>}`.
- The gateway side: receive, authenticate, reject, log, and display.

<br>

## Hardware

| Peripheral | Pico 2 pin | Role |
| --- | --- | --- |
| Red / Yellow / Green | GP16 / GP18 / GP17 | the chase |
| Onboard LED | GP25 | heartbeat, one blink per step |
| RYLR998 | GP8 TX / GP9 RX | mesh heartbeat and relay |
| Debug Probe | SWCLK/SWDIO/GND, GP0/GP1 | SWD and the console |

<br>

## How it works

The node runs `monitor_step` in a loop. It advances the chase every 2 seconds
and every 5 seconds it seals `{"n":47,"s":<seq>,"r":<relayed>}` with the field
key and sends it over LoRa. When an inbound `+RCV` line carries a peer envelope,
the relay opens it with the shared key and the sender byte as associated data,
re-seals the recovered body, forwards it to the hub, and increments the relay
count. The gateway authenticates every frame and only then parses it.

<br>

## Build and flash

```bash
cd firmware
cmake -S . -B build -G Ninja -DPICO_BOARD=pico2 -DPICO_PLATFORM=rp2350-arm-s
cmake --build build
openocd -f interface/cmsis-dap.cfg -f target/rp2350.cfg \
  -c "program build/picokit_47_two_node_mesh.elf verify reset exit"
```

<br>

## Watch the node

Open the console at 115200 and reset:

```text
BOOT
I2C scan:
  no devices
=== PICOKIT-47 TWO NODE MESH // LED CHASE + AUTHENTICATED HEARTBEAT ===
STEP 0 LED=RED seq=0
STEP 1 LED=YELLOW seq=1
STEP 2 LED=GREEN seq=2
RELAY from 0x002E
```

<br>

## The gateway

```bash
cd gateway
python3 -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt
python3 listen.py --port /dev/cu.usbserial-A50285BI --hub 0001 --network 18 --db gateway.db
```

It prints `OK node=47 rssi=...` per authenticated heartbeat. The terminal
dashboard `python3 tui.py --db gateway.db` and the web dashboard
`python3 web/app.py --db gateway.db` show the same rows.

<br>

## Verify

```bash
python3 .opencode/skill/embedded-c-standard/audit_c_standard.py
python3 .opencode/skill/embedded-python-standard/audit_python_standard.py
python3 .opencode/skill/iot-readme-standard/validate_readme.py
python3 .opencode/skill/iot-banner-standard/validate_banner.py
python3 scripts/run_tests.py
python3 scripts/check_coverage.py
```

<br>

# Next
[picokit-48-gateway-dashboard](https://github.com/mytechnotalent/picokit-48-gateway-dashboard)

<br>

# License
[MIT License](https://github.com/mytechnotalent/picokit-47-two-node-mesh/blob/main/LICENSE)
