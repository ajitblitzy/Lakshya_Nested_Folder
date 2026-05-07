# Integration

This page narrates how code integrates with Node.js v26.0.0-pre across three personas — application
authors, addon authors, and embedders — and how the runtime integrates with external infrastructure
(continuous integration, code coverage, security scanning, vulnerability disclosure).

The version stamps here are `NODE_MAJOR_VERSION 26`, `NODE_MODULE_VERSION 144`,
`NODE_API_SUPPORTED_VERSION_MAX 10`, `NODE_VERSION_IS_RELEASE 0`, sourced from `src/node_version.h`.

## Integration personas

<!--lint disable fenced-code-flag-->

```text
+--------------------------+                          +---------------------------+
| Inbound integration      |                          | Outbound integration      |
|                          |                          |                           |
|  Application authors     | -- node script.js -----> |  GitHub Actions           |
|  JS / TS programs        |    import 'node:*'       |  (37 workflows)           |
|                          |                          |                           |
|  Addon authors           | -- napi_* C ABI -------> |  Codecov                  |
|  C / C++ via Node-API    |    NODE_API_SUPPORTED_   |  (coverage gating)        |
|                          |    VERSION_MAX = 10      |                           |
|                          |                          |  CodeQL                   |
|  Embedders               | -- src/api/ -----------> |  (SAST)                   |
|  C++ host applications   |    NODE_MODULE_VERSION   |                           |
|                          |    = 144                 |  OSSF Scorecard           |
|                          |                          |  (supply-chain hygiene)   |
|                          |                          |                           |
|                          |                          |  HackerOne                |
|                          |                          |  (vulnerability           |
|                          |                          |   disclosure)             |
+------------+-------------+                          +-------------+-------------+
             |                                                      ^
             v                                                      |
       +---------------------------------------------+              |
       |   Node.js v26.0.0-pre (the node binary)     |              |
       |   workflow runs / coverage upload / SAST    | -------------+
       |   scans / weekly scoring / coordinated      |
       |   disclosure                                |
       +---------------------------------------------+
```

<!--lint enable fenced-code-flag-->

## Persona 1 — Application authors

Application authors use the runtime through CLI invocation and JavaScript or TypeScript imports.

### CLI entry point

The runtime starts at `src/node_main.cc`, which parses command-line arguments per
[`../api/cli.md`](../api/cli.md), sets up per-process state, creates a V8 isolate, and runs the
specified script (or REPL session). Synopsis at [`../api/synopsis.md`](../api/synopsis.md).

```bash
# Run a script
node script.js

# Inline evaluation
node --eval "console.log(process.versions.node)"

# Built-in test runner
node --test

# REPL
node
```

### Module imports

Public APIs are imported via the `node:` URL scheme:

```mjs
import http from 'node:http';
import fs from 'node:fs/promises';
import { test } from 'node:test';
```

```cjs
const http = require('node:http');
const fs = require('node:fs/promises');
const { test } = require('node:test');
```

The full module catalog is in [`../api/index.md`](../api/index.md). The module-loader internals are
in [`runtime.md`](runtime.md), and the module-resolution rules are in
[`../api/packages.md`](../api/packages.md), [`../api/modules.md`](../api/modules.md), and
[`../api/esm.md`](../api/esm.md).

### Package format

Application packages are described by `package.json`. The `"type"` field selects between CommonJS
(`"type": "commonjs"`) and ECMAScript modules (`"type": "module"`). Conditional exports, scoped
packages, and the `"imports"` map are documented in [`../api/packages.md`](../api/packages.md).

### Permission model

When run with `--permission`, the runtime denies file system, network, child-process, and worker
access by default. Allow-flags re-grant scoped access:

```bash
node --permission --allow-fs-read='/etc/hosts' script.js
```

The full enforcement model is documented in [Permission model](permission-model.md).

## Persona 2 — Addon authors (Node-API)

Native addon authors integrate against the **Node-API** (formerly N-API) C ABI declared in
`src/node_api.h` and `src/js_native_api.h`. Addons are compiled as shared libraries and loaded via
`process.dlopen` (or, equivalently, `require('./build/Release/myaddon.node')`).

### ABI version range

`src/node_version.h` declares the supported Node-API ABI range:

| Constant                              | Value |
| ------------------------------------- | ----- |
| `NODE_API_SUPPORTED_VERSION_MIN`      | `1`   |
| `NODE_API_SUPPORTED_VERSION_MAX`      | `10`  |
| `NODE_API_DEFAULT_MODULE_API_VERSION` | `8`   |

An addon compiled against ABI version `N` (where `1 <= N <= 10`) is guaranteed to load and run on
this `node` binary. Newer Node.js releases extend the upper bound; addons that need a newer ABI must
declare a higher `NAPI_VERSION` macro before including `node_api.h`.

### Build configuration

The runtime defines `NAPI_EXPERIMENTAL=1` in `node.gyp` to expose Node-API experimental functions to
addons that opt in by predefining `NAPI_EXPERIMENTAL` themselves.

### Reference

The complete Node-API surface is documented in [`../api/n-api.md`](../api/n-api.md). The
addon-authoring tutorial (which covers both V8-direct addons and Node-API addons) is in
[`../api/addons.md`](../api/addons.md). The internal Node-API release process for the runtime team
is in [`../contributing/releases-node-api.md`](../contributing/releases-node-api.md), and the
workflow for adding new Node-API entries is in
[`../contributing/adding-new-napi-api.md`](../contributing/adding-new-napi-api.md).

## Persona 3 — Embedders (C++ embedder API)

Embedders link the runtime as a C++ library inside a host application (e.g., desktop applications,
managed runtimes, plugin hosts). The embedder API is declared in `src/node.h` and implemented in
`src/api/`.

### `src/api/` layout

| File                        | Subject                                                              |
| --------------------------- | -------------------------------------------------------------------- |
| `src/api/environment.cc`    | Per-process and per-instance lifecycle (see note below)              |
| `src/api/embed_helpers.cc`  | Helpers used by `embedtest` and other embedding samples              |
| `src/api/callback.cc`       | Calling JavaScript functions from C++ with proper error handling     |
| `src/api/async_resource.cc` | The `node::AsyncResource` helper for embedders scheduling async work |
| `src/api/encoding.cc`       | UTF-8 / Latin-1 / UCS-2 conversion helpers                           |
| `src/api/exceptions.cc`     | Translating system errors into JavaScript `Error` objects            |
| `src/api/hooks.cc`          | `BeforeExit` / `Exit` / `AtExit` hook registration                   |
| `src/api/utils.cc`          | Miscellaneous embedder utilities                                     |

The `src/api/environment.cc` file implements the core lifecycle entry points:
`InitializeNodeWithArgs`, `CreateEnvironment`, `LoadEnvironment`, and `FreeEnvironment`.

### ABI compatibility

Embedders link against `NODE_MODULE_VERSION 144`. Unlike Node-API, the embedder API is **not**
ABI-stable across major versions of Node.js — embedders must rebuild against each new major release.
The full embedder narrative, including the `embedtest.cc` walkthrough, is in
[`../api/embedding.md`](../api/embedding.md).

## External integrations

The runtime integrates with five external systems for development infrastructure.

### GitHub Actions

The repository ships **37** GitHub Actions workflow files under `.github/workflows/`. They cluster
into six categories: testing (`test-linux.yml`, `test-macos.yml`, `test-shared.yml`,
`test-internet.yml`, `daily.yml`), coverage (`coverage-linux.yml`, `coverage-linux-without-intl.yml`,
`coverage-windows.yml`), security (`codeql.yml`, `scorecard.yml`), linting (`linters.yml`,
`commit-lint.yml`, `lint-release-proposal.yml`, `license-builder.yml`), release
(`create-release-proposal.yml`, `major-release.yml`, `post-release.yml`, `build-tarball.yml`,
`doc.yml`), and maintenance (`update-v8.yml`, `update-openssl.yml`, `update-wpt.yml`,
`daily-wpt-fyi.yml`, `timezone-update.yml`, `tools.yml`, `commit-queue.yml`, `auto-start-ci.yml`,
plus stale-issue / inactive-collaborator workflows).

The workflows are cataloged in
[`../contributing/testing-overview.md`](../contributing/testing-overview.md).

### Codecov

Code coverage is uploaded to Codecov per the configuration in `codecov.yml`. The configuration:

* `after_n_builds: 2` — wait for both Linux variants before publishing the comment
* `project: off`, `patch: off` — coverage diff is informational, not blocking
* `require_changes: true` — comment only when coverage changes

Coverage thresholds are pinned in `.nycrc` at `lines: 95`, `branches: 93`, `statements: 95`.

### CodeQL

The `codeql.yml` workflow runs Static Application Security Testing (SAST) daily and on pushes to
security-relevant branches. It scans JavaScript, C/C++, and Python sources for known
vulnerability patterns. Results surface in the GitHub security tab.

### OSSF Scorecard

The `scorecard.yml` workflow emits a project-health score on a weekly schedule. The Scorecard
report covers code review, branch protection, dependency update tooling, and other supply-chain
hygiene metrics consumable by downstream auditors.

### HackerOne

Coordinated vulnerability disclosure is handled through HackerOne. The disclosure policy and
severity matrix are documented in [`../../SECURITY.md`](../../SECURITY.md), and the security
release process for collaborators is in
[`../contributing/security-release-process.md`](../contributing/security-release-process.md).

## Stability summary

| Surface                                 | ABI stability                            |
| --------------------------------------- | ---------------------------------------- |
| Public API (`lib/*.js`)                 | Semver-stable; see Stability index below |
| Internal API (`lib/internal/`)          | Unstable; off-limits to user code        |
| Node-API (`napi_*`)                     | ABI-stable across major versions         |
| Embedder API (`src/api/`, `src/node.h`) | Major-version-stable only                |

Notes for each surface:

* **Public API (`lib/*.js`)** — Semver-stable per the
  [Stability index policy](../api/documentation.md). Stability levels are declared per-API in the
  reference pages under [`../api/`](../api/).
* **Internal API (`lib/internal/`)** — Off-limits to user code; semantics may change in any
  release per the [Internal API guide](../contributing/internal-api.md).
* **Node-API (`napi_*`)** — Supported version range `[1, 10]`; an addon built for version N runs
  on this runtime if `N` falls in the supported range.
* **Embedder API (`src/api/`, `src/node.h`)** — Embedders must rebuild against each new major
  release; `NODE_MODULE_VERSION 144` for v26.

## Cross-references

* Architectural overview: [Architecture overview](overview.md)
* Feature catalog: [Feature catalog](features.md)
* Bundled dependencies: [Dependencies](dependencies.md)
* JavaScript runtime details: [JavaScript runtime](runtime.md)
* Permission enforcement: [Permission model](permission-model.md)
* Public API index: [`../api/index.md`](../api/index.md)
* Node-API reference: [`../api/n-api.md`](../api/n-api.md)
* C++ embedder API: [`../api/embedding.md`](../api/embedding.md)
* Native addon tutorial: [`../api/addons.md`](../api/addons.md)
* Stability index policy: [`../api/documentation.md`](../api/documentation.md)
* Development workflow index: [`../contributing/development-overview.md`](../contributing/development-overview.md)
* Testing workflow index: [`../contributing/testing-overview.md`](../contributing/testing-overview.md)
* Internal API contract: [`../contributing/internal-api.md`](../contributing/internal-api.md)
* Adding a Node-API entry: [`../contributing/adding-new-napi-api.md`](../contributing/adding-new-napi-api.md)
* Node-API release process: [`../contributing/releases-node-api.md`](../contributing/releases-node-api.md)
* Security release process: [`../contributing/security-release-process.md`](../contributing/security-release-process.md)
* Project security policy: [`../../SECURITY.md`](../../SECURITY.md)
* Build and platform requirements: [`../../BUILDING.md`](../../BUILDING.md)
