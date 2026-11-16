# Framework: build a server from scratch, bottom-up

The goal is a tiny network stack written by you, layer by layer, that ends with
an HTTP server. When a browser loads a page from it, every byte on the wire was
produced by your code.

The ladder follows the textbook,
`reference/Computer Networks (Andrew S. Tanenbaum.pdf` (4th edition). Its
five-layer "hybrid model"
([§1.4, Fig. 1-24, p. 45](../reference/Computer%20Networks%20%28Andrew%20S.%20Tanenbaum.pdf#page=45))
is exactly the ladder below: physical, data link, network, transport,
application. Level 3 adds the MAC sublayer, which the book covers in ch. 4.

Each group of bites starts with a **Read first** line that links to the book
section, by PDF page, that the bites put into practice. Where a bite's
*Observe* check reproduces a figure (Fig. 3-5, 3-8, 4-16…), your output should
match the book's. [reference/README.md](../reference/README.md) has the full
map of sections to pages.

## Rules of the road

- You write all of the code. Each step (a "bite") is one file and fewer than
  10 lines, and has a visible result.
- Progress is tracked in the `level-N-bites.md` checklists. `[x]` means done
  and observed.
- Work is committed on your `learn/<name>` branch after each bite.
- Language: **Python 3** (standard library only until level 3).

## Project layout (grows as you go)

```
stack/
  phy/         level 1  bits, line coding, the simulated wire
  link/        level 2  framing, error detection, stop-and-wait ARQ
  eth/         level 3  TAP device, Ethernet, ARP
  ip/          level 4  IPv4, ICMP
  transport/   level 5  UDP, minimal TCP
  app/         level 6  HTTP server
tools/         small CLI scripts you run to observe each level (sender, receiver, sniff…)
web/           (optional view track) a page that visualizes trace events
```

Each layer only talks to the layer directly below it. Upper layers hand bytes
down; lower layers hand bytes up. That rule is the whole point of layering —
keep it even when a shortcut is tempting.

## Environments

| Levels | Where it runs | Why |
|---|---|---|
| 1–2 | macOS directly, two terminals | The "wire" is simulated with named pipes (FIFOs) — no real network needed |
| 3–6 | Linux VM (UTM) or Docker container with `/dev/net/tun` and `NET_ADMIN` | Needs a TAP device to receive raw Ethernet frames |

From level 3 on, the kernel owns `tap0` at `10.0.0.1` and **your stack** claims
`10.0.0.2`. Commands like `ping 10.0.0.2`, `nc 10.0.0.2 …`, and `curl
http://10.0.0.2/` run on the kernel side and talk to your code.

## The ladder

| Level | Layer | Book sections | You build | Finish line you can observe |
|---|---|---|---|---|
| 1 | Physical | §2.1–2.2, §4.3.2 | bits ↔ bytes, binary & Manchester encoding (Fig. 4-16), a simulated wire, noise | text sent as a waveform in terminal A arrives in terminal B, and garbles when you add noise |
| 2 | Data link | §3.1–3.4 | flag bytes + byte stuffing (Fig. 3-5), CRC (Fig. 3-8), protocol 3 (stop-and-wait with retransmission) over a full-duplex wire | a paragraph arrives intact over a noisy wire (0.2% of levels flipped); you see retransmissions in the trace |
| 3 | MAC / Ethernet | §4.3.3, §5.6.3 (ARP) | read raw frames from TAP, parse the DIX frame (Fig. 4-17), answer ARP | `ip neigh` on the kernel side shows the MAC address *your* code invented |
| 4 | Network | §5.6.1, §5.6.3 (ICMP) | parse/verify IPv4 headers (Fig. 5-53), build ICMP echo replies (Fig. 5-61) | `ping 10.0.0.2` gets replies from your stack |
| 5 | Transport | §6.2, §6.4.1, §6.5 | UDP (Fig. 6-23); minimal TCP (Fig. 6-29) following the state machine (Fig. 6-33) | `nc 10.0.0.2 7` completes a handshake and echoes text through your TCP |
| 6 | Application | §7.3.1, §7.3.4 | parse an HTTP request, send a response (Fig. 7-42–7-44) | `curl`, then a browser, loads a page served entirely by your stack |

### Bite checklists

| File | Bites | Note |
|---|---|---|
| [level-1-bites.md](level-1-bites.md) | 13 | macOS, simulated wire; includes 2 stretch bites |
| [level-2-bites.md](level-2-bites.md) | 26 | macOS, two FIFOs for full duplex; includes 2 stretch bites |
| [level-3-bites.md](level-3-bites.md) | 11 | switch to the Linux box |
| [level-4-bites.md](level-4-bites.md) | 12 | |
| [level-5-bites.md](level-5-bites.md) | 21 | includes 2 stretch bites |
| [level-6-bites.md](level-6-bites.md) | 10 | includes 2 stretch bites |
| [view-track-bites.md](view-track-bites.md) | 12 | optional, any time after bite 3.11 |

The checklists are a plan, not a contract. A bite that turns out too big gets
split, and later levels can change based on how earlier ones went.

## Observing: CLI first, web later

Every level is observed from the terminal first: printed bits, ASCII waveforms,
one trace line per frame or packet, such as `[IP] ← 10.0.0.1 ICMP len=84`.

**Optional view track (from level 3):** each layer also emits its trace event as
one JSON line, and a small page in `web/` shows each packet unwrapping layer by
layer (Ethernet → IP → TCP → HTTP). It's added as separate bites and never
replaces the CLI check.

## Tools that help you check your work

- `xxd` / `hexdump`: look at raw bytes.
- `tcpdump` / Wireshark on `tap0` (level 3+): compare your parser with a real one.
- `ping`, `nc`, `curl`: real clients talking to your stack.
