# Level 2: Data link layer (Tanenbaum ch. 3)

**Goal:** turn the raw wire from level 1 into a link that carries whole
messages: find where each frame starts and ends, detect corrupted frames, and
recover from them with acknowledgements and retransmission.

**Runs on:** macOS, plain Python 3, two terminal windows.

**Builds on:** `stack/phy/` from level 1. Nothing in `stack/link/` reaches
below `stack/phy/port.py`.

**Current bite:** not started (finish level 1 first)

## Bites

### Framing: where does a message start and end?

- [ ] **2.1 Byte stuffing**: `stack/link/framing.py`
  Define `FLAG = 0x7E` and `ESC = 0x7D`. Write `stuff(payload)`: every FLAG or
  ESC byte in the payload becomes ESC followed by (byte XOR 0x20), the PPP
  convention.
  *Observe:* stuffing bytes `41 7E 42 7D` prints `41 7D 5E 42 7D 5D` in hex.
- [ ] **2.2 Unstuffing**: `stack/link/framing.py`
  Write `unstuff(data)`, the inverse.
  *Observe:* a round-trip check on a payload made only of FLAG and ESC bytes
  prints OK.
- [ ] **2.3 Build a frame**: `stack/link/framing.py`
  `make_frame(payload)` = FLAG + stuffed payload + FLAG.
  *Observe:* the hex dump starts and ends with `7E`, and `7E` never appears in
  between.
- [ ] **2.4 Find frames in a stream**: `stack/link/framing.py`
  A small `FrameParser` with `feed(byte)`. It ignores bytes until a FLAG,
  collects bytes until the next FLAG, then returns the collected frame.
  Ignore escapes for now.
  *Observe:* feeding `junk + frame1 + junk + frame2` byte by byte prints exactly
  two payloads.
- [ ] **2.5 Handle escapes while parsing**: `stack/link/framing.py`
  Inside a frame, an ESC means "XOR the next byte with 0x20".
  *Observe:* a payload containing `7E` comes back intact through the parser.

### Error detection: is this frame damaged?

- [ ] **2.6 Parity bit**: `stack/link/checksum.py`
  `parity(data)` returns 1 if the total number of 1 bits is odd.
  *Observe:* flipping one bit changes the parity; flipping two bits doesn't.
  That's parity's blind spot.
- [ ] **2.7 Internet checksum**: `stack/link/checksum.py`
  `internet_checksum(data)`: sum 16-bit words, fold the carries back in, take
  the one's complement. Pad an odd-length input with a zero byte. You'll
  reuse this in levels 4 and 5.
  *Observe:* the RFC 1071 example bytes `00 01 F2 03 F4 F5 F6 F7` give `0x220D`.
- [ ] **2.8 CRC-32**: `stack/link/checksum.py`
  `crc32(data)`, computed bit by bit with the reflected polynomial
  `0xEDB88320`, start value all ones, and a final XOR with all ones.
  *Observe:* `"123456789"` gives `0xCBF43926`, and so does Python's
  `zlib.crc32`.
- [ ] **2.9 Frames carry a CRC**: `stack/link/framing.py`
  `make_frame` appends the 4-byte CRC of the payload before stuffing.
  *Observe:* the frame is 4 bytes longer, and the last 4 bytes before the
  closing flag match `crc32(payload)`.
- [ ] **2.10 Parser checks the CRC**: `stack/link/framing.py`
  When a frame completes, split off the last 4 bytes, recompute the CRC, and
  return `(payload, ok)`.
  *Observe:* flip one byte inside a built frame and the parser reports
  `ok = False`.

### Carrying frames over the wire

- [ ] **2.11 A wire that stays open**: `stack/phy/wire.py`
  Add `open_tx()`, which returns a writer you can call `send` on many times,
  and `receive_stream()`, which yields each level as it arrives instead of
  waiting for the sender to close.
  *Observe:* the sender sends three messages one second apart, and the
  receiver prints levels in three bursts.
- [ ] **2.12 PHY port, transmit side**: `stack/phy/port.py`
  `transmit(data)`: bytes → bits → Manchester → wire. This is the only
  function the link layer calls to send.
  *Observe:* `cat` on the FIFO shows the stream when you transmit a frame.
- [ ] **2.13 PHY port, receive side**: `stack/phy/port.py`
  `receive_bytes()`: take 16 levels at a time from `receive_stream`, decode
  them into one byte, and yield it.
  *Observe:* a quick test prints the transmitted bytes in hex on the receive
  side.
- [ ] **2.14 Link sender tool**: `tools/link_send.py`
  For each line typed, `make_frame` → `transmit`.
  *Observe:* typed lines go out; the receiver doesn't exist yet, so check with
  `cat`.
- [ ] **2.15 Link receiver tool**: `tools/link_recv.py`
  Feed `receive_bytes` into `FrameParser`, and print each payload with `OK` or
  `CORRUPT`.
  *Observe:* at noise `0` every line is OK. At noise `0.002` some lines are
  marked CORRUPT, and you never see silently wrong text.

### Recovering: stop-and-wait ARQ

- [ ] **2.16 A return path**: `stack/phy/wire.py`
  Give the wire a name, so two FIFOs exist: `a_to_b` and `b_to_a`. A
  full-duplex cable has one pair for each direction.
  *Observe:* `ls` shows both FIFOs, and each direction works independently.
- [ ] **2.17 Frame header**: `stack/link/arq.py`
  The first payload byte holds the frame type (DATA or ACK) and a 1-bit
  sequence number. Write `pack(kind, seq, data)` and `unpack(payload)`.
  *Observe:* pack and then unpack a DATA frame with seq 1 and get the same
  values back.
- [ ] **2.18 Receiver sends ACKs**: `tools/link_recv.py`
  For every good DATA frame, transmit `ACK(seq)` on the return wire. Drop
  corrupted frames without replying.
  *Observe:* the trace shows `→ ACK 0`, `→ ACK 1`… for each good frame.
- [ ] **2.19 Receiver drops duplicates**: `tools/link_recv.py`
  Keep `expected_seq`. Deliver only when `seq == expected_seq` and then
  toggle it. Still ACK duplicates, because the sender missed the earlier ACK.
  *Observe:* if you resend the same frame by hand, it's ACKed but printed once.
- [ ] **2.20 Sender waits for the ACK**: `stack/link/arq.py`
  `send_reliable(data)`: transmit `DATA(seq)`, block until a good ACK with that
  seq arrives, then toggle seq.
  *Observe:* at noise `0`, lines are delivered one at a time, each after its
  ACK.
- [ ] **2.21 Timeout and retransmit**: `stack/link/arq.py`
  Wait for the ACK with a timeout (use `select` on the FIFO). On timeout,
  retransmit and print `RETRANSMIT seq=…`.
  *Observe:* at noise `0.002` every line eventually arrives intact, and the
  sender's trace shows retransmissions.
- [ ] **2.22 Use it**: `tools/link_send.py`
  Switch from a plain `transmit` to `send_reliable`.
  *Observe (finish line):* type a paragraph at noise `0.002`. It arrives
  complete and in order, even though frames and ACKs are being damaged along
  the way.

## Concepts covered

Framing, flag bytes and byte stuffing, parity, the Internet checksum, CRC,
error detection vs. correction, ACKs, sequence numbers, duplicate detection,
timeouts, stop-and-wait ARQ, and simplex vs. full duplex.
