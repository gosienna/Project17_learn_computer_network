# Level 4: Network layer, IPv4 and ICMP (Tanenbaum ch. 5)

**Goal:** understand IP packets addressed to `10.0.0.2` and answer pings.

**Runs on:** the Linux box from level 3.

**Builds on:** `stack/netif.py`, `stack/eth/`, and `internet_checksum` from
`stack/link/checksum.py`.

**Current bite:** not started

## Bites

### Reading IP packets

- [ ] **4.1 IPv4 header, part 1**: `stack/ip/ipv4.py`
  `parse(packet)`: version and IHL (from the first byte), total length,
  protocol, source and destination addresses, and the payload (which starts
  at IHL × 4).
  *Observe:* a self-test on a captured ping packet (hex from `tcpdump -xx`)
  prints `v4 ihl=5 len=84 proto=1 10.0.0.1 → 10.0.0.2`.
- [ ] **4.2 IPv4 header, part 2**: `stack/ip/ipv4.py`
  Add identification, flags (DF, MF), fragment offset, TTL, and header
  checksum.
  *Observe:* the self-test shows `ttl=64 DF`, matching what `tcpdump -v` says.
- [ ] **4.3 Verify the header checksum**: `stack/ip/ipv4.py`
  Reuse `internet_checksum`. A correct header checksums to zero.
  *Observe:* the captured packet reports OK; change one byte and it reports
  BAD.
- [ ] **4.4 Build an IPv4 packet**: `stack/ip/ipv4.py`
  `build(src, dst, protocol, payload, ttl=64)`: a 20-byte header with the
  checksum filled in.
  *Observe:* `parse(build(...))` round-trips, and the checksum verifies.

### Hooking IP into the stack

- [ ] **4.5 Dispatch IPv4**: `stack/netif.py`
  EtherType `0x0800` → IP input. Drop the packet if the checksum is bad or
  the destination isn't `10.0.0.2`. Print one trace line per packet.
  *Observe:* during `ping 10.0.0.2`, the stack prints
  `[IP] ← 10.0.0.1 proto=1 len=84` once per second.
- [ ] **4.6 Drop fragments**: `stack/ip/ipv4.py`
  If MF is set or the fragment offset isn't zero, drop the packet and trace
  `fragment dropped`. Reassembly is out of scope.
  *Observe:* `ping -s 3000 10.0.0.2` (bigger than the 1500-byte MTU) produces
  `fragment dropped` lines.

### ICMP: answering ping

- [ ] **4.7 Parse ICMP echo**: `stack/ip/icmp.py`
  Type, code, checksum, identifier, sequence number, data.
  *Observe:* the trace shows `[ICMP] echo request id=… seq=1, 2, 3…`.
- [ ] **4.8 Build an echo reply**: `stack/ip/icmp.py`
  Type 0, with the same identifier, sequence and data, and a new checksum over
  the whole ICMP message.
  *Observe:* a self-test shows the reply's checksum verifies to zero.
- [ ] **4.9 IP output**: `stack/ip/ipv4.py`
  `output(dst_ip, protocol, payload, send_frame)`: look up the destination MAC
  in the ARP cache, build the IP packet, wrap it in Ethernet, and send.
  *Observe:* a manual call sends a packet that `tcpdump -i tap0` decodes
  correctly.
- [ ] **4.10 Answer pings**: `stack/ip/ipv4.py`
  In IP input, an ICMP echo request → build the reply → `output` it back to
  the sender.
  *Observe (finish line):* `ping -c 3 10.0.0.2` prints three replies with
  round-trip times, all answered by your code.
- [ ] **4.11 Play with TTL**: `stack/ip/ipv4.py`
  Send replies with `ttl=7`.
  *Observe:* `ping` shows `ttl=7`. Put it back to 64 afterwards.
- [ ] **4.12 Your own ping** (stretch): `tools/myping.py`
  Have your stack send echo requests to `10.0.0.1` and print each reply with
  its RTT.
  *Observe:* your tool's output looks like the real `ping`.

## Concepts covered

The IPv4 header layout, header checksum, TTL, protocol numbers, fragmentation
and MTU, how the destination address is checked, ICMP echo, and how an IP
packet is passed down to the link layer (where ARP supplies the destination
MAC).
