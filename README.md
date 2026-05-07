# Ajit-backprop-test

A test project for backprop integration that packages the Node.js v26.0.0-pre runtime source tree under
`Vs_Repo-main/` (extracted from `Vs_Repo-main.zip`).

## About this repository

This repository serves as a test harness for backprop integration workflows. Its primary payload is a frozen snapshot
of the upstream Node.js project at version **v26.0.0-pre** (`NODE_MAJOR_VERSION 26`, `NODE_MINOR_VERSION 0`,
`NODE_PATCH_VERSION 0`, `NODE_VERSION_IS_RELEASE 0`, `NODE_MODULE_VERSION 144`, `NODE_API_SUPPORTED_VERSION_MAX 10`),
captured as the binary archive `Vs_Repo-main.zip`. After extraction, the archive expands to the standard Node.js
source tree under `Vs_Repo-main/`, including the `lib/`, `src/`, `test/`, `tools/`, `benchmark/`, `deps/`, and `doc/`
directories.

The repository is **documentation-only** with respect to the Node.js source: no source files, build scripts, tests,
or CI workflows under `Vs_Repo-main/` are modified. New documentation is added under `Vs_Repo-main/doc/architecture/`
(architecture and feature deep-dives) and `Vs_Repo-main/doc/contributing/` (development and testing overviews) to
explain the runtime's features, integration expectations, and the differentiation between development activities and
testing activities.

## Documentation map

| Document | Purpose |
| --- | --- |
| [Upstream Node.js README][upstream-readme] | Project README from the upstream Node.js source |
| [Build instructions][building-md] | Build from source on Linux, macOS, Windows, AIX, and other platforms |
| [Contribution guide][contributing-md] | Contribution overview and Developer Certificate of Origin (DCO) |
| [Architecture index][arch-readme] | Architecture and feature documentation index (NEW) |
| [Development overview][dev-overview] | Building, configuring, debugging, contributing, releasing workflows (NEW) |
| [Testing overview][test-overview] | Test execution, coverage, linting, benchmarks, and CI gates (NEW) |
| [API reference][api-ref] | Node.js API reference (68 files, one per public module) |

[upstream-readme]: Vs_Repo-main/README.md
[building-md]: Vs_Repo-main/BUILDING.md
[contributing-md]: Vs_Repo-main/CONTRIBUTING.md
[arch-readme]: Vs_Repo-main/doc/architecture/README.md
[dev-overview]: Vs_Repo-main/doc/contributing/development-overview.md
[test-overview]: Vs_Repo-main/doc/contributing/testing-overview.md
[api-ref]: Vs_Repo-main/doc/api/

## Repository layout

```text
.
├── README.md                — this file (root entry point)
├── Vs_Repo-main.zip         — the Node.js v26.0.0-pre source archive
└── Vs_Repo-main/            — extracted Node.js source tree (after extraction)
    ├── README.md            — upstream Node.js README
    ├── BUILDING.md          — build instructions
    ├── CONTRIBUTING.md      — contribution guide
    ├── LICENSE              — MIT license
    ├── lib/                 — public JavaScript API (54 modules)
    ├── src/                 — native C++ runtime (453 files)
    ├── test/                — test suite (11,060 entries; ~10,317 files)
    ├── tools/               — build/lint/release tooling
    ├── benchmark/           — performance benchmarks (591 entries; ~525 files)
    ├── deps/                — bundled dependencies (V8, libuv, OpenSSL, etc.)
    └── doc/                 — documentation tree
        ├── api/             — API reference (68 files)
        ├── architecture/    — architecture and feature docs (NEW, 15 files)
        └── contributing/    — contributor guides (45 existing + 2 NEW)
```

> Note: `Vs_Repo-main.zip` must be extracted before its contents are available as a file tree. The documentation
> under `Vs_Repo-main/doc/architecture/` and the new overviews under `Vs_Repo-main/doc/contributing/` are companion
> documents that explain what the extracted code does.

## Node.js feature summary

The Node.js runtime under `Vs_Repo-main/` exposes 13 documented features (F-001 — F-013):

* **F-001 JavaScript runtime** — V8 engine integration, libuv event loop, ESM and CommonJS module systems.
* **F-002 Networking** — HTTP/1.1 (llhttp), HTTP/2 (nghttp2), HTTPS, TCP, UDP, DNS (c-ares), TLS (OpenSSL), QUIC
  (ngtcp2/nghttp3), and `fetch()` via undici.
* **F-003 File system** — callback, sync, and promise-based APIs over libuv's cross-platform abstraction.
* **F-004 Cryptography** — OpenSSL-backed traditional `crypto` module plus Web Crypto via `crypto.subtle`.
* **F-005 Concurrency** — child processes, cluster mode, and worker threads (separate V8 isolates).
* **F-006 Streams** — Readable, Writable, Transform, Duplex, plus the Web Streams API.
* **F-007 Diagnostics** — Inspector protocol, perf hooks, diagnostics channel, trace events, async hooks.
* **F-008 Built-in test runner** — `node --test` with TAP output and coverage reporting.
* **F-009 TypeScript support** — Amaro (SWC WASM) in-process type stripping. In v26, type stripping is the default
  behavior for `.ts`, `.mts`, and `.cts` files (the legacy `--experimental-strip-types` flag is now a no-op); the
  earlier `--experimental-transform-types` flag has been removed in v26.
* **F-010 SQLite** — bundled SQLite database access via `node:sqlite`.
* **F-011 Single executable applications** — `node --experimental-sea-config` to bundle a runtime + script.
* **F-012 Permission model** — `--permission` flag with default-deny resource access (`--allow-fs-read`,
  `--allow-fs-write`, `--allow-net`, `--allow-child-process`, `--allow-worker`, `--allow-wasi`, `--allow-addons`).
* **F-013 Web standards** — `fetch`, Web Crypto, Web Streams, `Blob`, `FormData`, `Headers`, `Request`, `Response`.

For per-feature deep-dives and source-code mappings, see
[`Vs_Repo-main/doc/architecture/README.md`](Vs_Repo-main/doc/architecture/README.md) and the per-feature documents
under [`Vs_Repo-main/doc/architecture/`](Vs_Repo-main/doc/architecture/).

## Quick links

* Upstream Node.js README → [`Vs_Repo-main/README.md`][upstream-readme]
* Architecture index → [`Vs_Repo-main/doc/architecture/README.md`][arch-readme]
* Development overview → [`Vs_Repo-main/doc/contributing/development-overview.md`][dev-overview]
* Testing overview → [`Vs_Repo-main/doc/contributing/testing-overview.md`][test-overview]
* Build instructions → [`Vs_Repo-main/BUILDING.md`][building-md]
* Contribution guide → [`Vs_Repo-main/CONTRIBUTING.md`][contributing-md]

## License

The Node.js source tree under `Vs_Repo-main/` is licensed under the MIT License; see
[`Vs_Repo-main/LICENSE`](Vs_Repo-main/LICENSE) for the full license text. The new documentation added by this
repository inherits the same license terms.
