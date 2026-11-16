# Level 5: Transport layer, UDP and minimal TCP (Tanenbaum ch. 6)

**Goal:** add ports and connections. First UDP, which adds very little on top
of IP. Then a small but real TCP: handshake, sequence and ACK numbers, data in
both directions, clean close, and resets.

**Runs on:** the Linux box from level 3.

**Scope:** in-order delivery on a clean link. No congestion control,
retransmission timer, or window scaling, except as stretch bites.

**Current bite:** not started

Book links open the PDF at the given page. Page numbers are PDF pages.

## Bites

### Shared helper

> **Read first:** [§6.5.4 The TCP Segment Header, p. 411](../reference/Computer%20Networks%20%28Andrew%20S.%20Tanenbaum.pdf#page=411),
> Fig. 6-30, the pseudoheader. UDP uses the same pseudoheader (RFC 768), even
> though the book only shows it for TCP.

- [ ] **5.1 Pseudo-header checksum**: `stack/ip/ipv4.py`
  `pseudo_checksum(src, dst, protocol, segment)`: put the 12-byte
  pseudoheader from Fig. 6-30 (source address, destination address, a zero
  byte, protocol, segment length) in front of the segment, then use
  `internet_checksum`. UDP and TCP both need it.
  *Observe:* a captured UDP datagram (from `tcpdump -xx`) verifies to zero.

### UDP

> **Read first:** [§6.4.1 Introduction to UDP, pp. 401–402](../reference/Computer%20Networks%20%28Andrew%20S.%20Tanenbaum.pdf#page=401):
> Fig. 6-23 is the 8-byte header. P. 402 explains replying by swapping ports,
> and that a checksum of 0 means "not computed", so a computed 0 is sent as
> all ones.

- [ ] **5.2 Parse UDP**: `stack/transport/udp.py`
  Source port, destination port, UDP length, UDP checksum, and data.
  *Observe:* a self-test on the captured datagram prints `:54321 → :7 "hi"`.
- [ ] **5.3 Dispatch protocol 17**: `stack/ip/ipv4.py`
  IP input passes UDP (protocol 17) to `udp.input`, which prints a trace line.
  *Observe:* `echo hi | nc -u -w1 10.0.0.2 7` prints
  `[UDP] 10.0.0.1:… → :7 "hi"`.
- [ ] **5.4 UDP echo on port 7**: `stack/transport/udp.py`
  Build a reply the way p. 402 describes: the incoming source port becomes
  the outgoing destination port. Compute the checksum (send all ones if it
  comes out as 0), and send it through `ipv4.output`.
  *Observe:* `nc -u 10.0.0.2 7` echoes every line you type.
- [ ] **5.5 Port table**: `stack/transport/udp.py`
  A dict mapping port → handler. Echo becomes the handler registered on
  port 7.
  *Observe:* echo still works, and a datagram to port 9999 is traced
  `no listener`.
- [ ] **5.6 Port unreachable**: `stack/ip/icmp.py`
  For a datagram with no listener, send DESTINATION UNREACHABLE (Fig. 5-61;
  type 3, code 3 for "port"). It carries the original IP header plus the first
  8 bytes of the datagram.
  *Observe:* `nc -u 10.0.0.2 9999` reports `Connection refused` after the
  second line.

### TCP: reading segments

> **Read first:** [§6.5.1–6.5.3, pp. 407–410](../reference/Computer%20Networks%20%28Andrew%20S.%20Tanenbaum.pdf#page=407)
> (TCP's byte-stream model and the fact that every byte has a sequence
> number), then
> [§6.5.4 The TCP Segment Header, pp. 410–412](../reference/Computer%20Networks%20%28Andrew%20S.%20Tanenbaum.pdf#page=410),
> Fig. 6-29, with the field-by-field text: ports, sequence number,
> acknowledgement number, header length, the six flags (URG, ACK, PSH, RST,
> SYN, FIN), window size, checksum.

- [ ] **5.7 Parse TCP**: `stack/transport/tcp.py`
  The Fig. 6-29 fields: ports, sequence number, acknowledgement number, TCP
  header length, flags, window size, checksum, and data. Add a flag formatter
  such as `S`, `SA`, `PA`, or `FA`.
  *Observe:* a self-test on a captured SYN prints `S seq=…`.
- [ ] **5.8 Dispatch protocol 6**: `stack/ip/ipv4.py`
  IP input passes TCP (protocol 6) to `tcp.input`, which verifies the checksum
  and prints a trace line.
  *Observe:* `nc 10.0.0.2 7` prints `[TCP] S seq=…` and then the same SYN
  again and again. That's the client retransmitting, because nobody answers.
- [ ] **5.9 Build TCP segments**: `stack/transport/tcp.py`
  `build(src_port, dst_port, seq, ack, flags, data, window)`, with the
  pseudo-header checksum.
  *Observe:* `tcpdump -v` decodes a manually sent segment and reports
  `cksum … (correct)`.

### TCP: connections

> **Read first:** [§6.2.2 Connection Establishment, pp. 379–383](../reference/Computer%20Networks%20%28Andrew%20S.%20Tanenbaum.pdf#page=379)
> (why it takes three messages), then
> [§6.5.5 TCP Connection Establishment, pp. 412–413](../reference/Computer%20Networks%20%28Andrew%20S.%20Tanenbaum.pdf#page=412)
> (Fig. 6-31a: SYN, SYN+ACK, ACK), and
> [§6.5.7 TCP Connection Management Modeling, pp. 414–416](../reference/Computer%20Networks%20%28Andrew%20S.%20Tanenbaum.pdf#page=414)
> (Fig. 6-32 lists the states, and Fig. 6-33 is the state machine. Keep it
> open while you work through 5.10–5.18).

- [ ] **5.10 Connection table**: `stack/transport/tcp.py`
  A connection is keyed by (remote IP, remote port, local port). Each one
  stores `state` (one of the Fig. 6-32 states), `snd_nxt`, `rcv_nxt`, and
  `iss`. Port 7 is in LISTEN.
  *Observe:* a helper prints the table, which is empty for now.
- [ ] **5.11 Answer SYN**: `stack/transport/tcp.py`
  A SYN arriving at a LISTEN port: create a connection, pick an ISS, send
  SYN+ACK (seq = ISS, ack = their seq + 1), and move to SYN RCVD. That's the
  middle arrow of Fig. 6-31(a).
  *Observe:* `nc -v 10.0.0.2 7` prints `succeeded`. The kernel considers the
  handshake done.
- [ ] **5.12 Complete the handshake**: `stack/transport/tcp.py`
  The client's ACK in SYN RCVD → ESTABLISHED. Trace every state change.
  *Observe:* the trace shows `LISTEN → SYN RCVD → ESTABLISHED`, the same path
  as the heavy dashed line (the server's normal path) in Fig. 6-33 on p. 415.
- [ ] **5.13 Receive data**: `stack/transport/tcp.py`
  In ESTABLISHED, if `seq == rcv_nxt`: deliver the data, advance `rcv_nxt` by
  its length, and send an ACK. Otherwise just re-send an ACK for `rcv_nxt`.
  *Observe:* text typed into `nc` shows up in the trace, and `tcpdump` shows no
  client retransmissions.
- [ ] **5.14 Send data (echo)**: `stack/transport/tcp.py`
  Send the received data back with flags `PA` and `seq = snd_nxt`, then
  advance `snd_nxt`.
  *Observe:* `nc 10.0.0.2 7` echoes your lines through your TCP.

### TCP: closing and resets

> **Read first:** [§6.2.3 Connection Release, pp. 383–387](../reference/Computer%20Networks%20%28Andrew%20S.%20Tanenbaum.pdf#page=383)
> (the two-army problem: why closing is harder than opening), then
> [§6.5.6 TCP Connection Release, p. 413](../reference/Computer%20Networks%20%28Andrew%20S.%20Tanenbaum.pdf#page=413)
> and the closing half of Fig. 6-33, p. 414.

- [ ] **5.15 Passive close**: `stack/transport/tcp.py`
  A FIN arrives: advance `rcv_nxt` by 1, ACK it, and move to CLOSE WAIT. Send
  your own FIN and move to LAST ACK. On the final ACK → CLOSED, and remove the
  connection.
  *Observe:* pressing Ctrl-D in `nc` closes it cleanly, and the trace follows
  Fig. 6-33's passive-close path to `CLOSED`.
- [ ] **5.16 Reset closed ports**: `stack/transport/tcp.py`
  A segment for a port nobody listens on → reply with RST (p. 411 describes
  the RST flag).
  *Observe:* `nc 10.0.0.2 9999` fails immediately with `Connection refused`.
- [ ] **5.17 An application interface**: `stack/transport/tcp.py`
  `listen(port, on_data)`. The handler receives a connection object with
  `send(data)` and `close()`. These are the primitives from §6.1.2
  ([p. 368](../reference/Computer%20Networks%20%28Andrew%20S.%20Tanenbaum.pdf#page=368)).
  Re-register echo through it.
  *Observe:* echo works exactly as before. This is the hook level 6 uses.
- [ ] **5.18 Active close**: `stack/transport/tcp.py`
  `close()` sends FIN and moves to FIN WAIT 1. Their ACK → FIN WAIT 2. Their
  FIN → ACK it, then TIMED WAIT, then CLOSED. A short timer is fine.
  *Observe:* make echo close the connection after the first line. `nc` exits
  on its own, and the trace follows Fig. 6-33's active-close path.
- [ ] **5.19 Segment large sends**: `stack/transport/tcp.py`
  `send(data)` splits the data into chunks of at most the MSS. Use 1460, or
  read the MSS option from the client's SYN (Options field, p. 411).
  *Observe:* sending 10 000 bytes arrives complete, and `nc … | wc -c` prints
  10000.

### Stretch

- [ ] **5.20 Retransmission timer**: `stack/transport/tcp.py`
  [§6.5.10 TCP Timer Management, pp. 421–423](../reference/Computer%20Networks%20%28Andrew%20S.%20Tanenbaum.pdf#page=421):
  keep unacknowledged data and resend it if no ACK arrives in time. Start with
  a fixed timeout, then try the book's smoothed RTT estimate.
  *Observe:* drop every 5th outgoing segment on purpose, and the client still
  receives everything.
- [ ] **5.21 Respect the window**: `stack/transport/tcp.py`
  [§6.5.8 TCP Transmission Policy, pp. 416–418](../reference/Computer%20Networks%20%28Andrew%20S.%20Tanenbaum.pdf#page=416),
  Fig. 6-34: never have more unacknowledged bytes in flight than the client's
  advertised window.
  *Observe:* a client with a small window still receives everything, in
  several bursts that match the Fig. 6-34 pattern.

## Concepts covered

Ports and multiplexing, the UDP header and checksum, the pseudoheader, ICMP
errors, the TCP header and flags, sequence and ACK numbers (SYN and FIN each
count as one), the three-way handshake, the TCP state machine, passive
vs. active close, TIMED WAIT, RST, transport primitives, MSS and
segmentation, and (stretch) timer management and window-based flow control.
