# View track (optional): a web page that shows your stack at work

**Goal:** a browser page that shows every frame, packet and segment your stack
handles, unwrapped layer by layer, with the raw bytes colored by layer.

**When:** any time after level 3, bite 3.11. Do these bites between levels, or
whenever a level feels too abstract. The CLI trace stays the main check; this
page is extra.

**How it works:** the stack appends one JSON object per event to
`trace.jsonl`. A static page in `web/` reads that file every second. Serve the
project folder with Python's built-in HTTP server and open `/web/`.

**Current bite:** not started

Book links open the PDF at the given page. Page numbers are PDF pages.

## Bites

> **Read first:** [§1.3.1 Protocol Hierarchies, pp. 28–30](../reference/Computer%20Networks%20%28Andrew%20S.%20Tanenbaum.pdf#page=28).
> Fig. 1-15 shows a message picking up a header at each layer on the way
> down and losing it on the way up. This page draws that figure for real
> packets.

### Events

- [ ] **V.1 Trace module**: `stack/trace.py`
  `emit(layer, direction, summary, fields)` prints the usual CLI line and also
  appends one JSON line to `trace.jsonl`, with a timestamp.
  *Observe:* `tail -f trace.jsonl` shows events while you ping.
- [ ] **V.2 Packet IDs**: `stack/netif.py`
  Give each frame read or written a packet ID, and pass it down so every
  layer's event carries it.
  *Observe:* the Ethernet, IP and ICMP events for one ping share an ID.
- [ ] **V.3 Emit from Ethernet and ARP**: `stack/netif.py`
  Switch the existing prints to `emit`, including the raw frame bytes as hex.
  *Observe:* `trace.jsonl` lines contain a `hex` field.
- [ ] **V.4 Emit from IP and ICMP**: `stack/ip/ipv4.py`
  Include the header field values and the byte offset where this layer's
  header starts.
  *Observe:* each IP event has `offset: 14`.
- [ ] **V.5 Emit from TCP and UDP**: `stack/transport/tcp.py`
  Include flags, seq/ack, and state changes.
  *Observe:* a handshake produces three events with matching seq/ack values.

### The page

- [ ] **V.6 Skeleton**: `web/index.html`
  A title, an empty packet table (time, direction, summary), and a script tag
  for `app.js`.
  *Observe:* the page loads and shows the empty table.
- [ ] **V.7 Show events**: `web/app.js`
  Fetch `/trace.jsonl`, split it into lines, parse each one, and add a row
  per packet ID.
  *Observe:* reloading shows one row per ping packet.
- [ ] **V.8 Live updates**: `web/app.js`
  Re-fetch every second and add only the new rows.
  *Observe:* rows appear by themselves while `ping` runs.
- [ ] **V.9 Layer breakdown**: `web/app.js`
  Clicking a row expands it into nested sections, Ethernet → IP → ICMP/TCP →
  data, listing each layer's fields.
  *Observe:* clicking a ping row shows the MACs, IPs, TTL and ICMP seq.
- [ ] **V.10 Colored hex**: `web/app.js`
  Show the raw bytes, coloring each layer's header range differently (using
  the offsets from V.4).
  *Observe:* the first 14 bytes are one color, the next 20 another, and so on.
- [ ] **V.11 Styles**: `web/style.css`
  Use a monospace font for hex, layer colors that work in both light and dark
  mode, and readable spacing.
  *Observe:* the page looks good in both color schemes.
- [ ] **V.12 TCP sequence diagram** (stretch): `web/app.js`
  For a chosen connection, draw two vertical lines (client and your stack)
  with a slanted arrow for each segment, labeled with its flags and seq/ack.
  *Observe:* a `curl` request shows the handshake, data, and FIN exchange as
  a diagram.

## Concepts covered

Encapsulation made visible, the byte offsets of each header, and timing and
ordering of events.
