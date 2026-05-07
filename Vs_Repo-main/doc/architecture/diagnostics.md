# Diagnostics

> Feature: F-007 — Diagnostics (`node:inspector`, `node:perf_hooks`, `node:diagnostics_channel`,
> `node:trace_events`, `node:async_hooks`)

This page deep-dives into the diagnostics surface of Node.js v26.0.0-pre. It covers the Chrome
DevTools Inspector protocol, perf hooks (high-resolution timing), the diagnostics channel pub/sub
event bus, trace events, and async hooks. The version stamps are `NODE_MAJOR_VERSION 26`,
`NODE_MODULE_VERSION 144`, `NODE_API_SUPPORTED_VERSION_MAX 10`, `NODE_VERSION_IS_RELEASE 0`, as
recorded in `src/node_version.h`.

For an overview of the runtime layers, see [Architecture overview](overview.md). For native
bindings (`src/inspector/`, `src/tracing/`) and bundled dependencies, see
[Dependencies](dependencies.md).

## Five diagnostic surfaces

The runtime exposes five distinct diagnostic surfaces. For each surface, the table below summarizes
its scope; full source-path details follow in a per-surface entry.

| Surface             | Public module                                               | Native bindings             |
| ------------------- | ----------------------------------------------------------- | --------------------------- |
| Inspector           | [`node:inspector`](../api/inspector.md)                     | `src/inspector/` (49 files) |
| Performance hooks   | [`node:perf_hooks`](../api/perf_hooks.md)                   | (V8 + native)               |
| Diagnostics channel | [`node:diagnostics_channel`](../api/diagnostics_channel.md) | (none)                      |
| Trace events        | [`node:trace_events`](../api/tracing.md)                    | `src/tracing/` (11 files)   |
| Async hooks         | [`node:async_hooks`](../api/async_hooks.md)                 | (V8 + native)               |

**Inspector** — `lib/inspector.js` composes V8's `v8::Inspector` with the native binding under
`src/inspector/` (49 files). Provides step-debugging, heap and CPU profiling, and console
redirection via the Chrome DevTools Protocol.

**Performance hooks** — `lib/perf_hooks.js` plus `lib/internal/perf/`, with V8 and native support
for high-resolution clocks. Exposes marks and measures, the observer pattern, and event-loop
utilization metrics.

**Diagnostics channel** — `lib/diagnostics_channel.js` is a pure-JavaScript pub/sub event bus for
cross-cutting events emitted by core modules and user code.

**Trace events** — `lib/trace_events.js` plus `lib/internal/trace_events_async_hooks.js`, backed by
the native binding under `src/tracing/` (11 files). Writes Chrome trace event format compatible
with `chrome://tracing` and Perfetto.

**Async hooks** — `lib/async_hooks.js` plus `lib/internal/async_hooks.js` and
`lib/internal/promise_hooks.js`, with V8 and native support. Exposes lifecycle hooks for async
resources (`init`, `before`, `after`, `destroy`, `promiseResolve`).

## Inspector protocol

The Inspector implements the Chrome DevTools Protocol (CDP) and is exposed when the runtime is
launched with `--inspect`, `--inspect-brk`, or `--inspect-wait`. The native side (`src/inspector/`,
49 files) bridges between V8's `v8::Inspector` and a WebSocket endpoint (default
`127.0.0.1:9229`) that DevTools or any CDP-compatible client can connect to.

```bash
# Listen for inspector connections; do not break before user code runs
node --inspect script.js

# Listen and break before the first line of user code
node --inspect-brk script.js

# Bind on all interfaces (CAUTION: do not expose to untrusted networks)
node --inspect=0.0.0.0:9229 script.js
```

The CDP supports debugging (breakpoints, step, evaluate, scope inspection), CPU profiling,
heap profiling and snapshots, console message redirection, and runtime metrics. The full surface is
at [`../api/inspector.md`](../api/inspector.md). For diagnostic-tooling support tiers, see
[`../contributing/diagnostic-tooling-support-tiers.md`](../contributing/diagnostic-tooling-support-tiers.md).

## Performance hooks

`node:perf_hooks` exposes the W3C User Timing Level 3 API (`performance.mark()`,
`performance.measure()`, `PerformanceObserver`) plus Node.js-specific extensions:

| API                                     | Purpose                                                  |
| --------------------------------------- | -------------------------------------------------------- |
| `performance.mark(name)`                | Record a timestamp                                       |
| `performance.measure(name, start, end)` | Compute elapsed time between two marks                   |
| `PerformanceObserver`                   | Asynchronous notification of performance entries         |
| `performance.eventLoopUtilization()`    | Fraction of wall time spent in the event loop            |
| `performance.timerify(fn)`              | Wrap a function so its calls produce performance entries |
| `monitorEventLoopDelay()`               | High-resolution histogram of event-loop delays           |
| `performance.nodeTiming`                | Phase-by-phase startup timing                            |

Implementation lives at `lib/internal/perf/`. Public surface at
[`../api/perf_hooks.md`](../api/perf_hooks.md).

## Diagnostics channel

`node:diagnostics_channel` is a low-overhead pub/sub bus for emitting and subscribing to
cross-cutting events. Channels are identified by string names; subscribers attach with
`channel.subscribe(callback)`. Core modules publish events on well-known channels (e.g.,
`http.client.request.start`, `http.server.request.start`, `dns.lookup`, etc.); user code can also
publish on its own channels.

```mjs
import dc from 'node:diagnostics_channel';

const channel = dc.channel('my.app.event');
channel.subscribe((message, name) => {
  // observe
});

channel.publish({ user: 'alice' });
```

```cjs
const dc = require('node:diagnostics_channel');

const channel = dc.channel('my.app.event');
channel.subscribe((message, name) => {
  // observe
});

channel.publish({ user: 'alice' });
```

When no subscriber is attached, `publish()` is essentially a no-op, so production deployments incur
near-zero overhead. The full API surface, including `TracingChannel` for the begin/end/asyncEnd/
error pattern, is at [`../api/diagnostics_channel.md`](../api/diagnostics_channel.md).

## Trace events

`node:trace_events` writes JSON-formatted trace events to a file (or stdio) compatible with the
Chrome trace event format and Perfetto. The native binding lives at `src/tracing/` (11 files), and
the JS-side glue at `lib/internal/trace_events_async_hooks.js`.

```bash
# Enable specific categories at startup
node --trace-event-categories='node,node.async_hooks,v8' --trace-event-file-pattern='trace.${pid}.json' script.js
```

The full surface is at [`../api/tracing.md`](../api/tracing.md). The list of available categories
matches the V8 trace categories plus Node.js-specific categories defined in `src/tracing/`.

## Async hooks

`node:async_hooks` exposes lifecycle hooks for asynchronous resources (timers, file I/O, sockets,
HTTP requests, etc.) and Promise resolution. Each resource is assigned a unique `asyncId`, and
hooks fire on five lifecycle events: `init`, `before`, `after`, `destroy`, and `promiseResolve`.

The implementation lives at `lib/async_hooks.js` (public) and `lib/internal/async_hooks.js` and
`lib/internal/promise_hooks.js` (internal). The public surface is at
[`../api/async_hooks.md`](../api/async_hooks.md). For high-level context propagation that uses async
hooks under the hood, see `node:async_context` ([`../api/async_context.md`](../api/async_context.md))
and the `AsyncLocalStorage` API.

## Reports

`node:report` (`process.report` and the `--report-on-fatalerror`, `--report-on-signal`,
`--report-uncaught-exception`, `--report-directory` flags) writes JSON-formatted runtime diagnostic
reports for the current process state on demand or on faults. The full surface is at
[`../api/report.md`](../api/report.md).

## Native subdirectories

| Subdirectory     | Files | Subject                                               |
| ---------------- | ----- | ----------------------------------------------------- |
| `src/inspector/` | 49    | Native inspector implementation (see below)           |
| `src/tracing/`   | 11    | Trace-event categories, event writer, config plumbing |

`src/inspector/` (49 files) houses the native inspector implementation, including the
DevTools-protocol agents, transport, and the per-isolate inspector binding. `src/tracing/` (11
files) houses the trace-event categories, the event writer, and the configuration plumbing that
feeds JSON trace output to disk or stdio.

## Cross-references

* Architectural overview: [Architecture overview](overview.md)
* Feature catalog: [Feature catalog](features.md)
* Integration narrative: [Integration](integration.md)
* Bundled dependencies: [Dependencies](dependencies.md)
* JavaScript runtime (V8 isolate, event loop): [JavaScript runtime](runtime.md)
* Concurrency (per-worker tracing): [Concurrency](concurrency.md)
* Public APIs:
  * [`../api/inspector.md`](../api/inspector.md)
  * [`../api/perf_hooks.md`](../api/perf_hooks.md)
  * [`../api/diagnostics_channel.md`](../api/diagnostics_channel.md)
  * [`../api/tracing.md`](../api/tracing.md)
  * [`../api/async_hooks.md`](../api/async_hooks.md)
  * [`../api/async_context.md`](../api/async_context.md)
  * [`../api/report.md`](../api/report.md)
  * [`../api/debugger.md`](../api/debugger.md)
* Diagnostic tooling support tiers:
  [`../contributing/diagnostic-tooling-support-tiers.md`](../contributing/diagnostic-tooling-support-tiers.md)
* Native memory leak investigation:
  [`../contributing/investigating-native-memory-leaks.md`](../contributing/investigating-native-memory-leaks.md)
* Post-mortem support: [`../contributing/node-postmortem-support.md`](../contributing/node-postmortem-support.md)
