# Level 3: MAC sublayer / Ethernet (Tanenbaum ch. 4)

**Goal:** leave the simulated wire. Receive real Ethernet frames from the Linux
kernel through a TAP device, parse them, and answer ARP so the kernel learns
your stack's MAC address.

**Runs on:** a Linux VM (UTM) or Docker container with `/dev/net/tun` and the
`NET_ADMIN` capability. Python 3, `iproute2`, `tcpdump`, `ping`, `nc`, `curl`,
and `socat` are installed.

**Addresses used from here on:**

| Side | Interface | IP | MAC |
|---|---|---|---|
| Kernel (the "other machine") | `tap0` | `10.0.0.1` | whatever the kernel picks |
| Your stack | — | `10.0.0.2` | `02:00:00:00:00:02` (locally administered) |

**Current bite:** not started

## Bites

### Environment

- [ ] **3.1 A Linux box**: `Dockerfile` (or notes in `framework/env.md` if you
  use a VM)
  Base it on a small Debian/Ubuntu image, install the tools listed above, and
  mount the project folder. Run it with the TUN device and `NET_ADMIN`.
  *Observe:* inside the box, `/dev/net/tun` exists and `python3 --version`
  works.
- [ ] **3.2 Create tap0**: `tools/tap_up.sh`
  Use `ip tuntap` to add `tap0` in tap mode, bring it up, and give it
  `10.0.0.1/24`.
  *Observe:* `ip addr show tap0` lists `10.0.0.1/24`, and
  `ip route` shows `10.0.0.0/24 dev tap0`.

### Raw frames

- [ ] **3.3 Attach to the TAP device**: `stack/eth/tap.py`
  `open_tap("tap0")`: open `/dev/net/tun` and use `ioctl` with `TUNSETIFF`
  (`0x400454ca`) and flags `IFF_TAP | IFF_NO_PI` (`0x0002 | 0x1000`) to attach.
  Add `read_frame()` and `write_frame(data)`.
  *Observe:* a tiny loop prints `got N bytes` while you run `ping 10.0.0.2` in
  another shell. Those frames are ARP requests.
- [ ] **3.4 A sniffer**: `tools/sniff.py`
  Print every frame as a hex dump, 16 bytes per row.
  *Observe:* your dump matches `tcpdump -i tap0 -xx` byte for byte, except
  that the preamble and FCS never reach you.
- [ ] **3.5 Parse the Ethernet header**: `stack/eth/ethernet.py`
  `parse(frame)` returns destination MAC (6 bytes), source MAC (6),
  EtherType (2), and payload. Add `mac_str()` to format a MAC address.
  *Observe:* the sniffer prints lines like
  `ff:ff:ff:ff:ff:ff ← 3e:…  type=0x0806`.
- [ ] **3.6 Build an Ethernet frame**: `stack/eth/ethernet.py`
  `build(dst, src, ethertype, payload)`. Pad the result to the 60-byte minimum.
  *Observe:* `parse(build(...))` returns what you put in, and a tiny payload
  comes out 60 bytes long.
- [ ] **3.7 Is this frame for me?**: `stack/eth/ethernet.py`
  `MY_MAC = 02:00:00:00:00:02`. `classify(dst)` returns MINE, BROADCAST, or
  OTHER.
  *Observe:* the sniffer labels ARP requests BROADCAST.

### ARP: from IP address to MAC address

- [ ] **3.8 Parse ARP**: `stack/eth/arp.py`
  Fields: hardware type, protocol type, lengths, operation, sender MAC and IP,
  target MAC and IP.
  *Observe:* the sniffer prints `ARP who-has 10.0.0.2 tell 10.0.0.1`.
- [ ] **3.9 Build an ARP reply**: `stack/eth/arp.py`
  `build_reply(request, my_mac, my_ip)`: operation 2, swap sender and target,
  and fill in your MAC.
  *Observe:* parsing your reply prints `ARP 10.0.0.2 is-at 02:00:00:00:00:02`.
- [ ] **3.10 ARP cache**: `stack/eth/arp.py`
  Keep a table mapping IP → MAC, and learn the sender's pair from every ARP
  packet you see.
  *Observe:* after one ping attempt, printing the table shows `10.0.0.1 → 3e:…`.
- [ ] **3.11 The interface loop**: `stack/netif.py`
  Read a frame → parse Ethernet → if it's an ARP request for `10.0.0.2`,
  build the reply, wrap it in Ethernet, and `write_frame`. This file is the
  stack's main loop from now on.
  *Observe (finish line):* while `ping 10.0.0.2` runs,
  `ip neigh show dev tap0` shows `10.0.0.2 lladdr 02:00:00:00:00:02 REACHABLE`.
  Ping still gets no replies, and `tcpdump` shows ICMP requests now arriving
  for you. That's level 4.

## Concepts covered

MAC addresses (unicast, broadcast, locally administered), Ethernet frame
layout, EtherType, minimum frame size, the preamble and FCS (which the
hardware handles), ARP and ARP caches, and why IP needs a link-layer address
to deliver anything.
