# Level 1: Physical layer (Tanenbaum ch. 2 + §4.3.2)

**Goal:** move text from one process to another as a stream of signal levels
over a simulated wire, see the waveform, and watch noise corrupt it.

**Runs on:** macOS, plain Python 3, two terminal windows.

**Current bite:** 1.1 (not started)

Book links open the PDF at the given page. Page numbers are PDF pages.

## Bites

### Bits

> **Read first:** [§2.1 The Theoretical Basis for Data Communication, pp. 69–72](../reference/Computer%20Networks%20%28Andrew%20S.%20Tanenbaum.pdf#page=69):
> a signal is a function of time, and a bit stream is the simplest signal.

- [ ] **1.1 Byte → bits**: `stack/phy/bits.py`
  Write `byte_to_bits(value)` → a list of 8 ints, most significant bit first.
  *Observe:* run the file directly; it prints the bits of `ord('h')` as
  `[0, 1, 1, 0, 1, 0, 0, 0]`.
- [ ] **1.2 Bits → byte**: `stack/phy/bits.py`
  Write `bits_to_byte(bits)`, the inverse.
  *Observe:* a self-check over all 256 values prints "round-trip OK".
- [ ] **1.3 Text ↔ bits**: `stack/phy/bits.py`
  Write `text_to_bits(text)` and `bits_to_text(bits)` using the two helpers
  (UTF-8 bytes, 8 bits each).
  *Observe:* `"hi"` → 16 bits → `"hi"`.

### Line coding: how bits become a signal

> **Read first:** [§4.3.2 Manchester Encoding, p. 206](../reference/Computer%20Networks%20%28Andrew%20S.%20Tanenbaum.pdf#page=206),
> Fig. 4-16 (binary, Manchester and differential Manchester waveforms for the
> same bit stream).

- [ ] **1.4 Binary (NRZ) encode/decode**: `stack/phy/linecode.py`
  Fig. 4-16(a): 1 → high, 0 → low. Use +1 and −1, not 1 and 0, so that "0"
  looks different from an idle line (the book's p. 206 argument).
  *Observe:* print the level list for `"hi"`.
- [ ] **1.5 Draw the waveform**: `stack/phy/linecode.py`
  Write `draw(levels)`, which turns levels into one line of ASCII (`‾` for
  high, `_` for low).
  *Observe:* draw the bit stream from Fig. 4-16 (`10000101111`). Your line
  matches the shape of row (a).
- [ ] **1.6 Manchester encode**: `stack/phy/linecode.py`
  Each bit becomes two half-bit levels with a transition in the middle. Use
  the book's convention: **1 = high→low, 0 = low→high**. IEEE 802.3 uses the
  reverse; both work as long as sender and receiver agree.
  *Observe:* draw `10000101111` and compare with Fig. 4-16(b). Then draw the
  byte `0x00` both ways: binary is one flat line, while Manchester keeps
  toggling. That's why the receiver can recover the clock.
- [ ] **1.7 Manchester decode**: `stack/phy/linecode.py`
  Read the level pairs back into bits. Flag a pair with no transition as a
  signal error.
  *Observe:* `"hi"` round-trips through Manchester.

### The wire

> **Read first:** [§2.2 Guided Transmission Media, pp. 73–75](../reference/Computer%20Networks%20%28Andrew%20S.%20Tanenbaum.pdf#page=73):
> a real cable just carries a signal from one end to the other. Your FIFO
> plays that role.

- [ ] **1.8 A one-way wire**: `stack/phy/wire.py`
  Create a named pipe (FIFO) at a fixed path if it doesn't exist. Write
  `send(levels)`, which writes each level as one character, and
  `receive()`, which reads until the sender closes.
  *Observe:* running `cat` on the FIFO in terminal B shows the raw stream when
  you call `send` from terminal A.
- [ ] **1.9 Sender tool**: `tools/phy_send.py`
  Take a message from the command line, then text → bits → Manchester →
  draw the waveform → `send`.
  *Observe:* terminal A shows the waveform it is sending.
- [ ] **1.10 Receiver tool**: `tools/phy_recv.py`
  `receive` → draw the waveform → Manchester decode → bits → text → print.
  *Observe:* start the receiver in B, send `"hello"` from A, and `hello`
  appears in B.

### Noise

> **Read first:** [§3.1.3 Error Control, p. 143](../reference/Computer%20Networks%20%28Andrew%20S.%20Tanenbaum.pdf#page=143):
> why the layer above has to deal with damaged bits.

- [ ] **1.11 A noisy wire**: `stack/phy/wire.py`
  Give `send` a `flip_probability` argument, and flip each level at random
  with that probability.
  *Observe:* at `0` the text arrives clean; at `0.02` you get garbage and
  signal errors. Nothing in level 1 can tell a good message from a bad one,
  and that's what level 2 fixes.
- [ ] **1.12 Count the damage** (stretch): `tools/phy_recv.py`
  Given the message you expected, report how many bits were wrong (the bit
  error rate).
  *Observe:* the error rate roughly follows `flip_probability` as you change it.
- [ ] **1.13 Differential Manchester** (stretch): `stack/phy/linecode.py`
  Fig. 4-16(c): every bit has a transition in the middle. A 0 also has a
  transition at the start; a 1 doesn't.
  *Observe:* draw `10000101111` and compare with Fig. 4-16(c).

## Concepts covered

Bits vs. signals, binary encoding vs. Manchester vs. differential Manchester,
clock recovery, a medium as a dumb byte stream, simplex links, and noise and
bit errors.
