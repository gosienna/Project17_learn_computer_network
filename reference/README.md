# Reference

`Computer Networks (Andrew S. Tanenbaum.pdf` is the **4th edition** (674 PDF
pages). The page numbers below are **PDF page numbers**, the ones a PDF viewer
shows, not the printed page numbers. Each link opens the PDF at that page
(`#page=N`). Some topics are in a different chapter than their layer:
Manchester encoding is in ch. 4 (Ethernet), and ARP and ICMP are in ch. 5.

## Section map by level

| Level | Topic | Section | PDF page |
|---|---|---|---|
| all | Layers and encapsulation | [§1.3.1 Protocol Hierarchies, Fig. 1-15](Computer%20Networks%20%28Andrew%20S.%20Tanenbaum.pdf#page=28) | 28–30 |
| all | The five-layer model this project follows | [§1.4, Fig. 1-24](Computer%20Networks%20%28Andrew%20S.%20Tanenbaum.pdf#page=45) | 45 |
| 1 | Signals, bandwidth, max data rate | [§2.1](Computer%20Networks%20%28Andrew%20S.%20Tanenbaum.pdf#page=69) | 69–72 |
| 1 | Twisted pair, coax, fiber | [§2.2](Computer%20Networks%20%28Andrew%20S.%20Tanenbaum.pdf#page=73) | 73–75 |
| 1 | Manchester encoding (**book: 1 = high→low, 0 = low→high**), Fig. 4-16 | [§4.3.2](Computer%20Networks%20%28Andrew%20S.%20Tanenbaum.pdf#page=206) | 206 |
| 2 | Services offered to the network layer | [§3.1.1](Computer%20Networks%20%28Andrew%20S.%20Tanenbaum.pdf#page=139) | 139–140 |
| 2 | Framing, byte stuffing (Fig. 3-5), bit stuffing (Fig. 3-6) | [§3.1.2](Computer%20Networks%20%28Andrew%20S.%20Tanenbaum.pdf#page=140) | 140–142 |
| 2 | Error control, flow control | [§3.1.3–3.1.4](Computer%20Networks%20%28Andrew%20S.%20Tanenbaum.pdf#page=143) | 143–144 |
| 2 | Parity, CRC (Fig. 3-8), IEEE 802 polynomial | [§3.2.2](Computer%20Networks%20%28Andrew%20S.%20Tanenbaum.pdf#page=146) | 146–148 |
| 2 | Frame definition `kind, seq, ack, info` (Fig. 3-9) | [§3.3](Computer%20Networks%20%28Andrew%20S.%20Tanenbaum.pdf#page=149) | 149–151 |
| 2 | Stop-and-wait (Fig. 3-11) | [§3.3.2](Computer%20Networks%20%28Andrew%20S.%20Tanenbaum.pdf#page=154) | 154 |
| 2 | Protocol 3: ACK + timeout + retransmit (Fig. 3-12) | [§3.3.3](Computer%20Networks%20%28Andrew%20S.%20Tanenbaum.pdf#page=155) | 155–157 |
| 2 | Protocol 4: one-bit sliding window (Fig. 3-14, 3-15) | [§3.4.1](Computer%20Networks%20%28Andrew%20S.%20Tanenbaum.pdf#page=160) | 160–161 |
| 2 | PPP (the real-world byte-stuffing variant) | [§3.6.2](Computer%20Networks%20%28Andrew%20S.%20Tanenbaum.pdf#page=177) | 177 |
| 3 | DIX Ethernet frame (Fig. 4-17), addresses, minimum size | [§4.3.3](Computer%20Networks%20%28Andrew%20S.%20Tanenbaum.pdf#page=206) | 206–207 |
| 3 | Switched Ethernet | [§4.3.6](Computer%20Networks%20%28Andrew%20S.%20Tanenbaum.pdf#page=211) | 211 |
| 3 | ARP (Fig. 5-62) | [§5.6.3, ARP](Computer%20Networks%20%28Andrew%20S.%20Tanenbaum.pdf#page=343) | 343–345 |
| 4 | IPv4 header (Fig. 5-53), fragmentation, TTL, header checksum | [§5.6.1](Computer%20Networks%20%28Andrew%20S.%20Tanenbaum.pdf#page=329) | 329–331 |
| 4 | IP addresses, subnets | [§5.6.2](Computer%20Networks%20%28Andrew%20S.%20Tanenbaum.pdf#page=332) | 332 |
| 4 | ICMP message types (Fig. 5-61) | [§5.6.3, ICMP](Computer%20Networks%20%28Andrew%20S.%20Tanenbaum.pdf#page=342) | 342 |
| 5 | Transport primitives, sockets | [§6.1.2–6.1.3](Computer%20Networks%20%28Andrew%20S.%20Tanenbaum.pdf#page=368) | 368–371 |
| 5 | Connection establishment and release (theory) | [§6.2.2–6.2.3](Computer%20Networks%20%28Andrew%20S.%20Tanenbaum.pdf#page=379) | 379–387 |
| 5 | UDP header (Fig. 6-23) | [§6.4.1](Computer%20Networks%20%28Andrew%20S.%20Tanenbaum.pdf#page=401) | 401–402 |
| 5 | TCP service model, byte stream | [§6.5.1–6.5.3](Computer%20Networks%20%28Andrew%20S.%20Tanenbaum.pdf#page=407) | 407–410 |
| 5 | TCP header (Fig. 6-29), pseudoheader (Fig. 6-30) | [§6.5.4](Computer%20Networks%20%28Andrew%20S.%20Tanenbaum.pdf#page=410) | 410–412 |
| 5 | Three-way handshake (Fig. 6-31) | [§6.5.5](Computer%20Networks%20%28Andrew%20S.%20Tanenbaum.pdf#page=412) | 412–413 |
| 5 | Closing a connection | [§6.5.6](Computer%20Networks%20%28Andrew%20S.%20Tanenbaum.pdf#page=413) | 413 |
| 5 | TCP states (Fig. 6-32) and state machine (Fig. 6-33) | [§6.5.7](Computer%20Networks%20%28Andrew%20S.%20Tanenbaum.pdf#page=414) | 414–416 |
| 5 | Window management (Fig. 6-34) | [§6.5.8](Computer%20Networks%20%28Andrew%20S.%20Tanenbaum.pdf#page=416) | 416–418 |
| 5 | Retransmission timer, RTT estimate | [§6.5.10](Computer%20Networks%20%28Andrew%20S.%20Tanenbaum.pdf#page=421) | 421–423 |
| 6 | NAT (Fig. 5-60) | [§5.6.2, NAT](Computer%20Networks%20%28Andrew%20S.%20Tanenbaum.pdf#page=339) | 339–341 |
| 6 | Web architecture | [§7.3.1](Computer%20Networks%20%28Andrew%20S.%20Tanenbaum.pdf#page=470) | 470–472 |
| 6 | HTTP: connections, methods, status codes, headers, example (Fig. 7-41–7-44) | [§7.3.4](Computer%20Networks%20%28Andrew%20S.%20Tanenbaum.pdf#page=502) | 502–506 |

## Where the framework differs from the book, on purpose

- **Byte stuffing:** the bites follow the book's Fig. 3-5 (ESC + the original
  byte). Real PPP also XORs the escaped byte with `0x20` (§3.6.2).
- **CRC-32:** the book teaches CRC as polynomial long division (Fig. 3-8), and
  bite 2.7 does exactly that. Bite 2.9's CRC-32 adds the real standard's
  conventions: reflected bit order, an all-ones starting value, and a final
  XOR.
- **Ethernet addresses:** the book's "high-order bit" means the first bit
  sent on the wire. In the bytes your code reads, that's the *lowest* bit of
  the first byte.

## Not in the 4th edition

Short notes in this folder will cover these as you reach them: TAP devices
(level 3), the ARP packet layout (RFC 826), the ICMP layout (RFC 792), the
Internet checksum worked example (RFC 1071), and modern HTTP/1.1 details.
