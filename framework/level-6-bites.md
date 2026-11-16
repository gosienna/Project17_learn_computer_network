# Level 6: Application layer, an HTTP server (Tanenbaum §7.3)

**Goal:** an HTTP/1.1 server running on your own TCP, IP, ARP and Ethernet
code, reachable from `curl` and then from a browser on your Mac.

**Runs on:** the Linux box from level 3. In bite 6.7 the browser reaches it
through a port forward.

**Builds on:** `tcp.listen(port, on_data)`, `send`, and `close` from level 5.

**Current bite:** not started

Book links open the PDF at the given page. Page numbers are PDF pages.

## Bites

### Reading requests

> **Read first:** [§7.3.1 Architectural Overview, pp. 470–472](../reference/Computer%20Networks%20%28Andrew%20S.%20Tanenbaum.pdf#page=470)
> (client, server, URL, and what happens when you click a link), then
> [§7.3.4 HTTP, pp. 502–504](../reference/Computer%20Networks%20%28Andrew%20S.%20Tanenbaum.pdf#page=502):
> connections, persistent connections, and Fig. 7-41's methods.

- [ ] **6.1 Collect the request**: `stack/app/http.py`
  Buffer each connection's bytes until the blank line (`\r\n\r\n`) that ends
  the headers. P. 506 notes that the blank line is required, because it tells
  the server there are no more headers. Register the handler on port 80.
  *Observe:* `curl http://10.0.0.2/` hangs (there's no reply yet), but the
  stack prints the full request text curl sent.
- [ ] **6.2 Parse the request**: `stack/app/http.py`
  Split the request line into method, path and version, and the headers into
  a dict (header names are case-insensitive). The Host header is mandatory in
  HTTP/1.1 (p. 505).
  *Observe:* the trace shows `[HTTP] GET / HTTP/1.1  Host=10.0.0.2`.

### Sending responses

> **Read first:** [§7.3.4, pp. 504–506](../reference/Computer%20Networks%20%28Andrew%20S.%20Tanenbaum.pdf#page=504):
> Fig. 7-42 (status code groups), Fig. 7-43 (common headers), and Fig. 7-44 (a
> real response, as seen through `telnet`).

- [ ] **6.3 Build a response**: `stack/app/http.py`
  `response(status, content_type, body)`: a status line, `Content-Type`,
  `Content-Length`, `Connection: close`, a blank line, then the body.
  *Observe:* a self-test prints the exact bytes, with every `\r\n` in place.
  The shape matches Fig. 7-44.
- [ ] **6.4 First page**: `stack/app/http.py`
  Answer every request with `200` and a short HTML body, then `close()`.
  *Observe:* `curl -v http://10.0.0.2/` shows your status line, headers and
  body. Also try the book's approach from p. 506: `nc 10.0.0.2 80`, then type
  the request by hand.
- [ ] **6.5 Routes and 404**: `stack/app/http.py`
  Map paths to handlers. Unknown paths get `404 Not Found`, from the 4xx group
  in Fig. 7-42.
  *Observe:* `curl -i http://10.0.0.2/nope` shows `404`.
- [ ] **6.6 Serve files**: `stack/app/http.py`
  Map `/x` to `www/x` (and `/` to `www/index.html`), with the content type
  chosen by file extension. Reject any path containing `..`.
  *Observe:* edit `www/index.html`, run `curl` again, and see the change.

### A real browser

> **Read first:** [§5.6.2, NAT, pp. 339–341](../reference/Computer%20Networks%20%28Andrew%20S.%20Tanenbaum.pdf#page=339)
> (Fig. 5-60). Your port forward is a tiny relative of a NAT box.

- [ ] **6.7 Reach it from the Mac**: `tools/forward.sh`
  Inside the Linux box, use `socat` to listen on port 8080 and forward each
  connection to `10.0.0.2:80`. Publish port 8080 to the Mac (Docker `-p` or a
  UTM port forward).
  *Observe (finish line):* Safari or Chrome on your Mac opens
  `http://localhost:8080/` and shows a page served entirely by your stack.
- [ ] **6.8 Several connections**: `stack/app/http.py`
  Browsers open several connections and also ask for `/favicon.ico`. Make
  sure each connection is handled independently, and serve a favicon from
  `www/`.
  *Observe:* the trace shows two connections (page and favicon), each
  running through the full handshake → data → close sequence.

### Stretch

- [ ] **6.9 The page shows its own packets**: `stack/app/http.py`
  A `/trace` route returns the last N trace lines as HTML, so the page lists
  the frames, packets and segments that delivered it.
  *Observe:* reloading `/trace` shows the handshake of the request that
  fetched it.
- [ ] **6.10 Keep-alive**: `stack/app/http.py`
  Persistent connections (p. 503): honor `Connection: keep-alive` and serve
  several requests on one connection.
  *Observe:* `curl -v http://10.0.0.2/ http://10.0.0.2/` shows the second
  request reusing the connection.

## Concepts covered

The client/server model, URLs, the HTTP/1.1 message format (request line,
headers, CRLF, blank line, body), methods, status code groups, content types,
`Content-Length`, persistent connections, port forwarding vs. NAT, and how
every layer below supports one page load.
