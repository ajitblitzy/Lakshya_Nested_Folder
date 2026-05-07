# File system

> Feature: F-003 — File system (`node:fs`, `node:fs/promises`, `node:path`)

This page deep-dives into the file-system surface of Node.js v26.0.0-pre. It covers the three API
styles (callback, sync, promises), the libuv abstraction across Linux, macOS, and Windows, file
watchers, and permission-model integration. The version stamps are `NODE_MAJOR_VERSION 26`,
`NODE_MODULE_VERSION 144`.

For an overview of the runtime layers, see [Architecture overview](overview.md). For dependency
details (libuv), see [Dependencies](dependencies.md). For permission enforcement, see
[Permission model](permission-model.md).

## API styles

`node:fs` exposes the same operations through three idiomatic styles. The `lib/fs.js` module is the
public face; `lib/internal/fs/` hosts the implementation, including the promise-based variant under
`lib/internal/fs/promises.js`.

| Style           | Import                              | Example operation                        |
| --------------- | ----------------------------------- | ---------------------------------------- |
| **Callback**    | `import fs from 'node:fs'`          | `fs.readFile('a.txt', cb)` (see below)   |
| **Synchronous** | `import fs from 'node:fs'`          | `fs.readFileSync('a.txt')` (see below)   |
| **Promises**    | `import fs from 'node:fs/promises'` | `await fs.readFile('a.txt')` (see below) |

Behavior of each style:

* **Callback** — non-blocking; the result is delivered to `cb(err, data)`.
* **Synchronous** — blocks the event loop; the result is returned directly.
* **Promises** — non-blocking; returns a `Promise`.

A typical pairing across all three styles:

```mjs
import fs from 'node:fs';
import fsp from 'node:fs/promises';

// Callback
fs.readFile('config.json', 'utf8', (err, contents) => {
  if (err) throw err;
});

// Sync
const contents = fs.readFileSync('config.json', 'utf8');

// Promises
const contents2 = await fsp.readFile('config.json', 'utf8');
```

```cjs
const fs = require('node:fs');
const fsp = require('node:fs/promises');

// Callback
fs.readFile('config.json', 'utf8', (err, contents) => {
  if (err) throw err;
});

// Sync
const contents = fs.readFileSync('config.json', 'utf8');

// Promises (top-level await unsupported in CJS)
fsp.readFile('config.json', 'utf8').then((contents2) => { /* ... */ });
```

The complete API surface is at [`../api/fs.md`](../api/fs.md).

## Cross-platform abstraction (libuv)

All non-blocking and synchronous file-system operations route through libuv, which abstracts the
platform-specific syscalls:

| Operation             | Linux                        | macOS             | Windows                            |
| --------------------- | ---------------------------- | ----------------- | ---------------------------------- |
| Open / read / write   | `open(2)` / `pread(2)`       | `open(2)`         | `CreateFileW` / `ReadFile`         |
| Directory listing     | `getdents64(2)`              | `readdir(3)`      | `FindFirstFileW` / `FindNextFileW` |
| Stat                  | `statx(2)` / `fstat(2)`      | `fstat(2)`        | `GetFileInformationByHandleEx`     |
| Watcher (single file) | `inotify(7)`                 | FSEvents          | `ReadDirectoryChangesW`            |
| Watcher (recursive)   | `inotify` (manual recursion) | FSEvents (native) | `ReadDirectoryChangesW` (native)   |

On macOS the watcher uses the FSEvents framework
(`<CoreServices/CoreServices.h>`); on Linux `statx(2)` is preferred when available, falling back
to `fstat(2)` otherwise. Recursive watchers are native on macOS and Windows; on Linux libuv
implements recursion manually on top of `inotify`.

Asynchronous operations are dispatched to libuv's worker thread pool (default size 4, configurable
via the `UV_THREADPOOL_SIZE` environment variable) so that the main event loop is never blocked.

## File watchers

Two watcher APIs are exposed:

| API                                     | Implementation                       | Use case                           |
| --------------------------------------- | ------------------------------------ | ---------------------------------- |
| `fs.watch(path, options, listener)`     | `lib/internal/fs/watchers.js`        | Watch a single file or directory   |
| `fs.watchFile(path, options, listener)` | `lib/internal/fs/watchers.js`        | Polling-based fallback (see notes) |
| `fs.promises.watch(path, options)`      | `lib/internal/fs/promises.js`        | AsyncIterable for promise-style    |
| `fs.watch(path, { recursive: true })`   | `lib/internal/fs/recursive_watch.js` | Recursive directory watching       |

Notes:

* `fs.watch` is implemented through libuv's `uv_fs_event_t` and dispatches platform-native events.
* `fs.watchFile` polls via `fs.stat()` and is the recommended fallback when native events are
  unavailable or unreliable.

The semantics differ subtly between platforms (e.g., the order of `'rename'` versus `'change'`
events). The full behavior table is in [`../api/fs.md`](../api/fs.md).

## Helpers

`lib/internal/fs/` ships several specialized helpers:

| Helper                                 | Purpose                                                    |
| -------------------------------------- | ---------------------------------------------------------- |
| `lib/internal/fs/cp/`                  | Recursive `fs.cp()` and `fs.cpSync()` implementation       |
| `lib/internal/fs/dir.js`               | Async iterator for directory listing (`fs.opendir()`)      |
| `lib/internal/fs/glob.js`              | `fs.glob()` and `fs.globSync()` implementation             |
| `lib/internal/fs/promises.js`          | The `node:fs/promises` module                              |
| `lib/internal/fs/read/`                | `fs.read()` and `fs.readSync()` paths                      |
| `lib/internal/fs/recursive_watch.js`   | Recursive `fs.watch()`                                     |
| `lib/internal/fs/rimraf.js`            | `fs.rm()` and `fs.rmSync()` recursive removal              |
| `lib/internal/fs/streams.js`           | `fs.createReadStream()`, `fs.createWriteStream()`          |
| `lib/internal/fs/sync_write_stream.js` | Synchronous write stream backing                           |
| `lib/internal/fs/utils.js`             | Path normalization, mode translation, and stat translation |
| `lib/internal/fs/watchers.js`          | `fs.watch()` and `fs.watchFile()`                          |

## Streams

`fs.createReadStream()` and `fs.createWriteStream()` produce stream instances that follow the
`node:stream` contract. For details on stream semantics (Readable, Writable, Transform, Duplex,
backpressure), see [Streams](streams.md) and [`../api/stream.md`](../api/stream.md).

## Permission Model interaction

The Permission Model gates file-system access through two flags:

| Flag               | Effect                                                                                  |
| ------------------ | --------------------------------------------------------------------------------------- |
| `--allow-fs-read`  | Permits read access; accepts `*` (any), an absolute path, a directory prefix, or a glob |
| `--allow-fs-write` | Permits write access; same accepted values                                              |

Examples:

```bash
# Allow reads from /etc/hosts only
node --permission --allow-fs-read='/etc/hosts' script.js

# Allow reads from anywhere under /data and writes to /tmp/output
node --permission --allow-fs-read='/data/*' --allow-fs-write='/tmp/output/*' script.js
```

When `--permission` is on and the requested path is not allowed, the operation throws
`ERR_ACCESS_DENIED`. Caveats around symlink resolution and file descriptor inheritance are
documented in [Permission model](permission-model.md). The CLI surface is in [`../api/cli.md`](../api/cli.md)
and the JS API is in [`../api/permissions.md`](../api/permissions.md).

## Path manipulation

Cross-platform path manipulation lives in `node:path` (`lib/path.js`), which is independent of the
file system but related: it exposes `path.posix`, `path.win32`, and `path` (the platform default).
See [`../api/path.md`](../api/path.md).

## Cross-references

* Architectural overview: [Architecture overview](overview.md)
* Feature catalog: [Feature catalog](features.md)
* Integration narrative: [Integration](integration.md)
* Bundled dependencies (libuv details): [Dependencies](dependencies.md)
* Streams (used by `fs.createReadStream`, etc.): [Streams](streams.md)
* Permission Model (`--allow-fs-read`, `--allow-fs-write`): [Permission model](permission-model.md)
* Public APIs:
  * [`../api/fs.md`](../api/fs.md)
  * [`../api/path.md`](../api/path.md)
  * [`../api/stream.md`](../api/stream.md)
  * [`../api/permissions.md`](../api/permissions.md)
  * [`../api/cli.md`](../api/cli.md)
* Build and platform requirements: [`../../BUILDING.md`](../../BUILDING.md)
