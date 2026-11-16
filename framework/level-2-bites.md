# Level 2: Data link layer (Tanenbaum ch. 3)

**Goal:** turn the raw wire from level 1 into a link that carries whole
messages: find where each frame starts and ends, detect corrupted frames, and
recover from them with acknowledgements and retransmission.

**Runs on:** macOS, plain Python 3, two terminal windows.

**Builds on:** `stack/phy/` from level 1. Nothing in `stack/link/` reaches
below `stack/phy/port.py`.

**Current bite:** not started (finish level 1 first)

Book links open the PDF at the given page. Page numbers are PDF pages.

## Bites

### Framing: where does a message start and end?

> **Read first:** [§3.1.2 Framing, pp. 140–142](../reference/Computer%20Networks%20%28Andrew%20S.%20Tanenbaum.pdf#page=140):
> Fig. 3-4 shows why a character count breaks after one error. Fig. 3-5 shows
> the frame layout `FLAG | Header | Payload | Trailer | FLAG` and four
> byte-stuffing examples.

- [ ] **2.1 Byte stuffing**: `stack/link/framing.py`
  Define `FLAG = 0x7E` and `ESC = 0x7D`. Write `stuff(data)` the way the book
  does it: put an ESC in front of every FLAG or ESC byte in the data, and
  leave the byte itself unchanged.
  *Observe:* reproduce the four rows of Fig. 3-5(b) with `A = 0x41` and
  `B = 0x42`. For example, `A FLAG B` → `A ESC FLAG B`, and `A ESC ESC B` →
  `A ESC ESC ESC ESC B`.
- [ ] **2.2 Unstuffing**: `stack/link/framing.py`
  Write `unstuff(data)`: an ESC means "the next byte is ordinary data, keep it
  and drop the ESC".
  *Observe:* all four Fig. 3-5(b) rows round-trip back to the original.
- [ ] **2.3 Build a frame**: `stack/link/framing.py`
  `make_frame(payload)` = FLAG + stuffed payload + FLAG.
  *Observe:* the hex dump starts and ends with `7E`, and every `7E` in between
  has a `7D` in front of it.
- [ ] **2.4 Find frames in a stream**: `stack/link/framing.py`
  A small `FrameParser` with `feed(byte)`. It ignores bytes until a FLAG,
  collects bytes until the next FLAG, then returns the collected frame.
  Ignore escapes for now.
  *Observe:* feeding `junk + frame1 + junk + frame2` byte by byte prints exactly
  two payloads. That's the resynchronization the book says a character count
  can't do.
- [ ] **2.5 Handle escapes while parsing**: `stack/link/framing.py`
  Inside a frame, the byte after an ESC is data, even if it's a FLAG.
  *Observe:* a payload containing `7E` comes back intact through the parser.

### Error detection: is this frame damaged?

> **Read first:** [§3.2.2 Error-Detecting Codes, pp. 146–148](../reference/Computer%20Networks%20%28Andrew%20S.%20Tanenbaum.pdf#page=146):
> parity, modulo-2 arithmetic, the CRC algorithm, the Fig. 3-8 worked
> example, and the IEEE 802 generator polynomial.
> Internet checksum: [§5.6.1, p. 331](../reference/Computer%20Networks%20%28Andrew%20S.%20Tanenbaum.pdf#page=331)
> (the IP header checksum paragraph).

- [ ] **2.6 Parity bit**: `stack/link/checksum.py`
  `parity(data)` returns 1 if the total number of 1 bits is odd.
  *Observe:* flipping one bit changes the parity; flipping two bits doesn't.
  That's parity's blind spot.
- [ ] **2.7 CRC by long division**: `stack/link/checksum.py`
  `crc_remainder(frame_bits, generator_bits)`, following the book's three
  steps: append r zero bits, divide using XOR (modulo-2 subtraction), and
  return the r-bit remainder. Work on strings of `'0'`/`'1'` so you can see
  each step.
  *Observe:* frame `1101011011` with generator `10011` (x⁴ + x + 1) gives
  remainder `1110`, so the transmitted frame is `11010110111110`, just like
  Fig. 3-8.
- [ ] **2.8 CRC check**: `stack/link/checksum.py`
  The receiver divides the whole received frame by the generator. A zero
  remainder means "accept".
  *Observe:* `11010110111110` gives remainder `0000`. Flip any single bit and
  the remainder isn't zero.
- [ ] **2.9 CRC-32**: `stack/link/checksum.py`
  `crc32(data)` over bytes, using the IEEE 802 generator from p. 148. Real
  CRC-32 adds three conventions the book leaves out: bits are processed
  least significant first (so the generator is used in reflected form,
  `0xEDB88320`), the register starts at all ones, and the result is XORed with
  all ones.
  *Observe:* `"123456789"` gives `0xCBF43926`, and so does Python's
  `zlib.crc32`.
- [ ] **2.10 Internet checksum**: `stack/link/checksum.py`
  `internet_checksum(data)`: add the data as 16-bit words in one's complement
  arithmetic (fold the carries back in), then take the one's complement.
  Pad an odd-length input with a zero byte. You'll reuse this in levels 4
  and 5.
  *Observe:* the RFC 1071 example bytes `00 01 F2 03 F4 F5 F6 F7` give `0x220D`.
- [ ] **2.11 Frames carry a CRC trailer**: `stack/link/framing.py`
  `make_frame` appends the 4-byte CRC-32 of the payload (the *Trailer* in
  Fig. 3-5a) before stuffing.
  *Observe:* the frame is 4 bytes longer, and the last 4 bytes before the
  closing flag match `crc32(payload)`.
- [ ] **2.12 Parser checks the CRC**: `stack/link/framing.py`
  When a frame completes, split off the last 4 bytes, recompute the CRC, and
  return `(payload, ok)`.
  *Observe:* flip one byte inside a built frame and the parser reports
  `ok = False`.

### Carrying frames over the wire

> **Read first:** [§3.1.1 Services Provided to the Network Layer, pp. 139–140](../reference/Computer%20Networks%20%28Andrew%20S.%20Tanenbaum.pdf#page=139):
> the data link layer uses the physical layer's bit pipe and offers frames to
> the layer above.

- [ ] **2.13 A wire that stays open**: `stack/phy/wire.py`
  Add `open_tx()`, which returns a writer you can call `send` on many times,
  and `receive_stream()`, which yields each level as it arrives instead of
  waiting for the sender to close.
  *Observe:* the sender sends three messages one second apart, and the
  receiver prints levels in three bursts.
- [ ] **2.14 PHY port, transmit side**: `stack/phy/port.py`
  `transmit(data)`: bytes → bits → Manchester → wire. This is the only
  function the link layer calls to send.
  *Observe:* `cat` on the FIFO shows the stream when you transmit a frame.
- [ ] **2.15 PHY port, receive side**: `stack/phy/port.py`
  `receive_bytes()`: take 16 levels at a time from `receive_stream`, decode
  them into one byte, and yield it.
  *Observe:* a quick test prints the transmitted bytes in hex on the receive
  side.
- [ ] **2.16 Link sender tool**: `tools/link_send.py`
  For each line typed, `make_frame` → `transmit`.
  *Observe:* typed lines go out; the receiver doesn't exist yet, so check with
  `cat`.
- [ ] **2.17 Link receiver tool**: `tools/link_recv.py`
  Feed `receive_bytes` into `FrameParser`, and print each payload with `OK` or
  `CORRUPT`.
  *Observe:* at noise `0` every line is OK. At noise `0.002` some lines are
  marked CORRUPT, and you never see silently wrong text.

### Recovering: stop-and-wait ARQ

> **Read first:** [§3.3 Elementary Data Link Protocols, p. 149](../reference/Computer%20Networks%20%28Andrew%20S.%20Tanenbaum.pdf#page=149)
> (Fig. 3-9 on p. 150 defines a frame as `kind, seq, ack, info`),
> [§3.3.2 Stop-and-Wait, p. 154](../reference/Computer%20Networks%20%28Andrew%20S.%20Tanenbaum.pdf#page=154)
> (Fig. 3-11), and
> [§3.3.3 A Simplex Protocol for a Noisy Channel, pp. 155–157](../reference/Computer%20Networks%20%28Andrew%20S.%20Tanenbaum.pdf#page=155)
> (Fig. 3-12, "protocol 3": positive acknowledgement with retransmission).
> Steps 2.18–2.24 build protocol 3.

- [ ] **2.18 A return path**: `stack/phy/wire.py`
  Give the wire a name, so two FIFOs exist: `a_to_b` and `b_to_a`. A
  full-duplex cable has one pair for each direction.
  *Observe:* `ls` shows both FIFOs, and each direction works independently.
- [ ] **2.19 Frame header**: `stack/link/arq.py`
  This is the *Header* of Fig. 3-5a. One byte holds `kind` (DATA or ACK) and a
  1-bit `seq`, following the book's frame definition in Fig. 3-9. Write
  `pack(kind, seq, info)` and `unpack(payload)`.
  *Observe:* pack and then unpack a DATA frame with seq 1 and get the same
  values back.
- [ ] **2.20 Receiver sends ACKs**: `tools/link_recv.py`
  For every good DATA frame, transmit an ACK on the return wire. Drop
  corrupted frames without replying, just as protocol 3 does on `cksum_err`.
  *Observe:* the trace shows `→ ACK 0`, `→ ACK 1`… for each good frame.
- [ ] **2.21 Receiver drops duplicates**: `tools/link_recv.py`
  Keep the book's `frame_expected`. Deliver only when `seq == frame_expected`,
  then toggle it. Still ACK duplicates, because the sender missed the earlier
  ACK (p. 156 explains why).
  *Observe:* if you resend the same frame by hand, it's ACKed but printed once.
- [ ] **2.22 Sender waits for the ACK**: `stack/link/arq.py`
  `send_reliable(data)`: transmit `DATA(seq)`, block until a good ACK arrives,
  then toggle seq (the book's `next_frame_to_send`).
  *Observe:* at noise `0`, lines are delivered one at a time, each after its
  ACK.
- [ ] **2.23 Timeout and retransmit**: `stack/link/arq.py`
  Wait for the ACK with a timeout (use `select` on the FIFO). On timeout,
  retransmit and print `RETRANSMIT seq=…`. This is protocol 3's `timeout`
  event.
  *Observe:* at noise `0.002` every line eventually arrives intact, and the
  sender's trace shows retransmissions.
- [ ] **2.24 Use it**: `tools/link_send.py`
  Switch from a plain `transmit` to `send_reliable`.
  *Observe (finish line):* type a paragraph at noise `0.002`. It arrives
  complete and in order, even though frames and ACKs are being damaged along
  the way.

### Stretch

- [ ] **2.25 Bit stuffing**: `stack/link/bitstuff.py`
  [§3.1.2, p. 142](../reference/Computer%20Networks%20%28Andrew%20S.%20Tanenbaum.pdf#page=142),
  Fig. 3-6: after five 1 bits in a row, insert a 0. The receiver removes it.
  *Observe:* the data `01111110` goes on the line as `011111010` and comes
  back as `01111110`.
- [ ] **2.26 One-bit sliding window**: `stack/link/arq.py`
  [§3.4.1, pp. 160–161](../reference/Computer%20Networks%20%28Andrew%20S.%20Tanenbaum.pdf#page=160),
  protocol 4 (Fig. 3-14): both sides send data, and each frame's `ack` field
  carries the acknowledgement (piggybacking).
  *Observe:* a two-way chat works over the noisy wire, and the trace matches
  the scenarios in Fig. 3-15.

## Concepts covered

Framing (character count vs. flag bytes), byte stuffing and bit stuffing,
parity, CRC via polynomial division, CRC-32, the Internet checksum, error
detection vs. correction, ACKs, sequence numbers, duplicate detection,
timeouts, protocol 3 (stop-and-wait with retransmission), piggybacking, and
simplex vs. full duplex.
