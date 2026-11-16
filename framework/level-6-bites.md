# Level 6: Application layer, an HTTP server (Tanenbaum ch. 7)

**Goal:** an HTTP/1.1 server running on your own TCP, IP, ARP and Ethernet
code, reachable from `curl` and then from a browser on your Mac.

**Runs on:** the Linux box from level 3. In bite 6.7 the browser reaches it
through a port forward.

**Builds on:** `tcp.listen(port, on_data)`, `send`, and `close` from level 5.

**Current bite:** not started

## Bites

### Reading requests

- [ ] **6.1 Collect the request**: `stack/app/http.py`
  Buffer each connection's bytes until the blank line (`\r\n\r\n`) that ends
  the headers. Register the handler on port 80.
  *Observe:* `curl http://10.0.0.2/` hangs (there's no reply yet), but the
  stack prints the full request text curl sent.
- [ ] **6.2 Parse the request**: `stack/app/http.py`
  Split the request line into method, path and version, and the headers into
  a dict (header names are case-insensitive).
  *Observe:* the trace shows `[HTTP] GET / HTTP/1.1  Host=10.0.0.2`.

### Sending responses

- [ ] **6.3 Build a response**: `stack/app/http.py`
  `response(status, content_type, body)`: a status line, `Content-Type`,
  `Content-Length`, `Connection: close`, a blank line, then the body.
  *Observe:* a self-test prints the exact bytes, with every `\r\n` in place.
- [ ] **6.4 First page**: `stack/app/http.py`
  Answer every request with `200` and a short HTML body, then `close()`.
  *Observe:* `curl -v http://10.0.0.2/` shows your status line, headers and
  body.
- [ ] **6.5 Routes and 404**: `stack/app/http.py`
  Map paths to handlers. Unknown paths get `404 Not Found`.
  *Observe:* `curl -i http://10.0.0.2/nope` shows `404`.
- [ ] **6.6 Serve files**: `stack/app/http.py`
  Map `/x` to `www/x` (and `/` to `www/index.html`), with the content type
  chosen by file extension. Reject any path containing `..`.
  *Observe:* edit `www/index.html`, run `curl` again, and see the change.

### A real browser

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
  Honor `Connection: keep-alive` and serve several requests on one connection.
  *Observe:* `curl -v http://10.0.0.2/ http://10.0.0.2/` shows the second
  request reusing the connection.

## Concepts covered

The request/response model, HTTP/1.1 message format (request line, headers,
CRLF, body), status codes, content types, `Content-Length`, connection
handling, port forwarding and NAT-like relaying, and how every layer below
supports one page load.
