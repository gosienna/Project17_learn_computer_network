# Level 1: Physical layer (Tanenbaum ch. 2)

**Goal:** move text from one process to another as a stream of signal levels
over a simulated wire, see the waveform, and watch noise corrupt it.

**Runs on:** macOS, plain Python 3, two terminal windows.

**Current bite:** 1.1 (not started)

## Bites

### Bits

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

- [ ] **1.4 NRZ encode/decode**: `stack/phy/linecode.py`
  1 → high level, 0 → low level, and back.
  *Observe:* print the level list for `"hi"`.
- [ ] **1.5 Draw the waveform**: `stack/phy/linecode.py`
  Write `draw(levels)`, which turns levels into one line of ASCII (`‾` for
  high, `_` for low).
  *Observe:* the waveform for `"hi"` appears in the terminal.
- [ ] **1.6 Manchester encode**: `stack/phy/linecode.py`
  Each bit becomes two half-bit levels with a transition in the middle (IEEE
  802.3: 0 = high→low, 1 = low→high).
  *Observe:* draw NRZ and Manchester for the byte `0x00`. NRZ is one flat
  line; Manchester keeps toggling. That's why the receiver can recover the
  clock.
- [ ] **1.7 Manchester decode**: `stack/phy/linecode.py`
  Read the level pairs back into bits. Flag a pair with no transition as a
  signal error.
  *Observe:* `"hi"` round-trips through Manchester.

### The wire

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

## Concepts covered

Bits vs. signals, line coding (NRZ, Manchester), clock recovery, a medium as a
dumb byte stream, simplex links, and noise and bit errors.
