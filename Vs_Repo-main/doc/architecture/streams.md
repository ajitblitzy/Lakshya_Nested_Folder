# Streams

> Feature: F-006 — Streams (`node:stream`, `node:stream/web`, `node:stream/promises`,
> `node:stream/consumers`)

This page deep-dives into the streaming abstractions of Node.js v26.0.0-pre. It covers the four
classic stream types (Readable, Writable, Duplex, Transform), the backpressure mechanism, the pipe
and pipeline operators, and the WHATWG Web Streams alignment exposed under `node:stream/web`. The
version stamps are `NODE_MAJOR_VERSION 26`, `NODE_MODULE_VERSION 144`.

For an overview of the runtime layers, see [Architecture overview](overview.md). For Web Standards
alignment, see [Web standards](web-standards.md). For specific stream consumers (`node:fs` streams,
HTTP streams, Inspector streams), see [File system](file-system.md), [Networking](networking.md),
and [Diagnostics](diagnostics.md).

## Stream types

The four classic stream types in `node:stream`:

| Type | Direction | Purpose | Example |
| --- | --- | --- | --- |
| **Readable** | producer | Source of data | `fs.createReadStream(path)`, `process.stdin`, `http.IncomingMessage` |
| **Writable** | consumer | Sink for data | `fs.createWriteStream(path)`, `process.stdout`, `http.ServerResponse` |
| **Duplex** | both | Independent read and write halves | `net.Socket`, `tls.TLSSocket` |
| **Transform** | both | Duplex where output depends on input | `zlib.createGzip()`, `crypto.createCipheriv(...)` |

The full surface is at [`../api/stream.md`](../api/stream.md). Iterable-stream wrappers (using
`AsyncIterator`) are at [`../api/stream_iter.md`](../api/stream_iter.md).

## Backpressure

`Readable` streams produce data, and `Writable` streams consume it. When a Writable's internal
buffer reaches `highWaterMark`, `write()` returns `false`, signaling the upstream Readable to pause.
Once the consumer drains, the Writable emits `'drain'`, signaling the Readable to resume.

The default `highWaterMark` is `16 KiB` for byte streams and `16` objects for object-mode streams,
configurable per stream instance. The full backpressure semantics, including object-mode streams,
async iterables, and the buffering algorithm, are documented in [`../api/stream.md`](../api/stream.md).

## Composition

Streams compose through three idioms:

| Idiom                                         | Module                 | Use case                                  |
| --------------------------------------------- | ---------------------- | ----------------------------------------- |
| `readable.pipe(writable)`                     | `node:stream`          | Classic; does not propagate errors        |
| `pipeline(readable, ..., writable, callback)` | `node:stream`          | Modern, error-propagating; recommended    |
| `await pipeline(readable, ..., writable)`     | `node:stream/promises` | Async/await variant of `pipeline`         |

The `pipeline()` and `compose()` operators are implemented at `lib/internal/streams/pipeline.js` and
`lib/internal/streams/compose.js`. Async iterators integrate naturally with both forms.

```mjs
import { pipeline } from 'node:stream/promises';
import { createReadStream, createWriteStream } from 'node:fs';
import { createGzip } from 'node:zlib';

await pipeline(
  createReadStream('input.txt'),
  createGzip(),
  createWriteStream('input.txt.gz'),
);
```

```cjs
const { pipeline } = require('node:stream/promises');
const { createReadStream, createWriteStream } = require('node:fs');
const { createGzip } = require('node:zlib');

pipeline(
  createReadStream('input.txt'),
  createGzip(),
  createWriteStream('input.txt.gz'),
).then(() => { /* done */ });
```

## Internal layout (`lib/internal/streams/`)

The implementation files under `lib/internal/streams/`:

| File | Purpose |
| --- | --- |
| `lib/internal/streams/add-abort-signal.js` | Wires `AbortSignal` cancellation into a stream |
| `lib/internal/streams/compose.js` | The `stream.compose()` operator |
| `lib/internal/streams/destroy.js` | Common destruction logic |
| `lib/internal/streams/duplex.js`, `duplexify.js`, `duplexpair.js` | Duplex stream constructors |
| `lib/internal/streams/end-of-stream.js` | The `finished()` predicate |
| `lib/internal/streams/fast-utf8-stream.js` | UTF-8 decoder optimized for streaming |
| `lib/internal/streams/from.js` | `Readable.from(iterable)` constructor |
| `lib/internal/streams/iter` | AsyncIterator integration helpers |
| `lib/internal/streams/lazy_transform.js` | Lazily-instantiated transform |
| `lib/internal/streams/legacy.js` | Backwards-compatibility shims for v0.10-style streams |
| `lib/internal/streams/operators.js` | Higher-order helpers (`map`, `filter`, `take`, etc.) |
| `lib/internal/streams/passthrough.js` | The `PassThrough` stream class |
| `lib/internal/streams/pipeline.js` | The `pipeline()` operator |

## Iterable streams

`node:stream/iter` exposes async-iterable wrappers for Readable streams. The full surface is at
[`../api/stream_iter.md`](../api/stream_iter.md). Modern code typically prefers iterating a Readable
directly with `for await (const chunk of readable)` rather than the explicit `iter` API.

## Web Streams (`node:stream/web`)

The WHATWG Streams Standard surface is exposed at [`../api/webstreams.md`](../api/webstreams.md):
`ReadableStream`, `WritableStream`, `TransformStream`, `ByteLengthQueuingStrategy`, and
`CountQueuingStrategy`. Implementation lives at `lib/internal/webstreams/`:

| File | Purpose |
| --- | --- |
| `lib/internal/webstreams/adapters.js` | Bridges between Node-style and Web-style streams |
| `lib/internal/webstreams/compression.js` | `CompressionStream`, `DecompressionStream` |
| `lib/internal/webstreams/encoding.js` | `TextEncoderStream`, `TextDecoderStream` |
| `lib/internal/webstreams/queuingstrategies.js` | The two queuing-strategy classes |
| `lib/internal/webstreams/readablestream.js` | `ReadableStream` and its readers and controllers |
| `lib/internal/webstreams/transfer.js` | Transferable-stream support across `MessagePort` |
| `lib/internal/webstreams/transformstream.js` | `TransformStream` |
| `lib/internal/webstreams/util.js` | Shared utilities |
| `lib/internal/webstreams/writablestream.js` | `WritableStream` and its writers and controllers |

`Readable.toWeb()` and `Readable.fromWeb()` (and their Writable counterparts) bridge between the two
worlds; they live at `lib/internal/webstreams/adapters.js`. The full alignment with Web Standards is
documented in [Web standards](web-standards.md).

## Stream consumers

Many runtime APIs return or accept streams:

| API | Stream type | Reference |
| --- | --- | --- |
| `node:fs` create-read/write stream | Node Readable / Writable | [`../api/fs.md`](../api/fs.md) |
| `process.stdin`, `stdout`, `stderr` | Node Readable / Writable | [`../api/process.md`](../api/process.md) |
| `node:http` `IncomingMessage`, `ServerResponse` | Node Readable / Writable | [`../api/http.md`](../api/http.md) |
| `node:zlib` `Gzip`, `Deflate`, `BrotliCompress`, `ZstdCompress` | Transform | [`../api/zlib.md`](../api/zlib.md) |
| `node:net` `Socket` | Node Duplex | [`../api/net.md`](../api/net.md) |
| `node:tls` `TLSSocket` | Node Duplex | [`../api/tls.md`](../api/tls.md) |
| `fetch()` `Response.body` | Web `ReadableStream` | [`../api/globals.md`](../api/globals.md) |
| `node:zlib` iterable compression API | Node Readable | [`../api/zlib_iter.md`](../api/zlib_iter.md) |

## Cross-references

* Architectural overview: [Architecture overview](overview.md)
* Feature catalog: [Feature catalog](features.md)
* Integration narrative: [Integration](integration.md)
* JavaScript runtime (event loop, async iterators): [JavaScript runtime](runtime.md)
* Web Standards (alignment with WHATWG Streams): [Web standards](web-standards.md)
* File system (stream consumers): [File system](file-system.md)
* Networking (stream consumers): [Networking](networking.md)
* Public APIs:
  * [`../api/stream.md`](../api/stream.md)
  * [`../api/stream_iter.md`](../api/stream_iter.md)
  * [`../api/webstreams.md`](../api/webstreams.md)
  * [`../api/zlib.md`](../api/zlib.md)
  * [`../api/zlib_iter.md`](../api/zlib_iter.md)
  * [`../api/buffer.md`](../api/buffer.md)
  * [`../api/string_decoder.md`](../api/string_decoder.md)
