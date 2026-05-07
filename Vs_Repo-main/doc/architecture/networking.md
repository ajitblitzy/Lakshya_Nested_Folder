# Networking

> Feature: F-002 — Networking (HTTP/1.1, HTTP/2, HTTPS, TCP, UDP, DNS, TLS, QUIC, fetch)

This page deep-dives into the networking stack of Node.js v26.0.0-pre. It enumerates the supported
protocols, the bundled libraries that back them (llhttp, nghttp2, ngtcp2, nghttp3, c-ares, OpenSSL,
undici), the public API surface, and the permission-model interaction. The version stamps are
`NODE_MAJOR_VERSION 26`, `NODE_MODULE_VERSION 144`.

For an overview of the runtime layers, see [Architecture overview](overview.md). For dependency
details (libuv, OpenSSL, nghttp2, c-ares, undici, ngtcp2, nghttp3, llhttp), see
[Dependencies](dependencies.md). For permission enforcement, see [Permission model](permission-model.md).

## Protocol matrix

| Protocol                 | Public module                   | Backing library               |
| ------------------------ | ------------------------------- | ----------------------------- |
| TCP                      | [`node:net`](../api/net.md)     | libuv (`deps/uv/`)            |
| UDP                      | [`node:dgram`](../api/dgram.md) | libuv                         |
| DNS                      | [`node:dns`](../api/dns.md)     | c-ares (`deps/cares/`), libuv |
| TLS                      | [`node:tls`](../api/tls.md)     | OpenSSL (`deps/openssl/`)     |
| HTTP/1.1 (server)        | [`node:http`](../api/http.md)   | llhttp (`deps/llhttp/`)       |
| HTTP/1.1 (client)        | [`node:http`](../api/http.md)   | llhttp                        |
| HTTPS                    | [`node:https`](../api/https.md) | OpenSSL + llhttp              |
| HTTP/2                   | [`node:http2`](../api/http2.md) | nghttp2 (`deps/nghttp2/`)     |
| QUIC                     | [`node:quic`](../api/quic.md)   | ngtcp2 + nghttp3              |
| HTTP/3                   | (via `node:quic`)               | ngtcp2 + nghttp3              |
| Fetch / global `fetch()` | (global; delegates to undici)   | undici (`deps/undici/`)       |

The internal implementation surface for each protocol:

* **TCP** (`lib/net.js`) — pure libuv binding, no JS-level internals
* **UDP** (`lib/dgram.js`) — pure libuv binding
* **DNS** (`lib/dns.js`) — `lib/internal/dns/`
* **TLS** (`lib/tls.js`) — `lib/internal/tls/`, `lib/_tls_common.js`, `lib/_tls_wrap.js`
* **HTTP/1.1 server** (`lib/http.js`) — `lib/_http_server.js`, `lib/_http_incoming.js`,
  `lib/_http_outgoing.js`, `lib/_http_common.js`
* **HTTP/1.1 client** (`lib/http.js`) — `lib/_http_client.js`, `lib/_http_agent.js`
* **HTTPS** (`lib/https.js`) — composes `node:http` + `node:tls`
* **HTTP/2** (`lib/http2.js`) — `lib/internal/http2/`
* **QUIC** (`lib/quic.js`) — `lib/internal/quic/`, `src/quic/` (33 files)
* **HTTP/3** — `lib/internal/quic/`
* **Fetch** — delegates to undici through globals

## Inbound request flow

<!--lint disable fenced-code-flag-->

```text
Client                                                             Handler
  |                                                                  |
  | (1) TCP SYN  (or QUIC Initial over UDP)                          |
  |--------------------> TCP layer (libuv)                            |
  |                              |                                    |
  |                              | (2) Accept; optional TLS handshake |
  |                              v                                    |
  |                       TLS layer (OpenSSL)  -- optional --         |
  |                              |                                    |
  |                              | (3) ALPN selection                 |
  |                              v                                    |
  |                     Protocol selector                             |
  |                     /        |        \                           |
  |              http/1.1      h2          h3                         |
  |                /           |             \                        |
  |               v            v              v                       |
  |           HTTP/1.1      HTTP/2          HTTP/3                    |
  |           (llhttp)      (nghttp2)       (ngtcp2 + nghttp3)        |
  |              |             |               |                      |
  |              | emit         | emit          | emit                |
  |              | 'request'    | 'stream'      | 'session'/'stream'  |
  |              +-------------+----------------+                     |
  |                            |                                      |
  |                            v                                      |
  |                     User handler ---------------------------------+
  |                            |                                      |
  | <----------------- Write response (4)                             |
```

<!--lint enable fenced-code-flag-->

## Permission Model interaction

When the runtime is launched with `--permission`, network access is denied by default and must be
re-granted through the `--allow-net` flag. The full enforcement narrative is in
[Permission model](permission-model.md). The CLI surface is documented in [`../api/cli.md`](../api/cli.md)
and the JS API is in [`../api/permissions.md`](../api/permissions.md).

```bash
# Permission off (default) — all networking permitted
node server.js

# Permission on, allow all networking
node --permission --allow-net server.js

# Permission on, allow only outbound connections to specific hosts
node --permission --allow-net='example.com' client.js
```

## DNS resolution

`node:dns` exposes two resolution paths:

* **`dns.lookup()` family** — uses the OS-provided resolver via libuv (synchronous-style with thread
  pool offload); honors `/etc/hosts`, `/etc/resolv.conf`, mDNS, etc.
* **`dns.resolve*()` family and `Resolver` class** — uses the bundled c-ares library
  (`deps/cares/`) directly; bypasses OS resolver, useful for testing or for systems without the
  necessary OS APIs.

The full API matrix is at [`../api/dns.md`](../api/dns.md).

## TLS

TLS termination, client connections, and certificate handling all flow through OpenSSL via the
bindings in `src/crypto/`. The build toggle is `node_use_openssl: true` (see
[Dependencies](dependencies.md) and [`../../BUILDING.md`](../../BUILDING.md)). The `node:tls`
module surface is at [`../api/tls.md`](../api/tls.md). For an end-to-end view of cryptography in
the runtime, see [Cryptography](cryptography.md).

## QUIC and HTTP/3

QUIC is implemented via two bundled dependencies:

* **ngtcp2** (`deps/ngtcp2/`) — IETF QUIC transport (RFC 9000)
* **nghttp3** (`deps/nghttp3/`) — HTTP/3 application layer over QUIC (RFC 9114)

The native bindings are in `src/quic/` (33 files), exposed to JavaScript through `lib/quic.js` and
`lib/internal/quic/`. The public API surface is at [`../api/quic.md`](../api/quic.md). The QUIC API
is currently behind an experimental stability declaration; consult the API reference for the current
status.

## fetch() and undici

The global `fetch()` function (along with `Request`, `Response`, `Headers`, `FormData`, `Blob`) is
backed by **undici**, a modern HTTP/1.1 client maintained by the Node.js project and bundled at
`deps/undici/`. The bindings live under `lib/internal/`. For Web Standards alignment specifically,
see [Web standards](web-standards.md). For the public undici-derived APIs (when surfaced via the
`undici` external module), refer to the upstream undici project documentation linked from
[`../api/globals.md`](../api/globals.md).

## HTTP server lifecycle

The `http.Server` constructor creates a server bound to a `net.Server` (TCP listener). Connections
flow through:

1. `net.Server` accepts a TCP socket
2. The HTTP parser (llhttp) parses incoming bytes into requests
3. Each request emits a `'request'` event with `IncomingMessage` and `ServerResponse` instances
4. Responses are serialized through `_http_outgoing.js` and written to the socket

For HTTPS, the same flow applies with a TLS layer between steps 1 and 2. For HTTP/2, nghttp2 frames
replace llhttp parsing, and `'stream'` events replace `'request'`. The full API surface is at
[`../api/http.md`](../api/http.md), [`../api/https.md`](../api/https.md),
[`../api/http2.md`](../api/http2.md).

## Connection pooling

The HTTP/1.1 client uses an `Agent` (`lib/_http_agent.js`) to pool connections per host. The default
is a global agent with `keepAlive: false`; user code typically overrides this. The Agent settings
are documented at [`../api/http.md`](../api/http.md) under `http.Agent`.

## Cross-references

* Architectural overview: [Architecture overview](overview.md)
* Feature catalog: [Feature catalog](features.md)
* Integration narrative: [Integration](integration.md)
* Bundled dependencies (OpenSSL, nghttp2, c-ares, undici, ngtcp2, nghttp3, llhttp, libuv):
  [Dependencies](dependencies.md)
* Cryptography (TLS, OpenSSL details): [Cryptography](cryptography.md)
* Permission Model (`--allow-net`): [Permission model](permission-model.md)
* Web Standards (fetch, FormData, Headers, Request, Response): [Web standards](web-standards.md)
* Public APIs:
  * [`../api/http.md`](../api/http.md)
  * [`../api/http2.md`](../api/http2.md)
  * [`../api/https.md`](../api/https.md)
  * [`../api/net.md`](../api/net.md)
  * [`../api/dgram.md`](../api/dgram.md)
  * [`../api/dns.md`](../api/dns.md)
  * [`../api/tls.md`](../api/tls.md)
  * [`../api/quic.md`](../api/quic.md)
  * [`../api/cli.md`](../api/cli.md)
  * [`../api/permissions.md`](../api/permissions.md)
  * [`../api/globals.md`](../api/globals.md)
* Build flags reference: [`../../BUILDING.md`](../../BUILDING.md)
