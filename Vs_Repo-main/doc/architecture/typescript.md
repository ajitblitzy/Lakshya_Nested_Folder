# TypeScript

> Feature: F-009 — TypeScript (`--experimental-strip-types`, `--experimental-transform-types`)

This page deep-dives into Node.js v26.0.0-pre's built-in TypeScript support. The runtime ships an
in-process type stripper based on the SWC compiler compiled to WASM, packaged as the
[Amaro](https://github.com/nodejs/amaro) project and bundled at `deps/amaro/`. The version stamps
are `NODE_MAJOR_VERSION 26`, `NODE_MODULE_VERSION 144`.

For an overview of the runtime layers, see [Architecture overview](overview.md). For the module
loader that triggers TypeScript handling, see [JavaScript runtime](runtime.md). For dependency
details (Amaro), see [Dependencies](dependencies.md).

## What the runtime does (and does not do)

The runtime's built-in TypeScript support is intentionally lightweight: it **erases types** at load
time so that TypeScript files can be executed without an explicit compile step. It does **not**
perform full TypeScript compilation, type checking, or down-leveling of newer ECMAScript syntax.

| Capability                                          | Built-in support                   | Use external tooling for                          |
| --------------------------------------------------- | ---------------------------------- | ------------------------------------------------- |
| Type erasure (TS → JS by removing type annotations) | ✓ via Amaro                        | n/a                                               |
| Type checking                                       | ✗                                  | `tsc --noEmit`, `tsd`, `vitest --typecheck`, etc. |
| Down-leveling (e.g., ES2024 → ES2017)               | ✗                                  | `tsc`, `swc`, `esbuild`, `babel`                  |
| `enum`, `namespace`, parameter properties           | ✗ (transform-types removed in v26) | tsc or swc/esbuild ahead of time                  |
| Decorators                                          | partial (legacy decorators)        | tsc or swc/esbuild ahead of time                  |
| Path mapping (`tsconfig.json` `paths`)              | ✗                                  | resolver hooks via `node:module` `register()`     |

## How it works

The Node.js module loader recognizes `.ts`, `.mts`, and `.cts` file extensions and routes them
through `lib/internal/modules/typescript.js`. That module invokes Amaro's WASM-bundled SWC core to
strip type annotations from the source, returning a stripped JavaScript string plus a source map.
The stripped JavaScript is then handed to V8 for compilation as if it were a `.js`, `.mjs`, or
`.cjs` file (per the file's `package.json` `"type"` field).

The bundled Amaro lives at `deps/amaro/` and is enabled by the build toggle
`node_use_amaro: true` declared in `node.gyp`. Dependency details and update automation are in
[Dependencies](dependencies.md).

## Default mode and `--experimental-strip-types`

In Node.js v23 and later, type stripping is the **default behavior** for `.ts`, `.mts`, and `.cts`
files. Earlier versions required the explicit `--experimental-strip-types` flag.

```bash
# Default in v26: type stripping just works
node app.ts

# Explicit flag (no-op in v26 because the feature is now on by default)
node --experimental-strip-types app.ts
```

The runtime's TypeScript reference at [`../api/typescript.md`](../api/typescript.md) documents the
exact stability status, configuration, and history of the feature in this release.

## `--experimental-transform-types`

A second mode, `--experimental-transform-types`, was previously available to additionally support
`enum`, `namespace`, and other transformations beyond pure type erasure. As recorded in the change
history at [`../api/typescript.md`](../api/typescript.md), this flag has been removed in v26 (the
default and only supported mode is type erasure).

If your code requires `enum`, `namespace`, or parameter-property syntax, the recommended path is to
compile ahead of time with `tsc`, `swc`, or `esbuild` and ship the `.js` output to Node.js.

## Source maps

Amaro emits source maps so that runtime errors and stack traces refer to the original `.ts` lines
and columns rather than the stripped `.js` shape. The runtime's source-map handling is controlled
by:

* `--enable-source-maps` (CLI flag)
* `process.setSourceMapsEnabled(true)`
* `NODE_OPTIONS=--enable-source-maps`

The full source-map subsystem is documented in [`../api/cli.md`](../api/cli.md) under
`--enable-source-maps`.

## Module-loading flow with TypeScript

The full module-loading sequence (with TypeScript handling) is illustrated in
[JavaScript runtime](runtime.md). Briefly:

1. Specifier resolution (relative, absolute, or bare) selects a file path.
2. The file extension and the closest `package.json` `"type"` field determine the module kind
   (CJS, ESM, or TypeScript).
3. For `.ts`, `.mts`, `.cts`, the loader calls `lib/internal/modules/typescript.js`, which invokes
   Amaro to strip types.
4. The stripped JavaScript is compiled by V8 (Ignition / SparkPlug / Maglev / TurboFan, see
   [JavaScript runtime](runtime.md)).
5. The module's exports are cached and returned.

## Module customization

For more sophisticated TypeScript handling (e.g., full compilation, type checking, custom resolver
behavior), the [`node:module`](../api/module.md) `register()` API allows user code to install
custom loader hooks. Examples include `tsx`, `ts-node`, and `vite-node`. The customization-hook
mechanism is implemented at `lib/internal/modules/customization_hooks.js`.

## Cross-references

* Architectural overview: [Architecture overview](overview.md)
* Feature catalog: [Feature catalog](features.md)
* Integration narrative: [Integration](integration.md)
* JavaScript runtime (module loader): [JavaScript runtime](runtime.md)
* Bundled dependencies (Amaro): [Dependencies](dependencies.md)
* Public APIs:
  * [`../api/typescript.md`](../api/typescript.md)
  * [`../api/module.md`](../api/module.md)
  * [`../api/modules.md`](../api/modules.md)
  * [`../api/esm.md`](../api/esm.md)
  * [`../api/packages.md`](../api/packages.md)
  * [`../api/cli.md`](../api/cli.md)
* Build and platform requirements: [`../../BUILDING.md`](../../BUILDING.md)
