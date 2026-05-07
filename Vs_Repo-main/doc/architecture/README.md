# Architecture overview

This directory hosts the architecture and feature documentation for **Node.js v26.0.0-pre**
(`NODE_MAJOR_VERSION 26`, `NODE_MODULE_VERSION 144`, `NODE_API_SUPPORTED_VERSION_MAX 10`,
`NODE_VERSION_IS_RELEASE 0`), as recorded in `src/node_version.h`. It complements, rather than
replaces, the canonical API references under [`../api/`](../api/) and the contributor guides under
[`../contributing/`](../contributing/). Every document here cross-links into those existing trees
rather than duplicating their content.

## Purpose

The pages in this directory answer four questions that span feature boundaries:

1. **What does the runtime look like end to end?** — see [Architecture overview](overview.md)
2. **What features are shipped and where do they live?** — see [Feature catalog](features.md) plus
   the per-feature deep-dives
3. **How does my code integrate with Node.js?** — see [Integration](integration.md)
4. **Which third-party dependencies are bundled and how?** — see [Dependencies](dependencies.md)

## Reading order

The recommended path through the documentation is:

1. Start with [Architecture overview](overview.md) for the layered picture of the runtime
2. Survey the [Feature catalog](features.md) to locate features by tier and source path
3. Drill into the per-feature deep-dives for the features relevant to your task
4. Read [Integration](integration.md) when authoring an addon, embedding the runtime, or wiring up
   external infrastructure (CI, security, distribution)
5. Read [Dependencies](dependencies.md) when bumping a bundled dependency, debugging a third-party
   library, or evaluating a build-flag change

## Index

| Document                                | Subject                                                                                                                                                                                                    |
| --------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| [Architecture overview](overview.md)    | The 5-layer architectural model: User Space → public `lib/` → internal `lib/internal/` → native `src/` → bundled `deps/`, with design principles and version stamps                                        |
| [Feature catalog](features.md)          | All 13 runtime features (F-001 through F-013) catalogued by tier, with source paths and links to per-feature deep-dives                                                                                    |
| [JavaScript runtime](runtime.md)        | F-001 — V8 integration (embedder string `-node.17`), libuv 6-phase event loop, ESM/CJS module systems, JIT pipeline (Ignition → SparkPlug → Maglev → TurboFan)                                             |
| [Networking](networking.md)             | F-002 — protocol matrix (HTTP/1.1 via llhttp, HTTP/2 via nghttp2, HTTPS, TCP/UDP via libuv, DNS via c-ares, TLS via OpenSSL, QUIC via ngtcp2/nghttp3, `fetch()` via undici)                                |
| [File system](file-system.md)           | F-003 — three API styles (callback, sync, promises), libuv abstraction across Linux/macOS/Windows, watchers, permission-model interaction                                                                  |
| [Cryptography](cryptography.md)         | F-004 — OpenSSL backing (`node_use_openssl: true`), traditional `crypto` versus Web Crypto API, FIPS-compliant build mode, `src/crypto/` structural breakdown                                              |
| [Concurrency](concurrency.md)           | F-005 — worker threads (separate V8 isolates), child processes (`exec`, `execFile`, `fork`, `spawn`), cluster mode (round-robin scheduling), permission-model interaction                                  |
| [Streams](streams.md)                   | F-006 — Readable, Writable, Transform, and Duplex types, backpressure semantics, Web Streams alignment                                                                                                     |
| [Diagnostics](diagnostics.md)           | F-007 — Inspector protocol (Chrome DevTools), perf hooks, diagnostics channel pub/sub, trace events, async hooks; covers `src/inspector/` and `src/tracing/`                                               |
| [TypeScript](typescript.md)             | F-009 — Amaro (SWC WASM) type stripping, source-map generation, `node_use_amaro` build toggle                                                                                                              |
| [Permission model](permission-model.md) | F-012 — default-deny model, allow flags (`--allow-fs-read`, `--allow-fs-write`, `--allow-child-process`, `--allow-worker`, `--allow-wasi`, `--allow-addons`, `--allow-net`), `src/permission/` enforcement |
| [Web standards](web-standards.md)       | F-013 — `fetch()` via undici, Web Streams, Web Crypto via `crypto.subtle`, `FormData`, `Blob`, `Headers`, `Request`, `Response` globals                                                                    |
| [Integration](integration.md)           | Three-persona integration narrative (application authors, addon authors via N-API, embedders via `src/api/`); ABI compatibility (`NODE_MODULE_VERSION 144`); external infrastructure integration           |
| [Dependencies](dependencies.md)         | Bundled dependency integration matrix — V8, libuv, OpenSSL, nghttp2, c-ares, undici, ICU, SQLite, Amaro, simdutf, zlib, brotli, zstd, ngtcp2, nghttp3                                                      |

## Cross-references

* API reference index: [`../api/index.md`](../api/index.md)
* Development workflow index: [`../contributing/development-overview.md`](../contributing/development-overview.md)
* Testing workflow index: [`../contributing/testing-overview.md`](../contributing/testing-overview.md)
* Build and platform requirements: [`../../BUILDING.md`](../../BUILDING.md)
* Project contribution policy: [`../../CONTRIBUTING.md`](../../CONTRIBUTING.md)
* Documentation style guide: [`../README.md`](../README.md)
* Glossary of terms: [`../../glossary.md`](../../glossary.md)

## Conventions used in this directory

* Every architectural claim cites the relevant source file or directory by inline backticks (e.g.,
  `` `lib/http.js` ``, `` `src/crypto/` ``, `` `common.gypi` ``)
* Diagrams are inline Mermaid fenced blocks; no external image files are introduced
* Cross-links between architecture pages use plain filenames; cross-links into `doc/api/` use
  `../api/<file>.md`; cross-links into `doc/contributing/` use `../contributing/<file>.md`; links
  to top-level repository files use `../../<file>.md`
* The version of Node.js documented here is **v26.0.0-pre**, as captured by the constants in
  `src/node_version.h`. Future revisions of this directory should re-stamp those constants.
