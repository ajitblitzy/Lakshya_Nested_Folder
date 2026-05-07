# Web standards

> Feature: F-013 — Web standards (`fetch`, `Request`, `Response`, `Headers`, `FormData`, `Blob`,
> `crypto.subtle`, Web Streams)

This page deep-dives into Node.js v26.0.0-pre's Web Platform alignment. The runtime ships a growing
set of WHATWG and W3C web-standard APIs as global constructors, allowing isomorphic JavaScript that
runs unmodified in browsers and on Node.js. The version stamps are `NODE_MAJOR_VERSION 26`,
`NODE_MODULE_VERSION 144`, `NODE_API_SUPPORTED_VERSION_MAX 10`, `NODE_VERSION_IS_RELEASE 0`, as
recorded in `src/node_version.h`.

For an overview of the runtime layers, see [Architecture overview](overview.md). For Web Streams in
the broader streams story, see [Streams](streams.md). For Web Crypto in the broader cryptography
story, see [Cryptography](cryptography.md). For the runtime's networking stack including
`fetch()`, see [Networking](networking.md).

## Web Platform surface exposed as globals

The runtime exposes the following Web Platform interfaces as globals — no `import` or `require`
required:

| Interface | Purpose | Backing |
| --- | --- | --- |
| `fetch` | HTTP/1.1 client (and HTTP/2 fallback) | undici (bundled under `deps/undici/` in the upstream Node.js tree) |
| `Request`, `Response`, `Headers` | Fetch primitives | undici |
| `FormData` | Multipart form data | undici |
| `Blob`, `File` | Blob-of-bytes container | `lib/internal/blob.js`, `lib/internal/file.js` |
| `URL`, `URLSearchParams`, `URLPattern` | WHATWG URL | ada (bundled under `deps/ada/` in the upstream Node.js tree) |
| `TextEncoder`, `TextDecoder` | UTF-8 encoding/decoding | (V8) |
| `TextEncoderStream`, `TextDecoderStream` | Stream encoding/decoding | `lib/internal/webstreams/encoding.js` |
| `ReadableStream`, `WritableStream`, `TransformStream` | WHATWG Streams | `lib/internal/webstreams/` |
| `ByteLengthQueuingStrategy` | Byte queuing strategy | `lib/internal/webstreams/queuingstrategies.js` |
| `CountQueuingStrategy` | Count queuing strategy | `lib/internal/webstreams/queuingstrategies.js` |
| `CompressionStream`, `DecompressionStream` | Stream-based compression | `lib/internal/webstreams/compression.js` |
| `crypto`, `crypto.subtle`, `SubtleCrypto` | Web Cryptography API | `lib/crypto.js` + `src/crypto/` |
| `crypto.randomUUID()` | UUIDv4 generator | `src/crypto/` |
| `MessageChannel`, `MessagePort` | Web Messaging | `lib/internal/worker/io.js` |
| `BroadcastChannel` | Worker-to-worker pub/sub | `lib/internal/worker/messaging.js` |
| `EventTarget`, `Event` | DOM-style event dispatch | `lib/internal/event_target.js` |
| `AbortController`, `AbortSignal` | Web cancellation primitive | `lib/internal/abort_controller.js` |
| `performance` | W3C User Timing | `lib/internal/perf/` |
| `queueMicrotask` | Microtask scheduling | (V8) |
| `structuredClone` | Structured-clone algorithm | `lib/internal/structured_clone.js` (where present) |

The full catalog of globals (including the few CommonJS-only and ESM-only differences) is at
[`../api/globals.md`](../api/globals.md).

## fetch and undici

The global `fetch()` function and the `Request`, `Response`, `Headers`, `FormData` classes are
backed by **undici**, a modern HTTP/1.1 client maintained by the Node.js project and bundled in the
upstream Node.js tree under `deps/undici/`. (In this repository's source archive the `deps/` tree is
omitted; the bundled copy is identified through `src/undici_version.h` and the update automation in
`tools/dep_updaters/update-undici.sh`.) The bindings live under `lib/internal/`. The undici project
also ships a richer public API (e.g., `Pool`, `Agent`, `MockAgent`, `Dispatcher`, `Client`) that user
code can opt into by `npm install undici`; the bundled copy is the global-fetch surface only. For
details, see the upstream undici project linked from [`../api/globals.md`](../api/globals.md).

```mjs
// Global fetch, no import required
const res = await fetch('https://example.com/data.json');
const json = await res.json();
```

```cjs
// Global fetch, no require required
fetch('https://example.com/data.json')
  .then((res) => res.json())
  .then((json) => { /* ... */ });
```

For the broader networking stack and the protocol selection between HTTP/1.1, HTTP/2, and QUIC, see
[Networking](networking.md).

## Web Streams

The WHATWG Streams Standard surface (`ReadableStream`, `WritableStream`, `TransformStream`,
queuing strategies) lives at `lib/internal/webstreams/` and is documented in detail at
[`../api/webstreams.md`](../api/webstreams.md). For a side-by-side comparison with Node Streams
(`Readable`, `Writable`, `Duplex`, `Transform`) and the bridging adapters between the two, see
[Streams](streams.md).

`CompressionStream` and `DecompressionStream` (also part of the Streams Standard) live at
`lib/internal/webstreams/compression.js` and back the `gzip`, `deflate`, `deflate-raw` formats from
the WHATWG specification. For Node.js-style compression (Gzip / Deflate / BrotliCompress / Zstd
streams), see [`../api/zlib.md`](../api/zlib.md).

`TextEncoderStream` and `TextDecoderStream` live at `lib/internal/webstreams/encoding.js`.

## Web Crypto API

`globalThis.crypto.subtle` exposes the W3C Web Cryptography API. The implementation shares the
OpenSSL-backed primitives with the traditional `node:crypto` module and is documented at
[`../api/webcrypto.md`](../api/webcrypto.md). For the full cryptography story (including the
non-Web-Crypto `node:crypto` API surface), see [Cryptography](cryptography.md).

```mjs
const data = new TextEncoder().encode('input');
const digest = await globalThis.crypto.subtle.digest('SHA-256', data);
```

```cjs
const data = new TextEncoder().encode('input');
globalThis.crypto.subtle.digest('SHA-256', data).then((digest) => { /* ... */ });
```

`crypto.randomUUID()`, `crypto.getRandomValues()` are also exposed as globals.

## Blob and File

`globalThis.Blob` (a sized chunk of binary data) and `globalThis.File` (a `Blob` with a name and
last-modified timestamp) live at `lib/internal/blob.js` and `lib/internal/file.js`. They are
interoperable with `fetch()` request bodies, `FormData` entries, and the `node:stream` and Web
Streams worlds (e.g., `blob.stream()` returns a `ReadableStream`). For details, see
[`../api/buffer.md`](../api/buffer.md) under "Class: Blob" and "Class: File".

## URL, URLSearchParams, URLPattern

The runtime ships a fully-WHATWG-compliant URL parser backed by the **ada** library bundled in the
upstream Node.js tree under `deps/ada/`. (In this repository's source archive the `deps/` tree is
omitted; the bundled copy is tracked through `tools/dep_updaters/` automation.) The full surface
(including encoding, percent-encoding, IDNA / Punycode handling) is at
[`../api/url.md`](../api/url.md).

## Worker-platform alignment

`MessageChannel`, `MessagePort`, `BroadcastChannel` align Node.js's `node:worker_threads` semantics
with the Web Workers specification. See [Concurrency](concurrency.md) and
[`../api/worker_threads.md`](../api/worker_threads.md).

## EventTarget and AbortController

`EventTarget` and `Event` provide a DOM-style event-dispatch surface across many runtime APIs.
`AbortController` and `AbortSignal` are the canonical web cancellation primitive and are accepted
by `fetch()`, `node:fs/promises` operations, `node:timers/promises`, `node:stream` `pipeline()`, and
others. See [`../api/events.md`](../api/events.md) and [`../api/globals.md`](../api/globals.md).

## Performance, queueMicrotask, structuredClone

`globalThis.performance` is the same `Performance` instance documented in [Diagnostics](diagnostics.md)
and [`../api/perf_hooks.md`](../api/perf_hooks.md). `queueMicrotask` schedules a microtask in the
same queue used for Promise jobs (see [JavaScript runtime](runtime.md) for microtask interleaving).
`structuredClone` implements the structured-clone algorithm used by `MessagePort.postMessage` and
`BroadcastChannel`.

## Web Platform Tests integration

The runtime's compliance with the Web Platform standards is validated by the Web Platform Tests
(WPT) suite under `test/wpt/`. Two CI workflows back the integration: `daily-wpt-fyi.yml` (daily
run + upload to wpt.fyi) and `update-wpt.yml` (auto-PR to vendor the latest tests). See
[`../contributing/testing-overview.md`](../contributing/testing-overview.md).

## Cross-references

* Architectural overview: [Architecture overview](overview.md)
* Feature catalog: [Feature catalog](features.md)
* Integration narrative: [Integration](integration.md)
* JavaScript runtime (microtask queue, V8): [JavaScript runtime](runtime.md)
* Networking (fetch, undici, HTTP/2, QUIC): [Networking](networking.md)
* Cryptography (Web Crypto, OpenSSL): [Cryptography](cryptography.md)
* Streams (Web Streams alignment): [Streams](streams.md)
* Concurrency (MessageChannel, BroadcastChannel): [Concurrency](concurrency.md)
* Diagnostics (`performance`): [Diagnostics](diagnostics.md)
* Bundled dependencies (undici, ada): [Dependencies](dependencies.md)
* Public APIs:
  * [`../api/globals.md`](../api/globals.md)
  * [`../api/webcrypto.md`](../api/webcrypto.md)
  * [`../api/webstreams.md`](../api/webstreams.md)
  * [`../api/buffer.md`](../api/buffer.md) (Blob, File)
  * [`../api/url.md`](../api/url.md)
  * [`../api/events.md`](../api/events.md)
  * [`../api/perf_hooks.md`](../api/perf_hooks.md)
  * [`../api/worker_threads.md`](../api/worker_threads.md)
* Testing workflow index (WPT): [`../contributing/testing-overview.md`](../contributing/testing-overview.md)
