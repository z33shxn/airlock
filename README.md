# AirLock

Secure drone-to-base telemetry. Runner-up at HackSymmetric, a 48-hour state-level hackathon in Chennai, and later written up as a B.E. project at Chennai Institute of Technology (March 2026) with Naresh Kumar Raju, supervised by Dr. S. I. Shaik Hussain.

The full write-up is in [`AirLock_Project_Report.pdf`](AirLock_Project_Report.pdf).

## The problem

Drone telemetry covers position, altitude, speed and battery, and it usually travels over links that anyone nearby can listen to. Encryption alone doesn't stop an attacker from recording a valid packet and sending it again later, and a single long-lived key means one leak exposes everything.

## The design

AirLock splits the system across three machines so each part can be tested on its own:

| Role | What it does |
|---|---|
| **Sender (Laptop A)** | A drone simulator generates telemetry, and a Flask REST API encrypts and sends each packet with a unique message ID and timestamp. |
| **Receiver (Laptop B)** | Checks structure, rejects duplicates and replays using the message ID and a recent-packet cache, checks freshness and ordering, then decrypts, verifies and logs to SQLite behind a REST API. |
| **Dashboard (Laptop C)** | A Streamlit dashboard shows a live map, packet rate, security status and alerts. |

On the crypto side, ECC key pairs establish a shared secret and an AES session key is derived from it. Each packet is encrypted with AES-GCM and carries a SHA-256 hash and a message ID for integrity. The communication code changes every two seconds, which limits how long any single key is useful. A USB-based initialisation step gates the session before any telemetry flows.

## Results

In the three-laptop test, encryption added about 3 to 5 ms of average overhead per message. The attack tests covered replay, tampering, unauthorised access and jamming.

The report is honest about what's left:
- **Resource limits:** encryption and logging cost real cycles on very small drone boards.
- **Scale:** a single stream isn't a swarm.
- **Physical access:** a hardware trust module only helps if the hardware can't be reached.
