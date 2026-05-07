# Concurrency

> Feature: F-005 — Concurrency (`node:worker_threads`, `node:child_process`, `node:cluster`)

This page deep-dives into the three concurrency primitives of Node.js v26.0.0-pre: worker threads,
child processes, and cluster mode. The version stamps are `NODE_MAJOR_VERSION 26`,
`NODE_MODULE_VERSION 144`.

For an overview of the runtime layers, see [Architecture overview](overview.md). For the event-loop
and V8 isolate fundamentals that underlie this discussion, see [JavaScript runtime](runtime.md).
For permission enforcement, see [Permission model](permission-model.md).

## Three concurrency models

| Model | Public module | Isolation | Communication |
| --- | --- | --- | --- |
| **Worker threads** | [`node:worker_threads`](../api/worker_threads.md) (`lib/worker_threads.js`) | Separate V8 isolate, separate libuv loop, same OS process | `MessagePort`, `BroadcastChannel`, `SharedArrayBuffer`, `Atomics`, structured-clone messaging |
| **Child processes** | [`node:child_process`](../api/child_process.md) (`lib/child_process.js`) | Separate OS process | stdin/stdout/stderr pipes, optional IPC channel for `child_process.fork()` |
| **Cluster** | [`node:cluster`](../api/cluster.md) (`lib/cluster.js`) | Separate OS processes (built on `child_process.fork()`) | Round-robin or OS-default port distribution; IPC channel between primary and workers |

## Worker threads

`node:worker_threads` runs JavaScript on a separate V8 isolate, with its own libuv event loop and
its own JavaScript heap, but inside the same OS process as the spawning thread. Public API at
[`../api/worker_threads.md`](../api/worker_threads.md). Internal implementation at
`lib/internal/worker/`:

| File | Purpose |
| --- | --- |
| `lib/internal/worker/clone_dom_exception.js` | Cross-isolate `DOMException` cloning helpers |
| `lib/internal/worker/io.js` | Per-worker stdin/stdout/stderr wrappers |
| `lib/internal/worker/js_transferable.js` | Identifies and packages JS-transferable objects |
| `lib/internal/worker/messaging.js` | Structured-clone messaging glue |

```mjs
import { Worker, parentPort } from 'node:worker_threads';

if (process.argv[2] === 'worker') {
  parentPort.postMessage('hello from worker');
} else {
  const w = new Worker(import.meta.filename, { argv: ['worker'] });
  w.on('message', (msg) => console.log(msg));
}
```

```cjs
const { Worker, parentPort } = require('node:worker_threads');

if (process.argv[2] === 'worker') {
  parentPort.postMessage('hello from worker');
} else {
  const w = new Worker(__filename, { argv: ['worker'] });
  w.on('message', (msg) => console.log(msg));
}
```

### Sharing memory

Worker threads can share buffer-backed memory through `SharedArrayBuffer` and synchronize through
the `Atomics` JavaScript built-ins. Other transfer types (e.g., `MessagePort`, typed arrays) are
moved (zero-copy transfer) by listing them in the `transferList` argument to `postMessage`. The
full list is at [`../api/worker_threads.md`](../api/worker_threads.md) under "transferable values".

## Child processes

`node:child_process` spawns external OS processes. The four primary entry points:

| API | Use case |
| --- | --- |
| `spawn(cmd, args, options)` | Long-running process; stream stdin/stdout/stderr |
| `exec(cmdline, options, callback)` | Short-running shell command; buffer output |
| `execFile(file, args, options, callback)` | Same as exec but bypasses the shell |
| `fork(modulePath, args, options)` | Spawn a child Node.js process with an IPC channel |

`fork()` is the foundation of `node:cluster`. Public surface at
[`../api/child_process.md`](../api/child_process.md). Internal implementation at
`lib/internal/child_process/` and `lib/internal/child_process.js`.

```mjs
import { spawn } from 'node:child_process';

const ls = spawn('ls', ['-la', '/']);
ls.stdout.on('data', (chunk) => { /* ... */ });
ls.on('exit', (code) => console.log('exit', code));
```

```cjs
const { spawn } = require('node:child_process');

const ls = spawn('ls', ['-la', '/']);
ls.stdout.on('data', (chunk) => { /* ... */ });
ls.on('exit', (code) => console.log('exit', code));
```

## Cluster

`node:cluster` builds on `child_process.fork()` to run multiple Node.js processes that share TCP
listening sockets, distributing inbound connections among them. Internal implementation at
`lib/internal/cluster/`:

| File | Purpose |
| --- | --- |
| `lib/internal/cluster/primary.js` | Primary process orchestration |
| `lib/internal/cluster/worker.js` | Per-worker process logic |
| `lib/internal/cluster/child.js` | Child-side IPC handlers |
| `lib/internal/cluster/round_robin_handle.js` | Round-robin scheduling on non-Windows |
| `lib/internal/cluster/shared_handle.js` | OS-default scheduling (used on Windows) |
| `lib/internal/cluster/utils.js` | Common helpers |

The default scheduling policy is **round-robin** on non-Windows platforms (the primary process
accepts connections and dispatches them to workers in turn). On Windows, the default is
**shared-handle** (the OS distributes connections across worker processes). Both policies are
selectable at runtime through `cluster.schedulingPolicy`.

```mjs
import cluster from 'node:cluster';
import { availableParallelism } from 'node:os';
import { createServer } from 'node:http';

if (cluster.isPrimary) {
  for (let i = 0; i < availableParallelism(); i++) cluster.fork();
} else {
  createServer((req, res) => res.end('worker ' + process.pid)).listen(8000);
}
```

```cjs
const cluster = require('node:cluster');
const { availableParallelism } = require('node:os');
const { createServer } = require('node:http');

if (cluster.isPrimary) {
  for (let i = 0; i < availableParallelism(); i++) cluster.fork();
} else {
  createServer((req, res) => res.end('worker ' + process.pid)).listen(8000);
}
```

## Permission Model interaction

When `--permission` is enabled, both child processes and worker threads are denied by default and
must be re-granted with explicit flags:

| Flag | Effect |
| --- | --- |
| `--allow-child-process` | Permits `node:child_process` (`spawn`, `exec`, `execFile`, `fork`) |
| `--allow-worker` | Permits `new Worker(...)` |

Examples:

```bash
# Permission off: all child-process/worker creation allowed
node app.js

# Permission on: deny by default; re-grant only worker threads
node --permission --allow-worker app.js

# Permission on: re-grant both worker threads and child processes
node --permission --allow-worker --allow-child-process app.js
```

A known constraint of the permission model is that it does **not** propagate into worker threads —
each worker must be configured with its own permission boundaries. The full set of caveats is
documented in [Permission model](permission-model.md).

## Choosing the right model

| Workload | Recommended model |
| --- | --- |
| CPU-bound JavaScript work that can be partitioned | Worker threads (cheaper than processes; share heap-free memory via `SharedArrayBuffer`) |
| External CLI tooling, polyglot work, language interop | Child processes (`spawn`, `exec`, `execFile`) |
| Multi-core HTTP server scale-out | Cluster (round-robin) |
| Sandboxing untrusted user code | Cluster + Permission Model (worker threads share the parent's address space and memory) |

## Cross-references

* Architectural overview: [Architecture overview](overview.md)
* Feature catalog: [Feature catalog](features.md)
* Integration narrative: [Integration](integration.md)
* JavaScript runtime (V8 isolates, event loop): [JavaScript runtime](runtime.md)
* Permission Model (`--allow-child-process`, `--allow-worker`): [Permission model](permission-model.md)
* Public APIs:
  * [`../api/worker_threads.md`](../api/worker_threads.md)
  * [`../api/child_process.md`](../api/child_process.md)
  * [`../api/cluster.md`](../api/cluster.md)
  * [`../api/permissions.md`](../api/permissions.md)
  * [`../api/cli.md`](../api/cli.md)
  * [`../api/os.md`](../api/os.md) (for `availableParallelism`)
