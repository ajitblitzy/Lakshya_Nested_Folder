# TypeScript

> Feature: F-009 — TypeScript type stripping via Amaro
> (`--experimental-strip-types` is the default in v26; `--experimental-transform-types` was removed
> in v26)

This page deep-dives into Node.js v26.0.0-pre's built-in TypeScript support. The runtime ships an
in-process type stripper based on the SWC compiler compiled to WASM, packaged as the
[Amaro](https://github.com/nodejs/amaro) project and bundled in the upstream Node.js tree under
`deps/amaro/`. (In this repository's source archive the `deps/` tree is omitted; the bundled copy is
identified through `src/amaro_version.h` and the update automation in
`tools/dep_updaters/update-amaro.sh`.) The version stamps are `NODE_MAJOR_VERSION 26`,
`NODE_MODULE_VERSION 144`, `NODE_API_SUPPORTED_VERSION_MAX 10`, `NODE_VERSION_IS_RELEASE 0`, as
recorded in `src/node_version.h`.

For an overview of the runtime layers, see [Architecture overview](overview.md). For the module
loader that triggers TypeScript handling, see [JavaScript runtime](runtime.md). For dependency
details (Amaro), see [Dependencies](dependencies.md).

## What the runtime does (and does not do)

The runtime's built-in TypeScript support is intentionally lightweight: it **erases types** at load
time so that TypeScript files can be executed without an explicit compile step. It does **not**
perform full TypeScript compilation, type checking, or down-leveling of newer ECMAScript syntax.

The capabilities of the built-in TypeScript pipeline (versus the external tooling user code must
fall back to) are summarized below:

* **Type erasure (TS → JS by removing type annotations).** Built-in via Amaro. No external tooling
  required.
* **Type checking.** Not built in. Use `tsc --noEmit`, `tsd`, `vitest --typecheck`, or an editor
  with a TypeScript language service.
* **Down-leveling** (for example, ES2024 → ES2017). Not built in. Use `tsc`, `swc`, `esbuild`, or
  `babel` ahead of time.
* **`enum`, `namespace`, and parameter properties.** Not built in. (The
  `--experimental-transform-types` mode that previously supported these forms was removed in v26;
  see the section below.) Use `tsc` or an `swc`/`esbuild` build step ahead of time.
* **Decorators.** Partial — legacy decorators are accepted by Amaro for type-stripping purposes.
  For full decorator semantics, use `tsc` or an `swc`/`esbuild` build step ahead of time.
* **Path mapping (`tsconfig.json` `paths`).** Not built in. Install resolver hooks via the
  `node:module` `register()` API to translate specifier paths.

## How it works

The Node.js module loader recognizes `.ts`, `.mts`, and `.cts` file extensions and routes them
through `lib/internal/modules/typescript.js`. That module invokes Amaro's WASM-bundled SWC core to
strip type annotations from the source, returning a stripped JavaScript string plus a source map.
The stripped JavaScript is then handed to V8 for compilation as if it were a `.js`, `.mjs`, or
`.cjs` file (per the file's `package.json` `"type"` field).

The bundled Amaro lives at `deps/amaro/` in the upstream Node.js tree and is enabled by the build
toggle `node_use_amaro: true` declared in `node.gyp`. (In this repository's source archive the
`deps/` tree is omitted; the integration is identifiable through `src/amaro_version.h` and
`tools/dep_updaters/update-amaro.sh`.) Dependency details and update automation are in
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
