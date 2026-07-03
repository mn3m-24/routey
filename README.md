# routey

A custom UDP-based traceroute implementation in Python. Sends probe packets with incrementing TTLs and parses ICMP Time Exceeded / Destination Unreachable responses to discover the network path to a target host.

## Features

- Raw IPv4/ICMP/UDP header parsing from byte buffers
- Configurable max TTL (30) and probe retries (3) per hop
- Concurrent probe sending (all ports per TTL sent immediately)
- `select()`-based ICMP socket polling with a configurable timeout
- Reverse DNS lookup on discovered router IPs
- Standard Cisco-style UDP destination port range (33434–33464)

## Requirements

- **Python ≥ 3.14**
- **Root / raw socket privileges** (except on macOS where unprivileged ICMP sockets are available)

## Usage

```bash
sudo python traceroute.py <hostname-or-ip>
```

Example:

```bash
sudo python traceroute.py google.com
```

Use `util.py` as a library to construct custom `Socket` instances
(e.g. `Socket.make_udp()` / `Socket.make_icmp()`).

## Project Structure

| File | Purpose |
|---|---|
| `traceroute.py` | Core traceroute logic, IPv4/ICMP/UDP header parsers, probe orchestration |
| `util.py` | Socket wrappers, DNS helpers, argument parsing, result formatting |

## How It Works

1. **Probe**: Send 3 UDP datagrams (dst port = 33434 + offset) per TTL.
2. **Listen**: Receive ICMP responses on a raw socket; responses embed the original IP/UDP headers.
3. **Validate**: Filter responses by ICMP type/code, destination IP, and expected UDP port to discard noise.
4. **Record**: On ICMP Time Exceeded → record the responder IP. On ICMP Port Unreachable → destination reached.
5. **Repeat** until destination responds or TTL 30 is reached.

## Limitations

- IPv4 only
- Requires raw socket capabilities
- Single-threaded; probes are sent in bulk, responses drained per TTL
