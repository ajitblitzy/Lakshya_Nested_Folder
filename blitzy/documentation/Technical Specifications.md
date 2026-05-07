# Technical Specification

# 0. Agent Action Plan

## 0.1 Intent Clarification

### 0.1.1 Core Documentation Objective

Based on the provided requirements, the Blitzy platform understands that the documentation objective is to **read and analyze the Node.js v26.0.0-pre source code contained within `Vs_Repo-main/` (extracted from `Vs_Repo-main.zip`) and produce structured documentation that explains the runtime's features and integration expectations, while clearly differentiating between development activities (building, coding, contributing, releasing) and testing activities (test execution, coverage, CI gates, quality assurance)**.

The user's verbatim instructions read:

> "Read and analyze the code and document the code to explain the features, integration the expectation for the code. Ensure to differentiate between developement and testing activities."

A user-specified implementation rule named "Document code" with content "Test" is also attached and is captured verbatim in sub-section 0.9 (Rules for Documentation).

**Request Categorization:**

| Dimension | Classification |
|---|---|
| Primary action | **Update existing documentation** — augment the Node.js doc tree with structured feature/integration/development/testing guides |
| Secondary action | **Create new consolidated reference docs** — a top-level features index, an integration expectations guide, and a development-vs-testing companion |
| Documentation types in scope | Architecture overview, feature reference, integration guide, development guide, testing guide, README updates |
| Coverage stance | Conservative — every documented claim must trace to a specific file or directory in `Vs_Repo-main/` |

**Documentation Type Coverage:**

| Documentation Type | Scope | Target Output |
|---|---|---|
| **Architecture / Feature documentation** | The 13 runtime features (F-001 through F-013) catalogued in §2.1 — JavaScript Runtime, Networking, File System, Cryptography, Concurrency, Streams, Diagnostics, Test Runner, TypeScript, SQLite, SEA, Permission Model, Web Standards | New Markdown deliverables under `doc/architecture/` |
| **Integration expectations** | How public `lib/` modules bind to internal `lib/internal/` and native `src/` layers; how bundled dependencies (V8, libuv, OpenSSL, nghttp2, c-ares, undici, ICU, SQLite) integrate; how N-API addons integrate; how external CI systems (GitHub Actions, Codecov, CodeQL, OSSF Scorecard, HackerOne) integrate | New Markdown deliverables under `doc/architecture/integration.md` and updates to `doc/api/n-api.md`, `doc/api/embedding.md` |
| **Development activities** | Building from source (`make`, `vcbuild.bat`), configuring (`configure.py`), debugging, contributing PRs, code style, dependency updates, releases, onboarding | New consolidated guide `doc/contributing/development-overview.md` cross-linking the existing 45 docs in `doc/contributing/` |
| **Testing activities** | Running the test suite (`make test`, `make test-only`, `tools/test.py`), coverage (`nyc`, Codecov), benchmarks (`benchmark/`), linting (`make lint`), the built-in test runner (`node --test`), WPT conformance, security scanning (CodeQL, OSSF Scorecard) | New consolidated guide `doc/contributing/testing-overview.md` and updates to `doc/contributing/writing-tests.md` |
| **README and entry points** | The bare repository root `README.md` (currently a single-line placeholder) and `Vs_Repo-main/README.md` (the upstream Node.js README) | UPDATE `README.md` (root) to point at the documentation deliverables; UPDATE `doc/README.md` style guide cross-references where impacted |

**Implicit Documentation Needs Inferred from the Request:**

- The user's verb "analyze" implies mechanical extraction of feature-to-file mappings, public API surfaces, and dependency edges — this work has already been completed by upstream tech-spec sections (§1, §2, §3, §5, §6) and must be **referenced**, not redone.
- The phrase "integration the expectation" (interpreted as "integration expectations") implies documenting both **inbound integration** (how user code integrates with Node.js via `lib/` and N-API) and **outbound integration** (how Node.js integrates with V8, libuv, OpenSSL, nghttp2, ICU, etc., as listed in §1.2.1).
- "Differentiate between development and testing activities" implies producing two parallel guides whose scopes do not overlap — development covers the "how do I change the runtime" workflows, testing covers the "how do I validate the runtime" workflows. The `Makefile` already exposes the canonical split (`doc-only`, `doc`, `test`, `test-only`, `test-doc`, `lint-md`).
- Because the runtime contains pre-existing comprehensive documentation (68 files in `doc/api/`, 45 files in `doc/contributing/`), the documentation strategy must follow the existing style guide at `doc/README.md` and the existing `@node-core/doc-kit` build pipeline.

### 0.1.2 Special Instructions and Constraints

**Implementation Rules Captured Verbatim:**

The user supplied a single implementation rule:

```
[{"name": "Document code", "content": "Test"}]
```

The rule's title ("Document code") is treated as the canonical guiding directive: every artifact produced MUST document existing code without modifying it. The rule's content ("Test") is interpreted in conjunction with the user's instruction to "differentiate between development and testing activities", meaning the testing dimension MUST receive first-class documentation treatment alongside development.

**Style Constraints Inherited from the Repository:**

- **MUST** follow the Node.js documentation style guide at `Vs_Repo-main/doc/README.md`: file naming uses `lowercase-with-dashes.md`, line wrapping at 120 characters, US English spelling, Markdown rendered through `@node-core/doc-kit`.
- **MUST** preserve existing API documentation conventions: stability index (0/1/1.0/1.1/1.2/2/3 levels per `doc/api/documentation.md`), `<!-- YAML -->` metadata blocks, `<!-- source_link=... -->` annotations, parallel `mjs` and `cjs` code samples.
- **MUST** validate output via `make lint-md` and `make test-doc -j` (the canonical doc validation invocations from `doc/README.md` and `doc/contributing/writing-docs.md`).
- **MUST** preserve all existing documentation files; the task is additive plus targeted updates, not replacement.

**Constraints Inherited from the Provided Setup:**

- No setup instructions were provided; no environment variables or secrets were attached; no user files were uploaded.
- No design system is referenced — the **Design System Compliance** sub-section is therefore intentionally omitted (per protocol: design-system sub-section is conditional on a system being specified).
- No Figma attachments are referenced — no design-to-system mapping is required.

**Web Search Requirements:**

No additional web research is required. The repository ships with a complete documentation toolchain (`tools/doc/`), an authoritative style guide (`doc/README.md`), and explicit build/test/lint commands in the `Makefile`. All best practices needed for this task are already encoded in the codebase.

### 0.1.3 Technical Interpretation

These documentation requirements translate to the following technical documentation strategy:

- **To document features**, the Blitzy platform will create a new top-level architecture index `doc/architecture/README.md` and one feature-overview file per feature tier under `doc/architecture/`, cross-linking from each feature page to the corresponding existing API reference in `doc/api/`. Source code citations will reference the exact files in `lib/`, `lib/internal/`, and `src/` listed in §2.1.
- **To document integration expectations**, the Blitzy platform will create `doc/architecture/integration.md` capturing the layered architecture (User Space → `lib/` → `lib/internal/` → `src/` → bundled `deps/`), the bundled-dependency contract (V8 embedder string `-node.17`, OpenSSL, libuv, nghttp2, c-ares, undici, ICU, SQLite, Amaro), the N-API contract (`NODE_API_SUPPORTED_VERSION_MAX 10`), and the external development-infrastructure integrations (GitHub Actions, Codecov, CodeQL, OSSF Scorecard, HackerOne).
- **To differentiate development from testing activities**, the Blitzy platform will create `doc/contributing/development-overview.md` (covering build, configure, code-style, debug, dependency updates, releases) and `doc/contributing/testing-overview.md` (covering test execution, coverage, linting, benchmarks, CI workflows, security scans) with a clearly delineated scope at the top of each file, cross-referencing the 45 existing files in `doc/contributing/` rather than duplicating their content.
- **To make the documentation discoverable**, the Blitzy platform will UPDATE the root `README.md` (currently a 60-byte placeholder) to enumerate the new guides, and UPDATE `doc/api/index.md` to insert architecture and contributing entry points at the top of the page.

### 0.1.4 Inferred Documentation Needs

| Inference Source | Inferred Documentation Need | Resolution |
|---|---|---|
| The 13 features (F-001 — F-013) span 54 public `lib/` modules and 453 `src/` files but lack a single consolidated feature catalog visible to end-users | Consolidated feature index | CREATE `doc/architecture/features.md` |
| `lib/` public modules delegate to `lib/internal/` which binds to `src/` via N-API — integration semantics not surfaced in any single doc | Layered integration narrative with diagrams | CREATE `doc/architecture/integration.md` |
| Existing `doc/contributing/` has 45 disparate files with no top-level index distinguishing dev-only from test-only concerns | Development overview + testing overview | CREATE `doc/contributing/development-overview.md` and `doc/contributing/testing-overview.md` |
| Built-in test runner (F-008), benchmarks (591 files in `benchmark/`), and the polyglot lint stack (ESLint, clang-format, cpplint, ruff, yamllint) are documented in fragments | Single testing reference unifying all validation surfaces | CREATE `doc/contributing/testing-overview.md` |
| Bundled dependencies (V8, libuv, OpenSSL, nghttp2, c-ares, undici, ICU, SQLite, Amaro) and their patches/versions are scattered across `common.gypi`, `node.gyp`, `configure.py` | Single dependency-integration matrix | CREATE `doc/architecture/dependencies.md` |
| Repository root `README.md` is a 60-byte placeholder ("Ajit-backprop-test / test project for backprop integration") that does not orient new readers | Top-level orientation pointing into the documentation tree | UPDATE root `README.md` |
| Native addon developers (N-API, embedding API) currently navigate `doc/api/n-api.md`, `doc/api/embedding.md`, `doc/api/addons.md` independently | Cross-linked integration entry point for native addons | UPDATE `doc/api/n-api.md` and `doc/api/embedding.md` with cross-references to the new integration guide |


## 0.2 Documentation Discovery and Analysis

### 0.2.1 Existing Documentation Infrastructure Assessment

Repository analysis reveals a **mature, well-organized documentation tree** with 164 Markdown documents across `doc/`, an external doc-generation toolchain (`@node-core/doc-kit`), and Make/batch targets that build, lint, and serve the docs. The repository root, however, contains only a 60-byte placeholder `README.md` whose content is `# Ajit-backprop-test\ntest project for backprop integration.\n` — the actual Node.js codebase under analysis lives inside `Vs_Repo-main/` (extracted from `Vs_Repo-main.zip`).

**Documentation Framework Inventory:**

| Attribute | Detail | Source |
|---|---|---|
| Documentation generator | `@node-core/doc-kit` | `tools/doc/package.json` |
| Generator version | `1.0.2` | `tools/doc/package.json` |
| Generator entrypoint | `tools/doc/node_modules/@node-core/doc-kit/bin/cli.mjs` | `Makefile` (`DOC_KIT ?= ...`) |
| Build target (full) | `make doc` | `Makefile` (`.PHONY: doc`) |
| Build target (no recompile) | `make doc-only` | `Makefile` (`.PHONY: doc-only`) |
| Local server | `make docserve` (binds 127.0.0.1:8000 from `out/doc/api`) | `Makefile` (`.PHONY: docserve`) |
| Local browser open | `make docopen` | `Makefile` (`.PHONY: docopen`) |
| Clean target | `make docclean` | `Makefile` |
| Markdown linter | `make lint-md` (powered by ESLint + `@eslint/markdown`) | `Makefile`, `eslint.config.mjs` |
| Doc test runner | `make test-doc` / `make test-doc-ci` | `Makefile` (`.PHONY: test-doc`) |
| Output formats | HTML (`out/doc/api/*.html`), JSON (`out/doc/api/*.json`), aggregate `all.html` / `all.json`, `apilinks.json` | `Makefile` (`doc-only` target dependencies) |
| Diagram generator | Inline Mermaid blocks rendered by doc-kit | `doc/contributing/api-documentation.md` |
| Style guide | `doc/README.md` (Node.js documentation style guide) | Inspected — 120-char wrap, US spelling, `lowercase-with-dashes.md` filenames |
| Stability index policy | 0–3 stability levels with sub-stages 1.0/1.1/1.2 | `doc/api/documentation.md` |
| YAML metadata convention | HTML-comment YAML blocks with `added:` and `changes:` keys | `doc/api/test.md` and others |
| Source-link convention | HTML-comment `source_link=lib/<module>.js` annotation | `doc/api/test.md` and others |

**Documentation Hosting and Deployment:**

The artifacts produced by `make doc` (under `out/doc/api/*.html` and `*.json`) are uploaded to nodejs.org via the `doc-upload` Makefile target during release builds, as documented in `doc/contributing/api-documentation.md`. The promotion path moves docs to `/home/dist/nodejs/docs/<full_version>` for serving from `nodejs.org/api/`.

**API Documentation Tools In Use:**

- `@node-core/doc-kit` v1.0.2 — Node.js's in-house Markdown to HTML/JSON pipeline (formerly `tools/doc`)
- ESLint with `@eslint/markdown` plugin — Markdown linting integrated into `make lint`
- `@node-core/utils` (via onboarding doc) — collaborator tooling, not part of doc generation per se
- `tools/doc/deprecationCodes.mjs` — programmatic extraction of deprecation codes from `doc/api/deprecations.md`
- `tools/doc/generate-json-schema.mjs` — JSON schema generation
- `doc/type-map.json` — type mapping consumed by doc-kit
- `doc/node-config-schema.json` — config schema referenced by doc generation

**Diagram Tools Detected:**

- **Mermaid** — Used extensively in upstream tech-spec sections; doc-kit supports inline mermaid fenced blocks
- No PlantUML, Graphviz, or external diagram tooling present in `tools/`

### 0.2.2 Existing Documentation Tree Inventory

The current documentation surface is comprehensive but does not include consolidated feature, integration, or development-vs-testing overviews.

**Top-Level Documentation Files (root):**

| File | Purpose | Status |
|---|---|---|
| `README.md` (repo root) | Single-line placeholder for the test repository | Will be UPDATED |
| `Vs_Repo-main/README.md` | Upstream Node.js project README | Existing — referenced |
| `Vs_Repo-main/BUILDING.md` | Build instructions for all platforms | Existing — referenced |
| `Vs_Repo-main/CONTRIBUTING.md` | Contribution overview and DCO | Existing — referenced |
| `Vs_Repo-main/CHANGELOG.md` | Version history | Existing — referenced |
| `Vs_Repo-main/CODE_OF_CONDUCT.md` | Community standards | Existing — referenced |
| `Vs_Repo-main/GOVERNANCE.md` | OpenJS / TSC governance model | Existing — referenced |
| `Vs_Repo-main/SECURITY.md` | Security disclosure policy and HackerOne | Existing — referenced |
| `Vs_Repo-main/LICENSE` | MIT license | Existing — referenced |
| `Vs_Repo-main/glossary.md` | Project glossary | Existing — referenced |
| `Vs_Repo-main/onboarding.md` | Collaborator onboarding | Existing — referenced |

**API Reference Tree (`Vs_Repo-main/doc/api/` — 68 files):**

| File | Module Documented |
|---|---|
| `index.md` | Top-level API index (will be UPDATED) |
| `documentation.md` | Stability index and reading guide |
| `synopsis.md` | Usage examples |
| `addons.md`, `n-api.md`, `embedding.md` | Native addon and embedding APIs |
| `assert.md`, `test.md` | Built-in testing |
| `async_context.md`, `async_hooks.md`, `diagnostics_channel.md`, `inspector.md`, `perf_hooks.md`, `report.md`, `tracing.md` | Diagnostics & observability (F-007) |
| `buffer.md`, `stream.md`, `stream_iter.md`, `webstreams.md`, `string_decoder.md` | Streams & buffers (F-006) |
| `child_process.md`, `cluster.md`, `worker_threads.md` | Concurrency (F-005) |
| `crypto.md`, `tls.md`, `webcrypto.md` | Cryptography (F-004) |
| `dgram.md`, `dns.md`, `http.md`, `http2.md`, `https.md`, `net.md`, `quic.md` | Networking (F-002) |
| `fs.md` | File system (F-003) |
| `cli.md`, `console.md`, `debugger.md`, `errors.md`, `events.md`, `globals.md`, `process.md`, `repl.md`, `timers.md`, `tty.md`, `util.md`, `vm.md`, `os.md`, `path.md`, `url.md`, `querystring.md`, `punycode.md`, `readline.md`, `domain.md`, `environment_variables.md` | Core runtime, utilities, environment |
| `module.md`, `modules.md`, `esm.md`, `packages.md`, `typescript.md` | Module system (F-001, F-009) |
| `permissions.md` | Permission model (F-012) |
| `single-executable-applications.md` | SEA (F-011) |
| `sqlite.md` | SQLite (F-010) |
| `intl.md`, `wasi.md`, `v8.md`, `zlib.md`, `zlib_iter.md` | Specialized APIs |
| `deprecations.md` | Deprecated APIs |

**Contributing Documentation (`Vs_Repo-main/doc/contributing/` — 45 files):**

| File | Topic | Doc Class |
|---|---|---|
| `pull-requests.md` | PR workflow | Development |
| `collaborator-guide.md` | Collaborator responsibilities | Development |
| `commit-queue.md` | Commit queue mechanism | Development |
| `cpp-style-guide.md` | C++ style | Development |
| `primordials.md` | Internal primordials usage | Development |
| `building-node-with-ninja.md`, `gn-build.md` | Alternative builds | Development |
| `internal-api.md`, `components-in-core.md` | Internal architecture | Development |
| `static-analysis.md` | Static analysis tooling | Development |
| `adding-new-napi-api.md`, `adding-v8-fast-api.md` | Adding native APIs | Development |
| `releases.md`, `releases-node-api.md`, `backporting-to-release-lines.md`, `distribution.md` | Release engineering | Development |
| `using-devcontainer.md` | DevContainer usage | Development |
| `investigating-native-memory-leaks.md`, `node-postmortem-support.md`, `diagnostic-tooling-support-tiers.md` | Debugging | Development |
| `security-model-strategy.md`, `security-release-process.md`, `security-steward-on-off-boarding.md` | Security process | Development |
| `code-of-conduct.md`, `issues.md`, `feature-request-management.md`, `recognizing-contributors.md` | Community | Development |
| `offboarding.md`, `advocacy-ambassador-program.md`, `managing-social-media-accounts.md`, `sharing-project-news.md`, `streaming-to-youtube.md`, `suggesting-social-media-posts.md` | Project operations | Development |
| `technical-priorities.md`, `technical-values.md`, `strategic-initiatives.md`, `erm-guidelines.md` | Strategy & governance | Development |
| `api-documentation.md` | Doc-kit tooling internals | Development |
| `writing-docs.md` | How to write docs | Development |
| `writing-tests.md` | How to write tests | **Testing** |
| `maintaining/` (sub-folder) | Per-dependency maintenance | Development |

**Repository-Level Module READMEs:**

| File | Purpose |
|---|---|
| `Vs_Repo-main/src/README.md` | C++ codebase orientation (existing) |
| `Vs_Repo-main/lib/README.md` | (Empty — present but blank) |
| `Vs_Repo-main/test/README.md` | Test directory orientation (existing) |
| `Vs_Repo-main/doc/README.md` | Documentation style guide (existing) |
| `Vs_Repo-main/tools/doc/README.md` | Doc tooling pointer (existing) |

### 0.2.3 Repository Code Analysis for Documentation

**Search Strategy Employed:**

- Folder enumeration via `get_source_folder_contents` on the repository root
- ZIP extraction of `Vs_Repo-main.zip` to `/tmp/nodejs_extract/Vs_Repo-main/`
- File-system walks via `find` for `*.md`, `*.js`, `*.cc`, `*.py` to inventory documentation candidates
- Cross-reference against tech-spec §1, §2, §3, §5, §6 for already-extracted feature catalogs

**Search Patterns Used for Code-to-Document:**

| Pattern | Target | Result |
|---|---|---|
| `lib/*.js` | Public JavaScript API surface | 54 modules confirmed (`lib/http.js`, `lib/fs.js`, `lib/crypto.js`, `lib/test.js`, etc.) |
| `lib/internal/**/*.js` | Internal implementation modules | 80+ groups, 363+ files |
| `src/**/*.cc`, `src/**/*.h` | Native runtime layer | 453 files across 9 subdirectories |
| `src/<subdirectory>/` | Specialized native engines | `crypto/` (63), `inspector/` (50), `quic/` (34), `permission/` (18), `tracing/` (12), `api/` (9), `large_pages/` (4), `dataqueue/` (3), `res/` (4) |
| `doc/api/*.md` | Existing API docs | 68 files (one per public module + indices) |
| `doc/contributing/*.md` | Existing contributor docs | 45 files |
| `test/**/*.js` | Existing test corpus | 11,060 files across 37 categories |
| `benchmark/**/*.js` | Existing benchmark corpus | 591 files |
| `.github/workflows/*.yml` | CI workflow inventory | 37 workflows |
| `Makefile` | Build/doc/test targets | Verified `doc`, `doc-only`, `docserve`, `docopen`, `docclean`, `test`, `test-only`, `test-doc`, `lint`, `lint-md`, `lint-js-fix` targets |

**Key Directories Examined:**

- `Vs_Repo-main/` (root)
- `Vs_Repo-main/doc/` and all subdirectories
- `Vs_Repo-main/lib/` and `lib/internal/`
- `Vs_Repo-main/src/`
- `Vs_Repo-main/test/`
- `Vs_Repo-main/tools/doc/`
- `Vs_Repo-main/.github/workflows/`

**Related Existing Documentation Found:**

The discovery confirms that **every public JavaScript module already has a corresponding API reference in `doc/api/`** and **every contribution workflow already has a guide in `doc/contributing/`**. The gap addressed by this documentation effort is therefore a **synthesis layer** — a small number of high-level documents that index, cross-link, and contextualize the existing detailed references rather than duplicating their content.

### 0.2.4 Web Search Research Conducted

No external web search is required for this task. All required documentation conventions, build commands, and tooling versions are explicitly defined inside the repository:

- The `@node-core/doc-kit` version (`1.0.2`) is pinned in `tools/doc/package.json`
- The Markdown style is fully specified in `doc/README.md`
- The doc lint command (`make lint-md`) is defined in the `Makefile`
- The doc test command (`make test-doc -j`) is defined in the `Makefile`
- The diagram convention (Mermaid in fenced blocks) is established in existing docs

External reference points cited within the existing repository (and therefore available without web search) include:

- Stability index policy: `doc/api/documentation.md`
- API documentation tooling: `doc/contributing/api-documentation.md` and the upstream `nodejs/doc-kit` GitHub repository
- Markdown writing guide: `doc/contributing/writing-docs.md`
- Test writing guide: `doc/contributing/writing-tests.md`


## 0.3 Documentation Scope Analysis

### 0.3.1 Code-to-Documentation Mapping

The user's instruction to "explain the features" and "integration the expectation" maps to a comprehensive feature-to-source mapping. The runtime's 13 features (cataloged in §2.1) connect to public modules, internal implementations, native bindings, bundled dependencies, and existing API references as follows.

**Feature-to-Source Mapping:**

| Feature | Public Modules | Internal Modules | Native Bindings | Bundled Deps | Existing API Doc | New/Updated Doc |
|---|---|---|---|---|---|---|
| F-001 JS Runtime | `lib/module.js` | `lib/internal/modules/` | `src/node_main.cc`, `src/api/` | V8, libuv | `doc/api/module.md`, `modules.md`, `esm.md`, `packages.md` | NEW `doc/architecture/runtime.md` |
| F-002 Networking | `lib/http.js`, `https.js`, `http2.js`, `net.js`, `dgram.js`, `dns.js`, `tls.js`, `quic.js` | `lib/internal/http*`, `lib/internal/dns*`, `lib/internal/tls/*`, `lib/internal/quic/*` | `src/quic/` (34 files) | OpenSSL, nghttp2, c-ares, undici, libuv | 8 separate API files | NEW `doc/architecture/networking.md` |
| F-003 File System | `lib/fs.js` | `lib/internal/fs/` | (libuv binding) | libuv | `doc/api/fs.md` | NEW `doc/architecture/file-system.md` |
| F-004 Cryptography | `lib/crypto.js` | `lib/internal/crypto/` | `src/crypto/` (63 files) | OpenSSL | `doc/api/crypto.md`, `tls.md`, `webcrypto.md` | NEW `doc/architecture/cryptography.md` |
| F-005 Concurrency | `lib/child_process.js`, `cluster.js`, `worker_threads.js` | `lib/internal/child_process*`, `lib/internal/cluster/*`, `lib/internal/worker/*` | (libuv binding) | libuv | `doc/api/child_process.md`, `cluster.md`, `worker_threads.md` | NEW `doc/architecture/concurrency.md` |
| F-006 Streams | `lib/stream.js` | `lib/internal/streams/`, `lib/internal/webstreams/` | (none) | (none) | `doc/api/stream.md`, `webstreams.md`, `stream_iter.md` | NEW `doc/architecture/streams.md` |
| F-007 Diagnostics | `lib/inspector.js`, `perf_hooks.js`, `diagnostics_channel.js`, `trace_events.js`, `async_hooks.js` | `lib/internal/inspector/`, `lib/internal/perf/`, `lib/internal/async_hooks*` | `src/inspector/` (50 files), `src/tracing/` (12 files) | V8 Inspector | 5 separate API files | NEW `doc/architecture/diagnostics.md` |
| F-008 Testing | `lib/test.js`, `lib/assert.js` | `lib/internal/test_runner/`, `lib/internal/assert/` | (none) | (none) | `doc/api/test.md`, `assert.md` | NEW `doc/contributing/testing-overview.md` |
| F-009 TypeScript | (via module loader) | `lib/internal/modules/*` | (Amaro WASM binding) | Amaro (`deps/amaro`) | `doc/api/typescript.md` | NEW `doc/architecture/typescript.md` |
| F-010 SQLite | `lib/sqlite.js` | `lib/internal/sqlite*` | (SQLite bindings) | SQLite (bundled) | `doc/api/sqlite.md` | Cross-link from architecture index |
| F-011 SEA | `lib/sea.js` | `lib/internal/sea*` | (SEA build tooling) | (none) | `doc/api/single-executable-applications.md` | Cross-link from architecture index |
| F-012 Permissions | (cross-cutting; `process.permission`) | `lib/internal/process/permission*` | `src/permission/` (18 files) | (none) | `doc/api/permissions.md` | NEW `doc/architecture/permission-model.md` |
| F-013 Web Standards | (globals: `fetch`, `Request`, `Response`, `Headers`, `FormData`, `Blob`, `crypto.subtle`) | `lib/internal/webstreams/`, `lib/internal/webidl/`, `lib/internal/blob*` | `src/crypto/` (Web Crypto), undici binding | undici, OpenSSL | `doc/api/webcrypto.md`, `webstreams.md`, `globals.md` | NEW `doc/architecture/web-standards.md` |

**Configuration Options Requiring Documentation Cross-Reference:**

The runtime exposes configuration through CLI flags, environment variables, and build-time GYP variables. Existing documentation already covers these surfaces; the new architecture index will cross-link them rather than duplicate.

| Configuration Surface | Existing Doc | Source |
|---|---|---|
| CLI flags | `doc/api/cli.md` | Comprehensive — all flags documented |
| Environment variables | `doc/api/environment_variables.md` | Comprehensive |
| Build-time toggles (`node_use_*`) | `BUILDING.md`, `configure.py`, `common.gypi` | Build-side |
| `package.json` "type" field | `doc/api/packages.md` | Comprehensive |
| `tsconfig.json` for tooling | `tsconfig.json` itself | Code-only |
| Lint configuration | `eslint.config.mjs`, `.clang-format`, `.cpplint`, `pyproject.toml`, `.yamllint.yaml` | Code-only |

**Features Requiring Consolidated User-Journey Guides:**

The user's request for "integration the expectation" implies user-journey documentation across feature boundaries. The new `doc/architecture/integration.md` will consolidate three integration journeys:

| Integration Journey | Current Coverage | Gap | Resolution |
|---|---|---|---|
| Application code embedding `node` (CLI runtime) | `doc/api/cli.md`, `doc/api/synopsis.md` | No layered architecture overview | NEW `doc/architecture/integration.md` |
| Native addon authors embedding via N-API | `doc/api/n-api.md` (extensive), `doc/api/addons.md` | No quick-orientation guide | UPDATE `doc/api/n-api.md` cross-links |
| Embedders linking the runtime as a C++ library | `doc/api/embedding.md` | Standalone, not cross-linked | UPDATE `doc/api/embedding.md` cross-links |

### 0.3.2 Documentation Gap Analysis

Given the requirements and repository analysis, documentation gaps include:

**Gaps That Will Be Addressed:**

| Gap Category | Specific Gap | Resolution |
|---|---|---|
| **Feature index** | No single page consolidates the 13 features (F-001 — F-013) for new readers | CREATE `doc/architecture/README.md` and `doc/architecture/features.md` |
| **Architecture overview** | No layered-architecture diagram in `doc/api/` (only in tech spec, which is internal) | CREATE `doc/architecture/overview.md` with Mermaid layer diagrams |
| **Integration narrative** | The bundled-dependency contract (V8 `-node.17`, OpenSSL, libuv, nghttp2, c-ares, undici, ICU, SQLite, Amaro) is split across `BUILDING.md`, `common.gypi`, `node.gyp`, with no narrative summary | CREATE `doc/architecture/dependencies.md` |
| **Development overview** | 45 contributor files exist with no top-level index distinguishing dev workflows from test workflows | CREATE `doc/contributing/development-overview.md` |
| **Testing overview** | Only `writing-tests.md` exists in `doc/contributing/`; coverage, benchmarks, lint, and CI gates are scattered across `BUILDING.md` and individual workflow YAMLs | CREATE `doc/contributing/testing-overview.md` |
| **Repository orientation** | Root `README.md` is a placeholder ("test project for backprop integration") and does not orient readers to the documentation structure | UPDATE root `README.md` |
| **API index** | `doc/api/index.md` lists API modules but does not link to the new architecture or contributing overviews | UPDATE `doc/api/index.md` to include architecture and contributing entry points |
| **Native cross-links** | `doc/api/n-api.md` and `doc/api/embedding.md` do not cross-link to each other or to the new integration guide | UPDATE both with cross-references |

**Gaps That Will NOT Be Addressed (Out of Scope per User's Rule "Document code"):**

- The codebase itself (no source files modified)
- The 6,606 test files in `test/` (no test additions or removals)
- The 591 benchmark files (no benchmark changes)
- The bundled dependencies under `deps/` (governed by upstream projects)
- The 37 GitHub Actions workflows (already self-documenting via YAML)
- Existing API references in `doc/api/` (preserved as-is — the architecture overview cross-links to them, but does not modify their content)

**Outdated Documentation Identified:**

No outdated documentation has been identified within the scope of this task. The repository is the upstream Node.js v26.0.0-pre source tree and the existing docs reflect the current state of the runtime as of the pre-release.


## 0.4 Documentation Implementation Design

### 0.4.1 Documentation Structure Planning

The documentation will be organized in a way that **complements**, rather than replaces, the existing `doc/api/` and `doc/contributing/` trees. A new `doc/architecture/` directory will host the synthesized cross-feature material, and the existing `doc/contributing/` directory will receive two new top-level guides that distinguish development from testing concerns.

**Documentation Hierarchy (Target Tree):**

```text
Vs_Repo-main/
├── README.md                                     [unchanged — upstream Node.js README]
├── BUILDING.md                                   [unchanged]
├── CONTRIBUTING.md                               [unchanged]
├── doc/
│   ├── README.md                                 [unchanged — Node.js doc style guide]
│   ├── api/
│   │   ├── index.md                              [UPDATE — add architecture/contributing entry points]
│   │   ├── n-api.md                              [UPDATE — cross-link to integration guide]
│   │   ├── embedding.md                          [UPDATE — cross-link to integration guide]
│   │   └── ... (66 other files unchanged)
│   ├── architecture/                             [NEW DIRECTORY]
│   │   ├── README.md                             [NEW — landing page for architecture docs]
│   │   ├── overview.md                           [NEW — 5-layer architecture overview]
│   │   ├── features.md                           [NEW — F-001..F-013 catalog]
│   │   ├── runtime.md                            [NEW — F-001 deep dive]
│   │   ├── networking.md                         [NEW — F-002 deep dive]
│   │   ├── file-system.md                        [NEW — F-003 deep dive]
│   │   ├── cryptography.md                       [NEW — F-004 deep dive]
│   │   ├── concurrency.md                        [NEW — F-005 deep dive]
│   │   ├── streams.md                            [NEW — F-006 deep dive]
│   │   ├── diagnostics.md                        [NEW — F-007 deep dive]
│   │   ├── typescript.md                         [NEW — F-009 deep dive]
│   │   ├── permission-model.md                   [NEW — F-012 deep dive]
│   │   ├── web-standards.md                      [NEW — F-013 deep dive]
│   │   ├── integration.md                        [NEW — bundled dep + N-API + embedding integration]
│   │   └── dependencies.md                       [NEW — V8/libuv/OpenSSL/etc. integration matrix]
│   └── contributing/
│       ├── development-overview.md               [NEW — index of all dev workflows]
│       ├── testing-overview.md                   [NEW — index of all test workflows]
│       └── ... (45 files unchanged)
└── README.md (root)                              [UPDATE — orientation to new docs]
```

The repository root contains a placeholder `README.md`. The new structure preserves the upstream Node.js documentation tree under `Vs_Repo-main/` and only adds the synthesis layer plus root orientation.

### 0.4.2 Content Generation Strategy

#### Information Extraction Approach

| Extraction Source | Information Extracted | Used By |
|---|---|---|
| `lib/*.js` (54 modules) | Public API surface, entry-point delegations to `lib/internal/` | `doc/architecture/features.md`, per-feature deep-dives |
| `lib/internal/**/*.js` | Implementation entry points, primordials usage | Per-feature deep-dives |
| `src/**/*.cc`, `src/**/*.h` | Native bindings file counts and subdirectory purposes | `doc/architecture/overview.md`, per-feature deep-dives |
| `common.gypi`, `node.gyp`, `node.gypi`, `configure.py` | Build-time toggles (`node_use_amaro`, `node_use_sqlite`, `node_use_openssl`, `node_shared_*`), embedder string `-node.17` | `doc/architecture/dependencies.md`, `doc/architecture/integration.md` |
| `src/node_version.h` | `NODE_MAJOR_VERSION 26`, `NODE_MODULE_VERSION 144`, `NODE_API_SUPPORTED_VERSION_MAX 10`, `NODE_VERSION_IS_RELEASE 0` | All architecture docs (header banner) |
| `BUILDING.md` | Compiler matrix, platform tiers, build/test/doc commands | `doc/contributing/development-overview.md`, `doc/contributing/testing-overview.md` |
| `Makefile` | Canonical targets — `doc`, `doc-only`, `docserve`, `docopen`, `docclean`, `test`, `test-only`, `test-doc`, `test-doc-ci`, `lint`, `lint-md`, `lint-js-fix` | Both contributor overviews (commands referenced verbatim) |
| `.github/workflows/*.yml` (37 files) | CI workflow categories, triggers, runner platforms | `doc/contributing/testing-overview.md` |
| `eslint.config.mjs`, `.clang-format`, `.cpplint`, `pyproject.toml`, `.yamllint.yaml` | Lint tool inventory and configuration locations | `doc/contributing/testing-overview.md` |
| `.nycrc`, `codecov.yml` | Coverage thresholds (95% line / 93% branch / 95% statement) | `doc/contributing/testing-overview.md` |
| Existing tech-spec sections §1, §2, §3, §5, §6 | Validated feature catalogs, dependency lists, architecture diagrams | All NEW docs (cited as authoritative) |

#### Template Application

The user did not provide a documentation template. The Blitzy platform will apply the **existing repository style** as the template, derived from `doc/README.md` and observed in 68 API reference files:

- File header: `# <Title>` followed by an introductory paragraph
- Stability index where applicable: line beginning with `> Stability: <level> - <description>`
- YAML metadata block as HTML comment with `added:` and `changes:` keys
- Source-link annotation as HTML comment, e.g., `source_link=lib/<module>.js`
- Headings: `##` for major sections, `###` for sub-sections
- Code samples: parallel `mjs` and `cjs` fenced blocks for module imports
- Mermaid diagrams in fenced blocks
- Footnote references at file end
- Wrapping at 120 characters per line per `.editorconfig` and `doc/README.md`
- US English spelling (per `doc/README.md`)

For the new `doc/architecture/` deliverables (which are conceptual rather than reference), the Blitzy platform will follow the **existing `doc/contributing/` narrative style** — Markdown with tables, Mermaid diagrams, and inline source-path citations rather than the strict API-reference format.

#### Documentation Standards

| Standard | Implementation |
|---|---|
| Markdown formatting | GitHub-flavored Markdown with `#`/`##`/`###` headers, fenced code blocks, tables, lists |
| Mermaid diagrams | Inline fenced blocks; rendered by doc-kit |
| Code examples | Fenced blocks with language identifiers (`js`, `mjs`, `cjs`, `bash`, `cpp`, `python`) |
| Source citations | Inline backticked paths plus footnotes; format `path/to/file.js` or `path/to/file.cc:LineNumber` |
| Tables | GitHub-flavored Markdown tables for parameter lists, feature inventories, and dependency matrices |
| Terminology | Aligned with `Vs_Repo-main/glossary.md` and `doc/api/documentation.md` (stability index) |
| Naming | `lowercase-with-dashes.md` for filenames per `doc/README.md` |
| Line length | 120-character wrap per `doc/README.md` and `.editorconfig` |
| Spelling | US English per `doc/README.md` |
| Linting | `make lint-md` (ESLint + `@eslint/markdown`) before commit |
| Validation | `make test-doc -j` to verify doc-kit compatibility |

### 0.4.3 Diagram and Visual Strategy

The Blitzy platform will produce Mermaid diagrams for every new architecture document. The diagrams already validated in the upstream tech spec (§1.2, §5.1, §6.6) are authoritative and will be carried into the deliverables with adjusted captions.

**Mermaid Diagrams to Create:**

| Diagram Type | Subject | Target Document | Source Reference |
|---|---|---|---|
| Layered architecture (flowchart TB) | 5 layers: User Space → `lib/` → `lib/internal/` → `src/` → `deps/` | `doc/architecture/overview.md` | §5.1.1 |
| Feature catalog (flowchart) | F-001 — F-013 grouped by tier | `doc/architecture/features.md` | §2.1 |
| Networking flow (sequence) | Inbound HTTP request: TCP → TLS → HTTP/1.1, HTTP/2, or QUIC | `doc/architecture/networking.md` | §5.1.3, §4.3 |
| Module loading flow (flowchart) | Specifier → cache → compile → execute (ESM and CJS branches) | `doc/architecture/runtime.md` | §5.1.3 |
| Event loop cycle (flowchart) | 6-phase libuv loop with microtask checkpoints | `doc/architecture/runtime.md` | §5.1.3 |
| Permission model enforcement (sequence) | Public API → permission check → resource access | `doc/architecture/permission-model.md` | §6.6.7 |
| Test runner sequence | `node --test` → discovery → execution → coverage → TAP | `doc/contributing/testing-overview.md` | §6.6.4.2 |
| CI workflow categories (flowchart) | Triggers → testing/coverage/security/linting/release/maintenance workflows | `doc/contributing/testing-overview.md` | §6.6.5.1 |
| Build pipeline (flowchart) | Configure → GYP → compile → link → test/doc | `doc/contributing/development-overview.md` | §4.10 |
| Dependency integration (flowchart LR) | `node` binary statically linking V8/libuv/OpenSSL/nghttp2/c-ares/ICU/SQLite/Amaro/undici | `doc/architecture/dependencies.md` | §3.7 |

**Screenshot/Image Requirements:**

None. The Node.js runtime has no UI surface (per §7) and all visual material is diagram-based.

**Architecture Diagram Specifications:**

All Mermaid diagrams will be authored as fenced blocks inside Markdown files, adhering to the doc-kit rendering pipeline. No external image files (`.png`, `.svg`) will be created or referenced. Existing image assets under `doc/contributing/doc_img/` are unrelated to this task and will not be modified.


## 0.5 Documentation File Transformation Mapping

### 0.5.1 File-by-File Documentation Plan

This section enumerates every documentation file to be created, updated, or referenced. The transformation modes follow the prompt convention: **CREATE** (new file), **UPDATE** (in-place modification), **DELETE** (file removal — none required for this task), **REFERENCE** (existing file used as a style/structure exemplar).

| Target Documentation File | Transformation | Source Code/Docs | Content/Changes |
|---|---|---|---|
| `README.md` (repository root) | UPDATE | `Vs_Repo-main/README.md`, `doc/api/index.md`, new architecture/contributing files | Replace 60-byte placeholder with project orientation: project identity (test repo packaging Node.js v26.0.0-pre under `Vs_Repo-main/`), brief feature summary, links to `Vs_Repo-main/README.md`, `Vs_Repo-main/BUILDING.md`, `Vs_Repo-main/CONTRIBUTING.md`, and the new `doc/architecture/README.md` and `doc/contributing/development-overview.md` / `testing-overview.md` |
| `Vs_Repo-main/doc/architecture/README.md` | CREATE | `lib/`, `src/`, `doc/api/index.md` | Landing page for architecture documentation; orients readers across overview, features, per-feature deep-dives, integration, and dependencies; tabular index linking each architecture document |
| `Vs_Repo-main/doc/architecture/overview.md` | CREATE | `src/node_main.cc`, `lib/`, `lib/internal/`, `src/`, `deps/`, `common.gypi`, `node.gyp`, `src/node_version.h` | 5-layer architectural overview (User Space, Public API `lib/`, Internal `lib/internal/`, Native `src/`, Bundled `deps/`); Mermaid layered flowchart; design principles (single-threaded event loop, layered modules, static linking, cross-platform abstraction, compile-time toggling); references to §5.1 of the tech spec |
| `Vs_Repo-main/doc/architecture/features.md` | CREATE | `lib/*.js` (54 modules), `src/` subdirectories, §2.1 feature catalog | Consolidated catalog of F-001 — F-013; one row per feature with source paths, primary modules, native bindings, status; cross-link to per-feature deep-dives and to corresponding `doc/api/*.md` references |
| `Vs_Repo-main/doc/architecture/runtime.md` | CREATE | `src/node_main.cc`, `lib/module.js`, `lib/internal/modules/`, `deps/v8/`, `deps/uv/` | F-001 deep dive — V8 integration (embedder string `-node.17`), libuv event loop 6-phase cycle, ESM/CJS module system, JIT pipeline (Ignition → SparkPlug → Maglev → TurboFan); Mermaid event-loop and module-loading diagrams; cross-link to `doc/api/module.md`, `modules.md`, `esm.md`, `packages.md` |
| `Vs_Repo-main/doc/architecture/networking.md` | CREATE | `lib/http.js`, `https.js`, `http2.js`, `net.js`, `dgram.js`, `dns.js`, `tls.js`, `quic.js`, `lib/internal/http*`, `lib/internal/quic/`, `src/quic/` | F-002 deep dive — protocol matrix (HTTP/1.1 via llhttp, HTTP/2 via nghttp2, QUIC via ngtcp2/nghttp3), TLS via OpenSSL, DNS via c-ares, `fetch()` via undici; Mermaid request flow; cross-link to `doc/api/http.md`, `http2.md`, `https.md`, `net.md`, `dgram.md`, `dns.md`, `tls.md`, `quic.md` |
| `Vs_Repo-main/doc/architecture/file-system.md` | CREATE | `lib/fs.js`, `lib/internal/fs/` | F-003 deep dive — three API styles (callback, sync, promises), libuv abstraction across Linux/macOS/Windows, watcher capabilities, permission-model interaction; cross-link to `doc/api/fs.md`, `permissions.md` |
| `Vs_Repo-main/doc/architecture/cryptography.md` | CREATE | `lib/crypto.js`, `lib/internal/crypto/`, `src/crypto/` (63 files) | F-004 deep dive — OpenSSL backing (`node_use_openssl: true`), traditional `crypto` vs Web Crypto API, FIPS-compliant build mode (`BUILDING.md`), `src/crypto/` subdivision; cross-link to `doc/api/crypto.md`, `tls.md`, `webcrypto.md` |
| `Vs_Repo-main/doc/architecture/concurrency.md` | CREATE | `lib/child_process.js`, `cluster.js`, `worker_threads.js`, `lib/internal/worker/`, `lib/internal/cluster/` | F-005 deep dive — worker threads (separate V8 isolates), child processes (`exec`, `execFile`, `fork`, `spawn`), cluster mode (round-robin scheduling); permission-model interaction; cross-link to `doc/api/child_process.md`, `cluster.md`, `worker_threads.md` |
| `Vs_Repo-main/doc/architecture/streams.md` | CREATE | `lib/stream.js`, `lib/internal/streams/`, `lib/internal/webstreams/` | F-006 deep dive — Readable/Writable/Transform/Duplex types, backpressure semantics, Web Streams alignment; cross-link to `doc/api/stream.md`, `webstreams.md`, `stream_iter.md` |
| `Vs_Repo-main/doc/architecture/diagnostics.md` | CREATE | `lib/inspector.js`, `perf_hooks.js`, `diagnostics_channel.js`, `trace_events.js`, `async_hooks.js`, `src/inspector/` (50 files), `src/tracing/` (12 files) | F-007 deep dive — Inspector protocol (Chrome DevTools), perf hooks, diagnostics channel pub/sub, trace events, async hooks; cross-link to `doc/api/inspector.md`, `perf_hooks.md`, `diagnostics_channel.md`, `tracing.md`, `async_hooks.md`, `report.md` |
| `Vs_Repo-main/doc/architecture/typescript.md` | CREATE | `lib/internal/modules/`, `deps/amaro`, `common.gypi` (`node_use_amaro: true`) | F-009 deep dive — Amaro (SWC WASM) type stripping vs `--experimental-transform-types`, default behavior, source-map generation; cross-link to `doc/api/typescript.md` |
| `Vs_Repo-main/doc/architecture/permission-model.md` | CREATE | `src/permission/` (18 files), `lib/internal/process/permission*`, `--permission` CLI flag | F-012 deep dive — default-deny model, allow flags (`--allow-fs-read`, `--allow-fs-write`, `--allow-child-process`, `--allow-worker`, `--allow-wasi`, `--allow-addons`, `--allow-net`), known constraints (worker thread non-inheritance, symlink resolution); cross-link to `doc/api/permissions.md` |
| `Vs_Repo-main/doc/architecture/web-standards.md` | CREATE | `lib/internal/webstreams/`, `lib/internal/blob*`, `lib/crypto.js`, `src/crypto/`, undici binding | F-013 deep dive — `fetch()` via undici, Web Streams (ReadableStream/WritableStream/TransformStream), Web Crypto via `crypto.subtle`, FormData/Blob/Headers/Request/Response globals; cross-link to `doc/api/webcrypto.md`, `webstreams.md`, `globals.md` |
| `Vs_Repo-main/doc/architecture/integration.md` | CREATE | `src/api/` (9 files), `doc/api/n-api.md`, `doc/api/embedding.md`, `doc/api/addons.md`, `common.gypi`, `node.gyp` | Integration narrative — three integration personas: (1) application authors (CLI runtime, `lib/` API), (2) addon authors (N-API, `NODE_API_SUPPORTED_VERSION_MAX 10`), (3) embedders (C++ embedding API in `src/api/`); ABI compatibility guarantees (`NODE_MODULE_VERSION 144`); Mermaid integration-points diagram |
| `Vs_Repo-main/doc/architecture/dependencies.md` | CREATE | `common.gypi`, `node.gyp`, `configure.py`, `deps/`, `tools/dep_updaters/` | Dependency integration matrix — table of bundled dependencies (V8 with embedder string `-node.17`, libuv, OpenSSL, nghttp2, c-ares, undici, ICU, SQLite, Amaro, simdutf, zlib, brotli, zstd) with version source locations, build flags (`node_use_*`, `node_shared_*`), and update automation (`tools/dep_updaters/`) |
| `Vs_Repo-main/doc/contributing/development-overview.md` | CREATE | `BUILDING.md`, `Makefile`, `vcbuild.bat`, `configure.py`, all 45 files in `doc/contributing/` (excluding `writing-tests.md`) | Index of development workflows — building (Make / vcbuild), configuring (`configure.py` flags), debugging (Inspector, ASan), code style (clang-format, ESLint, ruff), dependency updates (`tools/dep_updaters/`), releases, onboarding; one-paragraph orientation per workflow with deep-link to the existing detailed contributor doc |
| `Vs_Repo-main/doc/contributing/testing-overview.md` | CREATE | `BUILDING.md`, `Makefile`, `tools/test.py`, `test/`, `benchmark/`, `.github/workflows/`, `eslint.config.mjs`, `.clang-format`, `.cpplint`, `pyproject.toml`, `.yamllint.yaml`, `.nycrc`, `codecov.yml`, `doc/contributing/writing-tests.md` | Index of testing workflows — running tests (`make test`, `make test-only`, `tools/test.py`), built-in test runner (`node --test`), categories (parallel/sequential/wpt/sea/etc.), benchmarks (`benchmark/`), linting (`make lint`, `make lint-md`, `make lint-js-fix`), coverage (nyc + Codecov, 95/93/95 thresholds), security scans (CodeQL, OSSF Scorecard), CI workflows (37 in `.github/workflows/`); cross-link to `writing-tests.md`, `static-analysis.md`, `doc/api/test.md`, `doc/api/assert.md` |
| `Vs_Repo-main/doc/api/index.md` | UPDATE | `doc/api/index.md` | Insert two new entries at the top of the page: (1) "Architecture overview" linking to `../architecture/README.md`; (2) "Contributing" linking to `../contributing/development-overview.md` and `../contributing/testing-overview.md` |
| `Vs_Repo-main/doc/api/n-api.md` | UPDATE | `doc/api/n-api.md` | Add a "See also" link near the top pointing to `../architecture/integration.md` for layered-architecture context |
| `Vs_Repo-main/doc/api/embedding.md` | UPDATE | `doc/api/embedding.md` | Add a "See also" link near the top pointing to `../architecture/integration.md` and to `../architecture/runtime.md` |
| `Vs_Repo-main/doc/api/test.md` | REFERENCE | n/a | Used as the canonical style/structure exemplar for any reference-style content in `doc/architecture/` |
| `Vs_Repo-main/doc/contributing/writing-tests.md` | REFERENCE | n/a | Used as the canonical style exemplar for `doc/contributing/testing-overview.md` |
| `Vs_Repo-main/doc/contributing/writing-docs.md` | REFERENCE | n/a | Used as the canonical style exemplar for the new architecture documents |
| `Vs_Repo-main/doc/README.md` | REFERENCE | n/a | Documentation style guide — applied to every new file (filename casing, line wrap, spelling, headings) |
| `Vs_Repo-main/doc/api/documentation.md` | REFERENCE | n/a | Stability-index policy — applied where stability claims are relevant in `doc/architecture/features.md` |
| `Vs_Repo-main/doc/contributing/api-documentation.md` | REFERENCE | n/a | Doc-kit pipeline reference — relevant for understanding how new files render |

**Coverage of all in-scope documentation files is exhaustive — nothing is left as "pending" or "to be discovered".**

### 0.5.2 New Documentation Files Detail

For each new documentation file the Blitzy platform will create, the following blueprint applies. Each blueprint pre-defines sections, source citations, and any required diagrams.

```text
File: Vs_Repo-main/doc/architecture/README.md
Type: Architecture Landing Page
Source Code: doc/api/index.md (style), lib/, src/
Sections:
    - Purpose (orientation for the architecture documentation)
    - Reading order (overview → features → per-feature deep-dives → integration → dependencies)
    - Index table (every doc/architecture/*.md file linked with one-line summary)
    - Cross-links (to doc/api/index.md, doc/contributing/development-overview.md, doc/contributing/testing-overview.md)
Diagrams:
    - None (landing page)
Key Citations: doc/api/index.md, lib/, src/
```

```text
File: Vs_Repo-main/doc/architecture/overview.md
Type: Architectural Overview
Source Code: src/node_main.cc, lib/, lib/internal/, src/, deps/, src/node_version.h, common.gypi
Sections:
    - Architecture style (event-driven, non-blocking I/O, layered)
    - The 5 layers (User Space, Public API, Internal Modules, Native Layer, Bundled Dependencies)
    - Layer interactions (inbound flow user → V8 → application; outbound flow application → public API → internal → native → deps)
    - Architectural principles (single-threaded event loop, layered modules, static linking, cross-platform abstraction, compile-time toggling)
    - System boundaries (the `node` binary as boundary; entry at src/node_main.cc)
    - Version constants (NODE_MAJOR_VERSION 26, NODE_MODULE_VERSION 144, NODE_API_SUPPORTED_VERSION_MAX 10, NODE_VERSION_IS_RELEASE 0)
Diagrams:
    - Mermaid layered architecture flowchart (5 layers)
Key Citations: src/node_main.cc, src/node_version.h, common.gypi, node.gyp, lib/, src/, deps/
```

```text
File: Vs_Repo-main/doc/architecture/features.md
Type: Feature Catalog
Source Code: §2.1 feature catalog, lib/*.js, src/ subdirectories
Sections:
    - Feature taxonomy (Core Runtime, Platform Capability, Modern Platform tiers)
    - Feature inventory table (F-001 — F-013 with source paths and existing API doc links)
    - Feature relationship matrix (cross-feature dependencies)
    - Status notation (all "In Development" per NODE_VERSION_IS_RELEASE 0)
Diagrams:
    - Mermaid feature-tier flowchart
Key Citations: lib/, src/, src/node_version.h, doc/api/*.md (for cross-links)
```

```text
File: Vs_Repo-main/doc/architecture/runtime.md
Type: Feature Deep Dive (F-001)
Source Code: src/node_main.cc, lib/module.js, lib/internal/modules/, deps/v8/, deps/uv/, common.gypi
Sections:
    - V8 integration (embedder string `-node.17` per common.gypi)
    - libuv event loop (6-phase cycle: timers, pending callbacks, idle/prepare, I/O poll, check, close callbacks)
    - Microtask queues (process.nextTick, Promise callbacks)
    - Module systems (ESM via lib/internal/modules/, CJS via lib/module.js)
    - JIT compilation pipeline (Ignition → SparkPlug → Maglev → TurboFan)
Diagrams:
    - Mermaid 6-phase event loop cycle
    - Mermaid module loading flow (specifier → cache → compile → execute)
Key Citations: src/node_main.cc, lib/module.js, lib/internal/modules/, common.gypi, doc/api/module.md, modules.md, esm.md, packages.md
```

```text
File: Vs_Repo-main/doc/architecture/networking.md
Type: Feature Deep Dive (F-002)
Source Code: lib/http.js, https.js, http2.js, net.js, dgram.js, dns.js, tls.js, quic.js, lib/internal/quic/, src/quic/
Sections:
    - Protocol matrix (HTTP/1.1 via llhttp, HTTP/2 via nghttp2, HTTPS, TCP via libuv, UDP via libuv, DNS via c-ares, TLS via OpenSSL, QUIC via ngtcp2/nghttp3)
    - undici as the modern HTTP/1.1 client backing fetch()
    - Permission Model interaction (--allow-net)
    - Header validation (VR-SEC-002 prototype pollution, VR-SEC-006 HPACK)
Diagrams:
    - Mermaid inbound request sequence (TCP → optional TLS → protocol selection → handler)
Key Citations: lib/http.js, lib/http2.js, lib/quic.js, src/quic/, doc/api/http.md, http2.md, https.md, net.md, dgram.md, dns.md, tls.md, quic.md
```

```text
File: Vs_Repo-main/doc/architecture/file-system.md
Type: Feature Deep Dive (F-003)
Source Code: lib/fs.js, lib/internal/fs/
Sections:
    - Three API styles (callback, sync, promises)
    - Cross-platform abstraction via libuv
    - File watchers
    - Permission Model interaction (--allow-fs-read, --allow-fs-write)
Diagrams:
    - None required (referential)
Key Citations: lib/fs.js, lib/internal/fs/, doc/api/fs.md, permissions.md
```

```text
File: Vs_Repo-main/doc/architecture/cryptography.md
Type: Feature Deep Dive (F-004)
Source Code: lib/crypto.js, lib/internal/crypto/, src/crypto/ (63 files), common.gypi (node_use_openssl)
Sections:
    - OpenSSL backing
    - Traditional crypto module vs Web Crypto API
    - FIPS-compliant build mode (per BUILDING.md)
    - src/crypto/ structural breakdown
Diagrams:
    - None required (referential)
Key Citations: lib/crypto.js, src/crypto/, common.gypi, BUILDING.md, doc/api/crypto.md, tls.md, webcrypto.md
```

```text
File: Vs_Repo-main/doc/architecture/concurrency.md
Type: Feature Deep Dive (F-005)
Source Code: lib/child_process.js, lib/cluster.js, lib/worker_threads.js, lib/internal/worker/, lib/internal/cluster/
Sections:
    - Worker threads (separate V8 isolates, SharedArrayBuffer transfer)
    - Child processes (exec, execFile, fork, spawn)
    - Cluster mode (round-robin scheduling default on non-Windows)
    - Permission Model interaction
Diagrams:
    - None required (referential)
Key Citations: lib/child_process.js, lib/cluster.js, lib/worker_threads.js, doc/api/child_process.md, cluster.md, worker_threads.md
```

```text
File: Vs_Repo-main/doc/architecture/streams.md
Type: Feature Deep Dive (F-006)
Source Code: lib/stream.js, lib/internal/streams/, lib/internal/webstreams/
Sections:
    - Stream types (Readable, Writable, Transform, Duplex)
    - Backpressure semantics
    - Pipe composition
    - Web Streams (ReadableStream, WritableStream, TransformStream)
Diagrams:
    - None required (referential)
Key Citations: lib/stream.js, lib/internal/streams/, lib/internal/webstreams/, doc/api/stream.md, webstreams.md, stream_iter.md
```

```text
File: Vs_Repo-main/doc/architecture/diagnostics.md
Type: Feature Deep Dive (F-007)
Source Code: lib/inspector.js, perf_hooks.js, diagnostics_channel.js, trace_events.js, async_hooks.js, src/inspector/ (50 files), src/tracing/ (12 files)
Sections:
    - Inspector protocol (Chrome DevTools)
    - Performance hooks (high-resolution timing)
    - Diagnostics channel (pub/sub for cross-cutting events)
    - Trace events
    - Async hooks
Diagrams:
    - None required (referential)
Key Citations: lib/inspector.js, lib/perf_hooks.js, lib/diagnostics_channel.js, lib/trace_events.js, lib/async_hooks.js, src/inspector/, src/tracing/, doc/api/inspector.md, perf_hooks.md, diagnostics_channel.md, tracing.md, async_hooks.md, report.md
```

```text
File: Vs_Repo-main/doc/architecture/typescript.md
Type: Feature Deep Dive (F-009)
Source Code: lib/internal/modules/, deps/amaro, common.gypi (node_use_amaro)
Sections:
    - Amaro integration (SWC WASM type stripping)
    - Default mode (--experimental-strip-types) vs transform mode (--experimental-transform-types)
    - Source map generation
    - Limitations (erasable syntax, no enum/namespace in default mode)
Diagrams:
    - None required (referential)
Key Citations: lib/internal/modules/, deps/amaro, common.gypi, doc/api/typescript.md
```

```text
File: Vs_Repo-main/doc/architecture/permission-model.md
Type: Feature Deep Dive (F-012)
Source Code: src/permission/ (18 files), lib/internal/process/permission*, --permission CLI flag
Sections:
    - Default-deny model
    - Resource-specific allow flags (--allow-fs-read, --allow-fs-write, --allow-child-process, --allow-worker, --allow-wasi, --allow-addons, --allow-net)
    - Cross-cutting enforcement (file system, networking, child processes, worker threads, SQLite extensions)
    - Known constraints (worker thread non-inheritance, symlink path resolution VR-SEC-005, --env-file/--openssl-config bypass before init)
Diagrams:
    - Mermaid permission enforcement sequence (public API → check → resource access)
Key Citations: src/permission/, doc/api/permissions.md, SECURITY.md
```

```text
File: Vs_Repo-main/doc/architecture/web-standards.md
Type: Feature Deep Dive (F-013)
Source Code: lib/internal/webstreams/, lib/internal/blob*, lib/crypto.js, src/crypto/, undici binding
Sections:
    - fetch() backed by undici
    - Web Streams in lib/internal/webstreams/
    - Web Crypto via crypto.subtle and src/crypto/
    - Globals (FormData, Blob, Headers, Request, Response)
Diagrams:
    - None required (referential)
Key Citations: lib/internal/webstreams/, lib/crypto.js, src/crypto/, doc/api/webcrypto.md, webstreams.md, globals.md
```

```text
File: Vs_Repo-main/doc/architecture/integration.md
Type: Integration Guide
Source Code: src/api/ (9 files), doc/api/n-api.md, doc/api/embedding.md, doc/api/addons.md, common.gypi, node.gyp, src/node_version.h
Sections:
    - Three integration personas (application authors, addon authors, embedders)
    - Application integration (CLI invocation, lib/ API, package.json type field)
    - N-API integration (NODE_API_SUPPORTED_VERSION_MAX 10, ABI stability)
    - Embedding integration (src/api/ public embedding API, NODE_MODULE_VERSION 144)
    - External infrastructure integration (CI/CD, Codecov, CodeQL, OSSF Scorecard, HackerOne)
Diagrams:
    - Mermaid integration-points diagram (3 personas + 5 external integrations)
Key Citations: src/api/, src/node_version.h, common.gypi, node.gyp, doc/api/n-api.md, doc/api/embedding.md, doc/api/addons.md
```

```text
File: Vs_Repo-main/doc/architecture/dependencies.md
Type: Dependency Integration Matrix
Source Code: common.gypi, node.gyp, configure.py, deps/, tools/dep_updaters/
Sections:
    - Bundled dependencies overview
    - Dependency matrix (V8, libuv, OpenSSL, nghttp2, c-ares, undici, ICU, SQLite, Amaro, simdutf, zlib, brotli, zstd, ngtcp2, nghttp3)
    - Build flags (node_use_*, node_shared_*)
    - V8 patches (embedder string -node.17 indicates 17 Node.js-specific patches)
    - Update automation (tools/dep_updaters/ — 32 dependencies tracked)
Diagrams:
    - Mermaid dependency-integration flowchart (LR; node binary → static linking → dependencies)
Key Citations: common.gypi, node.gyp, configure.py, deps/, tools/dep_updaters/
```

```text
File: Vs_Repo-main/doc/contributing/development-overview.md
Type: Development Workflow Index
Source Code: BUILDING.md, Makefile, vcbuild.bat, configure.py, doc/contributing/*.md (excluding writing-tests.md)
Sections:
    - Scope (development workflows ONLY — explicit exclusion of testing concerns)
    - Building from source (Unix/macOS via Makefile; Windows via vcbuild.bat; alternatives: Ninja per building-node-with-ninja.md, GN per gn-build.md)
    - Configuring (configure.py flags, common.gypi toggles, BUILDING.md options)
    - Code style (clang-format per cpp-style-guide.md, ESLint, ruff, yamllint)
    - Adding native APIs (adding-new-napi-api.md, adding-v8-fast-api.md)
    - Maintaining dependencies (per-dependency guides under doc/contributing/maintaining/)
    - Submitting changes (pull-requests.md, commit-queue.md, collaborator-guide.md, releases.md, backporting-to-release-lines.md)
    - Debugging (investigating-native-memory-leaks.md, node-postmortem-support.md, diagnostic-tooling-support-tiers.md, static-analysis.md)
    - Onboarding (onboarding.md, using-devcontainer.md)
    - Pointer to testing-overview.md for test-related workflows
Diagrams:
    - Mermaid build pipeline (configure → GYP → compile → link → produce node binary)
Key Citations: BUILDING.md, Makefile, vcbuild.bat, configure.py, doc/contributing/pull-requests.md, doc/contributing/cpp-style-guide.md, doc/contributing/api-documentation.md, doc/contributing/writing-docs.md
```

```text
File: Vs_Repo-main/doc/contributing/testing-overview.md
Type: Testing Workflow Index
Source Code: BUILDING.md, Makefile, tools/test.py, test/, benchmark/, .github/workflows/, eslint.config.mjs, .clang-format, .cpplint, pyproject.toml, .yamllint.yaml, .nycrc, codecov.yml, doc/contributing/writing-tests.md
Sections:
    - Scope (testing workflows ONLY — explicit exclusion of build/code/release concerns)
    - Test corpus (11,060 files across 37 categories)
    - Running tests (`make test`, `make test-only`, `tools/test.py`, `tools/test.py <subsystem>`, single-file invocation)
    - Built-in test runner (`node --test`, F-008)
    - Test categories (parallel/4102, sequential/121, fixtures/5006, es-module/226, addons/226, js-native-api/176, node-api/131, module-hooks/113, test-runner/109, async-hooks/101, wasi/87, pseudo-tty/82, wpt/61, sea/39, cctest/36)
    - Benchmarks (591 files in benchmark/)
    - Linting (`make lint`, `make lint-md`, `make lint-js-fix` — multi-language: ESLint/JS, clang-format/cpplint, ruff/Python, yamllint/YAML)
    - Coverage (nyc per .nycrc; Codecov per codecov.yml; thresholds 95% line / 93% branch / 95% statement)
    - CI workflows (37 in .github/workflows/ — testing, coverage, security, linting, release, maintenance categories)
    - Security testing (CodeQL daily SAST, OSSF Scorecard, HackerOne)
    - Web Platform Tests (WPT — daily-wpt-fyi.yml, update-wpt.yml)
    - Cross-link to writing-tests.md, doc/api/test.md, doc/api/assert.md
Diagrams:
    - Mermaid test runner sequence (`node --test` → discovery → execute → coverage → TAP)
    - Mermaid CI workflow categories (triggers → testing/coverage/security/linting/release/maintenance)
Key Citations: BUILDING.md, Makefile, tools/test.py, test/, benchmark/, .github/workflows/, .nycrc, codecov.yml, doc/contributing/writing-tests.md, doc/api/test.md, doc/api/assert.md
```

### 0.5.3 Documentation Files to Update Detail

**`README.md` (repository root)** — Replace placeholder with project orientation:

- New sections: Project Identity, About This Repository, Documentation Map, Quick Links, License
- Updated examples: None — root README is a navigation document, not a code-example surface
- New diagrams: None
- Source citations: Links to `Vs_Repo-main/README.md`, `Vs_Repo-main/BUILDING.md`, `Vs_Repo-main/CONTRIBUTING.md`, `Vs_Repo-main/doc/architecture/README.md`, `Vs_Repo-main/doc/contributing/development-overview.md`, `Vs_Repo-main/doc/contributing/testing-overview.md`

**`Vs_Repo-main/doc/api/index.md`** — Insert two new entries near the top:

- New sections: "Architecture overview" and "Contributing" entries above the existing "About this documentation" line
- Updated table of contents: Two new bullets pointing into `../architecture/README.md` and `../contributing/development-overview.md` / `testing-overview.md`
- New diagrams: None
- Source citations: Self-referential (links to architecture and contributing trees)

**`Vs_Repo-main/doc/api/n-api.md`** — Add cross-reference banner:

- New sections: "See also" banner at top with links to `../architecture/integration.md` and `../architecture/runtime.md`
- Updated content: Single inserted banner; no rewriting of existing content
- New diagrams: None
- Source citations: Self-referential to architecture docs

**`Vs_Repo-main/doc/api/embedding.md`** — Add cross-reference banner:

- New sections: "See also" banner at top with links to `../architecture/integration.md` and `../architecture/runtime.md`
- Updated content: Single inserted banner; no rewriting of existing content
- New diagrams: None
- Source citations: Self-referential to architecture docs

### 0.5.4 Documentation Configuration Updates

The Node.js documentation pipeline does not use `mkdocs.yml`, `docusaurus.config.js`, `.readthedocs.yml`, or Sphinx — it uses the in-house `@node-core/doc-kit` driven by `Makefile` targets. No configuration files require updates because:

- `Makefile` `doc-only` target enumerates input files dynamically based on glob patterns, so newly created `doc/architecture/*.md` and `doc/contributing/*.md` files do not require explicit registration.
- `tools/doc/package.json` pins `@node-core/doc-kit` at `1.0.2` and is unaffected by content additions.
- `eslint.config.mjs` includes `doc/eslint.config_partial.mjs` which applies markdown linting to all `*.md` files under `doc/` automatically.
- `doc/type-map.json` and `doc/node-config-schema.json` are referenced by doc-kit for API typing; the new architecture/contributing files are narrative (non-API) and do not require type-map entries.

**Configuration files that DO NOT require updates:**

| File | Reason |
|---|---|
| `Makefile` | Glob-based; new `doc/**/*.md` files picked up automatically |
| `tools/doc/package.json` | Tooling version unchanged |
| `eslint.config.mjs` | Glob-based; new `doc/**/*.md` files linted automatically |
| `doc/type-map.json` | Type mapping not required for narrative docs |
| `doc/node-config-schema.json` | Config schema unaffected |
| `.editorconfig` | Already specifies 2-space indent, LF line endings, charset UTF-8 |

### 0.5.5 Cross-Documentation Dependencies

**Shared content / includes:**

- The Node.js documentation pipeline does not use `include` directives. Cross-document references are resolved via Markdown links.
- Citations to source files use the format `path/to/file.ext` (inline backticks) or `path/to/file.ext:LineNumber` for line-specific references.

**Navigation links between documents:**

| Source Document | Links To |
|---|---|
| `README.md` (root) | `Vs_Repo-main/README.md`, `Vs_Repo-main/BUILDING.md`, `Vs_Repo-main/CONTRIBUTING.md`, `Vs_Repo-main/doc/architecture/README.md`, `Vs_Repo-main/doc/contributing/development-overview.md`, `Vs_Repo-main/doc/contributing/testing-overview.md` |
| `doc/architecture/README.md` | All `doc/architecture/*.md` siblings, `doc/api/index.md`, `doc/contributing/development-overview.md`, `doc/contributing/testing-overview.md` |
| `doc/architecture/overview.md` | `doc/architecture/features.md`, all per-feature deep-dives |
| `doc/architecture/features.md` | All per-feature deep-dives, all corresponding `doc/api/*.md` references |
| Per-feature deep-dives (e.g., `runtime.md`, `networking.md`) | Corresponding `doc/api/*.md` files, `doc/architecture/integration.md`, `doc/architecture/dependencies.md` |
| `doc/architecture/integration.md` | `doc/api/n-api.md`, `doc/api/embedding.md`, `doc/api/addons.md`, all per-feature deep-dives |
| `doc/architecture/dependencies.md` | `BUILDING.md`, `common.gypi`, `tools/dep_updaters/` (path references, not links) |
| `doc/contributing/development-overview.md` | All 45 existing `doc/contributing/*.md` files, `BUILDING.md`, `CONTRIBUTING.md`, `doc/contributing/testing-overview.md` (cross-link), `doc/contributing/api-documentation.md`, `doc/contributing/writing-docs.md` |
| `doc/contributing/testing-overview.md` | `doc/contributing/writing-tests.md`, `doc/contributing/static-analysis.md`, `doc/api/test.md`, `doc/api/assert.md`, `BUILDING.md`, `doc/contributing/development-overview.md` (cross-link) |
| `doc/api/index.md` | `../architecture/README.md`, `../contributing/development-overview.md`, `../contributing/testing-overview.md` (new entries) |
| `doc/api/n-api.md` | `../architecture/integration.md`, `../architecture/runtime.md` (new banner) |
| `doc/api/embedding.md` | `../architecture/integration.md`, `../architecture/runtime.md` (new banner) |

**Table of contents updates required:**

- `doc/api/index.md` — two new entries at the top (architecture and contributing)
- `doc/architecture/README.md` — full TOC of architecture docs (CREATE)
- `doc/contributing/development-overview.md` — TOC of dev workflows (CREATE)
- `doc/contributing/testing-overview.md` — TOC of test workflows (CREATE)
- Root `README.md` — TOC of doc landing pages (UPDATE)

**Index/glossary updates:**

- `Vs_Repo-main/glossary.md` is unchanged; the new docs use existing glossary terms (event loop, V8, libuv, N-API, etc.) and reference `glossary.md` from the architecture overview where appropriate.


## 0.6 Dependency Inventory

### 0.6.1 Documentation Dependencies

The documentation toolchain is fully self-contained inside the repository. The Blitzy platform will use the **already-pinned** documentation dependencies as found in `tools/doc/package.json` and the `Makefile` build targets. No new documentation dependencies will be added.

**Already-Pinned Documentation Dependencies (no change required):**

| Registry | Package Name | Version | Purpose | Source |
|---|---|---|---|---|
| npm | `@node-core/doc-kit` | `1.0.2` | Markdown to HTML/JSON pipeline; renders `doc/api/*.md` and (after this task) `doc/architecture/*.md` and `doc/contributing/*.md` | `tools/doc/package.json` |
| npm | `@eslint/markdown` | (resolved by `eslint.config.mjs`) | Markdown linting integrated with `make lint-md` | `eslint.config.mjs` |
| npm | `@eslint/js` | (resolved by `eslint.config.mjs`) | JavaScript linting (used during `make lint`) | `eslint.config.mjs` |
| npm | `@babel/eslint-parser` | (resolved by `eslint.config.mjs`) | Parser for ESLint over JavaScript and Markdown code blocks | `eslint.config.mjs` |
| npm | `eslint-formatter-tap` | (resolved by `eslint.config.mjs`) | TAP-format ESLint output | `eslint.config.mjs` |
| pip | `ruff` | (resolved by `pyproject.toml`) | Python linter (lints `tools/`, `configure.py`, etc.) | `pyproject.toml` |

**Build Tools Required (already documented in `BUILDING.md`):**

| Registry | Package Name | Version | Purpose | Source |
|---|---|---|---|---|
| system | Python | ≥ 3.10 (CI uses 3.14) | Build configuration scripting; required by `make doc` for doc-kit invocation | `BUILDING.md`, `pyproject.toml`, `.github/workflows/test-linux.yml` |
| system | Node.js | (built by `make`) | The doc generator runs **on the freshly built `node` binary** for `make doc`; on a pre-existing binary for `make doc-only` | `Makefile` (`DOC_KIT` invocation pattern) |
| system | GNU Make | (system default) | Drives `make doc`, `make doc-only`, `make docserve`, `make lint-md`, `make test-doc` | `Makefile` |

**Versioning Note:** All version values above reflect the EXACT names and versions present in the dependency manifests of the inspected repository. No placeholder versions ("latest", "1.0.0") have been used. The `@node-core/doc-kit` version `1.0.2` is verified verbatim in `Vs_Repo-main/tools/doc/package.json`.

**Tools Used by the Blitzy Platform for Authoring (no installation required):**

The Blitzy platform produces Markdown text files. Authoring requires no separate toolchain beyond a text editor; rendering, linting, and validation are performed by the repository's existing toolchain via the `Makefile` targets enumerated above.

**Diagram Tooling:**

| Type | Tool | Status |
|---|---|---|
| Mermaid diagrams | Inline fenced code blocks (rendered by `@node-core/doc-kit`) | No external CLI required |
| PlantUML / Graphviz / Lucidchart | Not used | n/a |
| Image authoring | Not required | n/a |

### 0.6.2 Documentation Reference Updates

**Documentation files requiring link updates:**

| Files | Update |
|---|---|
| Root `README.md` | New section linking into `Vs_Repo-main/doc/architecture/README.md`, `doc/contributing/development-overview.md`, `doc/contributing/testing-overview.md` |
| `Vs_Repo-main/doc/api/index.md` | New entries linking to `../architecture/README.md`, `../contributing/development-overview.md`, `../contributing/testing-overview.md` |
| `Vs_Repo-main/doc/api/n-api.md` | New "See also" banner linking to `../architecture/integration.md`, `../architecture/runtime.md` |
| `Vs_Repo-main/doc/api/embedding.md` | New "See also" banner linking to `../architecture/integration.md`, `../architecture/runtime.md` |

**Link transformation rules:**

| Pattern | Application |
|---|---|
| Inbound link from `doc/api/*.md` to `doc/architecture/*.md` | Use relative path `../architecture/<file>.md` |
| Inbound link from `doc/api/*.md` to `doc/contributing/*.md` | Use relative path `../contributing/<file>.md` |
| Inbound link from `doc/contributing/*.md` to `doc/api/*.md` | Use relative path `../api/<file>.md` |
| Inbound link from `doc/architecture/*.md` to `doc/api/*.md` | Use relative path `../api/<file>.md` |
| Inbound link from root `README.md` to anything in `Vs_Repo-main/` | Use relative path `Vs_Repo-main/<rest>` |
| Inbound link to `BUILDING.md`, `CONTRIBUTING.md`, `SECURITY.md` | Use absolute path within `Vs_Repo-main/` (e.g., `../../BUILDING.md` from `doc/architecture/`) |
| Source-code citations | Inline backticked path (e.g., `lib/http.js`); for line-specific references use `path/to/file.ext:LineNumber` |
| Cross-link between architecture docs | Use plain filename (e.g., `[Networking](networking.md)`) since they share a directory |
| Cross-link between contributing docs | Use plain filename (e.g., `[Writing tests](writing-tests.md)`) since they share a directory |

**Files explicitly NOT requiring link updates:**

- All 66 other `doc/api/*.md` files (only `index.md`, `n-api.md`, `embedding.md` are touched)
- All 45 existing `doc/contributing/*.md` files (development/testing overviews link OUT to them; they do not link back inbound)
- `BUILDING.md`, `CONTRIBUTING.md`, `SECURITY.md`, `GOVERNANCE.md`, `CHANGELOG.md`, `glossary.md`, `onboarding.md` — preserved as-is


## 0.7 Coverage and Quality Targets

### 0.7.1 Documentation Coverage Metrics

The user's instruction is to "explain the features" and "integration the expectation" with a clear development/testing split. The coverage targets below are calibrated to the scope established in 0.3 and 0.5: synthesis docs that close the architectural-overview, integration, development, and testing gaps without duplicating the existing 113 detailed reference files in `doc/api/` and `doc/contributing/`.

**Current Coverage Analysis (Pre-Task Baseline):**

| Coverage Dimension | Existing Coverage | Source of Measurement |
|---|---|---|
| Public modules with API reference | 54 of 54 (100%) | `lib/*.js` cross-checked against `doc/api/*.md` |
| Public APIs with stability index | High (per `doc/api/documentation.md` policy) | Stability levels declared throughout `doc/api/` |
| Native subdirectories with C++ orientation | 1 of 9 (`src/README.md` covers `src/`; subdirectories not individually documented) | `find src/ -name README.md` |
| Contributor workflows with detailed docs | 45 (extensive coverage) | `doc/contributing/*.md` count |
| **Architecture overview / feature index** | **0 (gap)** | No `doc/architecture/` directory exists |
| **Integration narrative (3 personas)** | **Fragmented** | `doc/api/n-api.md`, `embedding.md`, `addons.md` exist independently with no synthesis layer |
| **Development workflow index** | **0 (gap)** | No top-level dev overview; 45 disparate files |
| **Testing workflow index** | **0 (gap)** | Only `writing-tests.md`; coverage/lint/CI scattered |
| **Repository root orientation** | **Placeholder only** | `README.md` is 60 bytes |

**Target Coverage (Post-Task):**

| Target | Metric | Justification |
|---|---|---|
| Architecture overview | 1 file (`doc/architecture/overview.md`) | Single layered-architecture document covering all 5 layers |
| Feature catalog | 1 file (`doc/architecture/features.md`) covering 13 of 13 features (100%) | Complete F-001 — F-013 catalog |
| Per-feature deep-dives | 11 files (one per feature where deep-dive adds value: F-001, F-002, F-003, F-004, F-005, F-006, F-007, F-009, F-012, F-013, plus integration.md) | F-008 covered by `testing-overview.md`; F-010 and F-011 cross-linked from `features.md` |
| Integration guide | 1 file covering 3 personas (application authors, addon authors, embedders) and 5 external integrations | Closes the synthesis gap |
| Dependency matrix | 1 file covering 13+ bundled dependencies | Closes the dependency-narrative gap |
| Development workflow index | 1 file linking 44 of 45 contributing docs (excludes `writing-tests.md`) | Distinguishes dev from testing per user requirement |
| Testing workflow index | 1 file linking `writing-tests.md`, `static-analysis.md`, `doc/api/test.md`, `doc/api/assert.md`, plus all CI/coverage/benchmark/lint topics | Distinguishes testing from dev per user requirement |
| Root README orientation | 1 file (UPDATE) covering full doc map | Replaces placeholder |

**Coverage Gaps to Address:**

| Module/Topic | Current Coverage | Target Coverage |
|---|---|---|
| Layered architecture (5 layers) | 0% (only in tech spec) | 100% (overview + per-layer treatment in `doc/architecture/`) |
| Feature catalog as cross-reference | 0% | 100% (13 of 13 features in `features.md`) |
| Bundled dependency integration narrative | 0% (split across `BUILDING.md`, `common.gypi`, `node.gyp`) | 100% (`dependencies.md`) |
| Development vs testing distinction | 0% (no top-level index) | 100% (two parallel overview docs) |
| Permission Model cross-cutting documentation | Per-API only (`doc/api/permissions.md`) | Cross-cutting view (`permission-model.md`) |
| TypeScript runtime architecture | API-level only (`doc/api/typescript.md`) | Runtime-architecture view (`typescript.md`) |

**Focus Areas:**

- Public APIs (54 modules) — already 100% covered in `doc/api/`; the new docs provide architectural context, not API replication
- User workflows (build, test, debug, contribute) — split between development and testing overviews
- Error handling — already covered comprehensively in `doc/api/errors.md` (no new documentation required)
- Dependency integration — addressed in `dependencies.md`

### 0.7.2 Documentation Quality Criteria

**Completeness Requirements:**

- Every architecture document MUST include: title, optional stability/scope banner, introduction, body sections, references with file paths
- Every per-feature deep-dive MUST cite the specific public modules, internal modules, native bindings, and bundled dependencies for that feature
- Every diagram MUST be embedded as inline Mermaid (no external image files)
- Every cross-reference to existing `doc/api/*.md` files MUST be a working relative link
- The development overview MUST enumerate every relevant `doc/contributing/*.md` file with a one-line summary and link
- The testing overview MUST enumerate every relevant test category, lint tool, CI workflow, and coverage tool with explicit commands

**Accuracy Validation:**

| Validation | Method |
|---|---|
| Source paths exist in repository | Verified during authoring against `Vs_Repo-main/` extraction |
| File counts match repository | Cross-checked against tech-spec §1, §2, §3, §5, §6 (which were generated from the same repo) |
| Build commands are valid | All commands verified present in `Makefile` |
| Lint commands are valid | All commands verified present in `Makefile` and `eslint.config.mjs` |
| Mermaid syntax is valid | Diagrams reuse syntax already validated in tech-spec sections |
| Markdown linting passes | `make lint-md` will be run as the validation step (per `doc/contributing/writing-docs.md`) |
| Doc-kit rendering passes | `make test-doc -j` will be run as the validation step |

**Clarity Standards:**

- Technical accuracy with accessible language: terminology aligned with `Vs_Repo-main/glossary.md` and `doc/api/documentation.md`
- Progressive disclosure: each architecture document opens with a one-paragraph orientation before drilling into details; each feature deep-dive opens with a high-level summary before listing source paths
- Consistent terminology: "the runtime", "the binary", "Node.js v26", "the public API surface", "internal modules", "native bindings", "bundled dependencies" — used consistently across all new documents

**Maintainability:**

- Source citations: every architectural claim cites a specific file or directory (e.g., `lib/http.js`, `src/crypto/`, `common.gypi`)
- Update vectors: each new document explicitly links to the corresponding `doc/api/*.md` and `doc/contributing/*.md` files so that future code changes that update those reference docs implicitly affect the synthesis layer
- Template-based: all new documents use the same heading hierarchy and section ordering for predictability

### 0.7.3 Example and Diagram Requirements

**Minimum Examples per Document:**

| Document Type | Minimum Code/Command Examples |
|---|---|
| Architecture overview | At least 1 layered diagram |
| Feature catalog | At least 1 feature-tier diagram |
| Per-feature deep-dive | At least 1 source-path table; diagrams optional except for runtime, networking, and permission-model |
| Integration guide | At least 1 integration-points diagram |
| Dependency matrix | At least 1 dependency-integration diagram |
| Development overview | At least 1 build-pipeline diagram; concrete `make`/`vcbuild` invocations as inline shell blocks |
| Testing overview | At least 1 test-runner sequence diagram and 1 CI-workflow diagram; concrete `make test*` and `tools/test.py` invocations as inline shell blocks |

**Diagram Types Required:**

- **Flowcharts** for layered architecture, build pipelines, dependency integration
- **Sequence diagrams** for request flows, test execution, permission enforcement
- **Tier/category groupings** for feature catalogs and CI workflow organization

**Code Example Testing:**

- All code examples in new documents are either:
  - Inline shell commands corresponding to existing `Makefile` targets (verified to exist in `Makefile`)
  - References to existing samples in `doc/api/*.md` (which are already validated by `make test-doc`)
- No newly authored executable JavaScript samples will be introduced; the new documents are conceptual, with all executable detail referenced from existing docs

**Visual Content Freshness Policy:**

- All new documents will note their relationship to `Vs_Repo-main/` as the upstream source — the documents are accurate as of Node.js v26.0.0-pre (`NODE_VERSION_IS_RELEASE 0`)
- A short "Update policy" footer in each new architecture document will indicate that it reflects the layered architecture and feature inventory captured in the codebase at the time of writing, with explicit version references (e.g., `NODE_MAJOR_VERSION 26`, `NODE_MODULE_VERSION 144`)


## 0.8 Scope Boundaries and Execution Parameters

### 0.8.1 Exhaustively In Scope

**New Documentation Files:**

- `Vs_Repo-main/doc/architecture/README.md` (CREATE)
- `Vs_Repo-main/doc/architecture/overview.md` (CREATE)
- `Vs_Repo-main/doc/architecture/features.md` (CREATE)
- `Vs_Repo-main/doc/architecture/runtime.md` (CREATE)
- `Vs_Repo-main/doc/architecture/networking.md` (CREATE)
- `Vs_Repo-main/doc/architecture/file-system.md` (CREATE)
- `Vs_Repo-main/doc/architecture/cryptography.md` (CREATE)
- `Vs_Repo-main/doc/architecture/concurrency.md` (CREATE)
- `Vs_Repo-main/doc/architecture/streams.md` (CREATE)
- `Vs_Repo-main/doc/architecture/diagnostics.md` (CREATE)
- `Vs_Repo-main/doc/architecture/typescript.md` (CREATE)
- `Vs_Repo-main/doc/architecture/permission-model.md` (CREATE)
- `Vs_Repo-main/doc/architecture/web-standards.md` (CREATE)
- `Vs_Repo-main/doc/architecture/integration.md` (CREATE)
- `Vs_Repo-main/doc/architecture/dependencies.md` (CREATE)
- `Vs_Repo-main/doc/contributing/development-overview.md` (CREATE)
- `Vs_Repo-main/doc/contributing/testing-overview.md` (CREATE)

**Documentation File Updates:**

- `README.md` (root) — UPDATE
- `Vs_Repo-main/doc/api/index.md` — UPDATE (insert architecture and contributing entries)
- `Vs_Repo-main/doc/api/n-api.md` — UPDATE (add "See also" banner)
- `Vs_Repo-main/doc/api/embedding.md` — UPDATE (add "See also" banner)

**Pattern-Based Inclusion (informative — confirms wildcard reach):**

- `Vs_Repo-main/doc/architecture/**/*.md` — all newly created architecture docs
- `Vs_Repo-main/doc/contributing/development-overview.md` and `testing-overview.md` — only these two new files in `doc/contributing/`; existing 45 files are NOT modified

**Documentation Configuration:**

- No documentation configuration changes required (see 0.5.4)

**Documentation Assets:**

- No new images, screenshots, or asset files
- All diagrams are inline Mermaid

**Documentation Generation:**

- No build-script changes required; existing `make doc-only`, `make docserve`, `make lint-md`, `make test-doc -j` targets cover the new files automatically

### 0.8.2 Explicitly Out of Scope

**Source Code Modifications — Strictly Excluded:**

The user's implementation rule "Document code" is interpreted as **document the existing code without modifying it**. The following are explicitly out of scope:

- Any modification to files under `Vs_Repo-main/lib/` (54 public modules, 80+ internal module groups)
- Any modification to files under `Vs_Repo-main/src/` (453 C++ source files)
- Any modification to files under `Vs_Repo-main/deps/` (bundled dependencies — V8, libuv, OpenSSL, etc.)
- Any modification to files under `Vs_Repo-main/test/` (11,060 test files)
- Any modification to files under `Vs_Repo-main/benchmark/` (591 benchmark files)
- Any modification to files under `Vs_Repo-main/tools/` (build/lint/release/dep-update tooling)
- Any modification to `Makefile`, `vcbuild.bat`, `BSDmakefile`, `BUILD.gn`, `node.gyp`, `node.gypi`, `common.gypi`, `configure`, `configure.py`, `android-configure`, `android_configure.py`
- Any modification to lint configurations (`eslint.config.mjs`, `.clang-format`, `.cpplint`, `pyproject.toml`, `.yamllint.yaml`)
- Any modification to coverage configuration (`.nycrc`, `codecov.yml`)
- Any modification to CI workflows (`.github/workflows/*.yml`)
- Any modification to `tsconfig.json`, `package.json`, `package-lock.json`, `shell.nix`, `unofficial.gni`, `node.gni`
- Adding inline JSDoc to source files
- Adding inline doxygen comments to C++ source files

**Test File Modifications — Strictly Excluded:**

- No new tests, no test deletions, no test refactoring
- No modifications to fixture files in `test/fixtures/`
- No modifications to benchmark files in `benchmark/`

**Feature Additions, Refactoring, Behavior Changes — Strictly Excluded:**

- No new APIs, no API deprecations beyond what is already in `doc/api/deprecations.md`
- No refactoring of any source code
- No changes to module exports
- No changes to dependency versions

**Deployment Configuration Changes — Strictly Excluded (with one carve-out):**

- No changes to deployment configurations
- No changes to `tools/macos-firewall.sh`, install scripts, or release tooling
- The doc deployment path (`doc-upload` Makefile target) is unchanged — the new docs render through the existing pipeline

**Existing Documentation NOT to be Modified:**

- All 65 of 68 `doc/api/*.md` files except `index.md`, `n-api.md`, `embedding.md` (the three explicitly listed for UPDATE)
- All 45 `doc/contributing/*.md` files (the development and testing overviews link OUT to them; they are not edited)
- `Vs_Repo-main/README.md` (the upstream Node.js README — preserved as-is)
- `Vs_Repo-main/BUILDING.md`, `CONTRIBUTING.md`, `CHANGELOG.md`, `CODE_OF_CONDUCT.md`, `GOVERNANCE.md`, `SECURITY.md`, `LICENSE`, `glossary.md`, `onboarding.md`
- `Vs_Repo-main/doc/README.md` (the documentation style guide)
- `Vs_Repo-main/src/README.md`, `Vs_Repo-main/test/README.md`, `Vs_Repo-main/tools/doc/README.md`

**Items Excluded by User Instructions:**

The user's verbatim instruction is to "document the code to explain the features, integration the expectation for the code" with the rule "Document code". By that rule, all code-modification activities are out of scope.

### 0.8.3 Execution Parameters

#### Documentation-Specific Instructions

| Instruction | Exact Command |
|---|---|
| Documentation build (after compiling `node`) | `make doc` |
| Documentation build (using existing `node` binary) | `NODE=/path/to/node make doc-only` |
| Local preview server | `make docserve` (binds to `127.0.0.1:8000` and serves `out/doc/api/`) |
| Open built docs in default browser | `make docopen` |
| Clean documentation outputs | `make docclean` |
| Markdown linting | `make lint-md` |
| Doc-kit rendering validation | `make test-doc -j` |
| Doc-kit CI validation | `make test-doc-ci` |
| Default markdown format | GitHub-flavored Markdown with Mermaid fenced blocks |
| Citation requirement | Every section MUST cite source files using inline backticked paths |
| Style guide | `Vs_Repo-main/doc/README.md` (Node.js documentation style guide) |
| Documentation validation pipeline | `make lint-md && make test-doc -j` |
| File naming convention | `lowercase-with-dashes.md` (per `doc/README.md`) |
| Line wrap | 120 characters per line (per `doc/README.md` and `.editorconfig`) |
| Spelling | US English (per `doc/README.md`) |
| Stability index policy | Reference `doc/api/documentation.md` where applicable |

**Cross-Platform Considerations:**

- All `make` invocations apply on Unix and macOS; the equivalents for Windows are documented in `BUILDING.md` and use `vcbuild.bat`. The new documents will reference both.
- Documentation rendering is identical on all platforms — `@node-core/doc-kit` is platform-agnostic Node.js code.

**Validation Workflow:**

For each new or updated file the Blitzy platform applies, the validation pipeline is:

1. Run `make lint-md` — verifies markdown lint compliance against `eslint.config.mjs` and `@eslint/markdown` rules
2. Run `make test-doc -j` — verifies doc-kit can parse and render the file (ensures Mermaid syntax, link validity, YAML metadata block syntax)
3. Optionally run `make docserve` — local visual inspection at `http://127.0.0.1:8000`

These validations are documented in `Vs_Repo-main/doc/contributing/writing-docs.md` and represent the canonical doc-validation workflow established by the Node.js project.


## 0.9 Rules for Documentation

### 0.9.1 User-Specified Rules (Verbatim Capture)

The user's prompt includes one explicit implementation rule. The rule is captured here verbatim and interpreted in technical terms below.

**User-Provided Rule (Verbatim):**

> **Rule name:** Document code  
> **Rule content:** Test

**User-Provided Instruction (Verbatim):**

> "Read and analyze the code and document the code to explain the features, integration the expectation for the code. Ensure to differentiate between developement and testing activities."

### 0.9.2 Rule Interpretation and Application

**Rule: "Document code" / "Test"**

| Aspect | Interpretation |
|---|---|
| Title — "Document code" | Treated as the canonical task directive: every artifact produced MUST document the existing code without modifying it. The platform produces Markdown files; it does not edit `lib/`, `src/`, `test/`, `benchmark/`, or any source/build/CI file. |
| Content — "Test" | Treated in conjunction with the user instruction's "differentiate between development and testing activities" requirement: the testing dimension MUST receive first-class documentation treatment alongside development. Specifically, the testing-overview document MUST be peer-equivalent in depth, length, and structure to the development-overview document, and BOTH must be present. |
| Combined effect | Documentation only (no code changes); first-class testing coverage; first-class development coverage; explicit dev-vs-test separation. |

**Documentation-Specific Rules Derived from the Instruction Wording:**

- "Read and analyze the code" — The Blitzy platform MUST inspect actual files (verified by ZIP extraction of `Vs_Repo-main.zip`) and produce documentation grounded in observed file paths, file counts, and configuration values; no fabricated paths or counts. All claims trace to specific repository artifacts.
- "Document the code to explain the features" — The deliverables MUST cover all 13 features (F-001 — F-013) catalogued in §2.1 of the tech spec and confirmed by inspecting `lib/*.js`, `lib/internal/`, and `src/`.
- "Integration the expectation for the code" (interpreted as "integration expectations") — The deliverables MUST cover both inbound integration (how user code uses the runtime) and outbound integration (how the runtime uses bundled dependencies and external systems).
- "Ensure to differentiate between development and testing activities" — Two parallel overview documents MUST be produced (`development-overview.md` and `testing-overview.md`) with explicit scope statements at the top declaring what each does and does NOT cover.

**Documentation-Specific Rules Derived from the Repository Style Guide (`doc/README.md`):**

- File naming: `lowercase-with-dashes.md` (e.g., `file-system.md` not `file_system.md`)
- Line wrap: 120 characters per line
- Spelling: US English (no British spellings)
- Markdown formatting: Follow `.editorconfig` settings (UTF-8, LF, 2-space indent, trim trailing whitespace, final newline)
- Validation: Documentation changes validated using `make test-doc -j` and `make lint-md`

**Documentation-Specific Rules Derived from the API Documentation Convention (`doc/contributing/api-documentation.md`):**

- Stability index: When a feature is documented, declare its stability per `doc/api/documentation.md` (0 — Deprecated, 1 — Experimental [with sub-stages 1.0/1.1/1.2], 2 — Stable, 3 — Legacy)
- YAML metadata: API-style files use `<!-- YAML -->` HTML-comment blocks with `added:` and `changes:` keys
- Source link annotation: API-style files use HTML-comment annotations referencing the source path (e.g., `source_link=lib/<module>.js`)
- Cross-references: Use Markdown link references with descriptive labels

**Documentation-Specific Rules Derived from `doc/contributing/writing-docs.md`:**

- Build documentation locally with `make docserve` for inspection
- Lint with `make lint-md` before committing
- The new architecture and contributing overview docs are NARRATIVE (not API-reference); they may omit the strict stability/YAML/source_link conventions used in `doc/api/`

### 0.9.3 Source-of-Truth Hierarchy

When information is available from multiple sources, the Blitzy platform applies the following precedence (highest to lowest):

1. **The repository source code itself** — `Vs_Repo-main/` files (the canonical truth for file paths, counts, configuration values, version constants)
2. **Existing technical specification sections** — §1, §2, §3, §5, §6 (already validated against the codebase)
3. **Existing documentation in the repository** — `doc/api/`, `doc/contributing/`, `BUILDING.md`, `CONTRIBUTING.md`, `glossary.md`, etc.
4. **Repository style guide** — `doc/README.md`
5. **External web documentation** — only used when the repository does not contain the answer (none required for this task)

**Conflict Resolution:**

If a tech-spec section conflicts with the repository source, the repository source wins. The Blitzy platform will NOT introduce documentation that contradicts an inspected source file. All architectural claims in the new documents are stamped with the version constants from `src/node_version.h` (`NODE_MAJOR_VERSION 26`, `NODE_MODULE_VERSION 144`, `NODE_API_SUPPORTED_VERSION_MAX 10`, `NODE_VERSION_IS_RELEASE 0`).

### 0.9.4 Negative Constraints (Explicit Rules of "Do Not")

| Constraint | Rationale |
|---|---|
| Do NOT modify any file under `lib/`, `src/`, `deps/`, `test/`, `benchmark/`, `tools/` | User rule "Document code" — documentation only |
| Do NOT modify build configuration (`Makefile`, `node.gyp`, `common.gypi`, `configure.py`, `vcbuild.bat`) | User rule "Document code" — no build changes |
| Do NOT modify CI workflow YAMLs (`.github/workflows/`) | User rule "Document code" — no CI changes |
| Do NOT modify lint configurations (`eslint.config.mjs`, `.clang-format`, `.cpplint`, `pyproject.toml`, `.yamllint.yaml`) | User rule "Document code" — no tooling changes |
| Do NOT replace the upstream Node.js `Vs_Repo-main/README.md` | The upstream README is preserved as-is |
| Do NOT replace `Vs_Repo-main/doc/README.md` (style guide) | Existing style guide is the authoritative reference |
| Do NOT delete any existing `doc/api/*.md` or `doc/contributing/*.md` file | Additive task only |
| Do NOT introduce a separate documentation generator (mkdocs, Sphinx, Docusaurus) | Repository uses `@node-core/doc-kit`; that pipeline is preserved |
| Do NOT add JavaScript or shell scripts to the documentation tree | Only Markdown files are produced |
| Do NOT introduce binary assets (images, PDFs, archives) | All diagrams are inline Mermaid |
| Do NOT alter version constants or release flags | `src/node_version.h` and similar are referenced, never modified |
| Do NOT use British English spellings | Style guide mandates US English |
| Do NOT exceed 120 characters per line in any new Markdown file | Style guide mandates 120-char wrap |
| Do NOT use `under_scored_filenames.md` for new files | Style guide mandates `lowercase-with-dashes.md` |
| Do NOT duplicate content already in existing `doc/api/*.md` files | Synthesis layer cross-links rather than duplicates |
| Do NOT mix dev and test concerns in a single overview document | User instruction explicitly requires differentiation |

### 0.9.5 Positive Constraints (Explicit Rules of "Do")

| Constraint | Rationale |
|---|---|
| DO inspect actual files in `Vs_Repo-main/` before making any architectural claim | Rule "Read and analyze the code" |
| DO cite specific file paths, directory paths, and configuration keys for every claim | Style guide and tech-spec convention |
| DO use Mermaid diagrams for layered architecture, request flows, build pipelines, and CI flows | Repository convention (Mermaid-only) |
| DO use US English spellings, 120-char line wrap, `lowercase-with-dashes.md` filenames | Style guide mandate |
| DO produce parallel `development-overview.md` and `testing-overview.md` with explicit scope statements at the top of each | User instruction "differentiate between development and testing activities" |
| DO cross-link the new architecture docs to the existing `doc/api/*.md` references | Synthesis layer pattern |
| DO update the root `README.md` from its placeholder state to a navigation document | The repository root is the discoverability surface |
| DO validate every new and updated file via `make lint-md` and `make test-doc -j` | Repository validation convention |
| DO carry `NODE_MAJOR_VERSION 26`, `NODE_MODULE_VERSION 144`, `NODE_API_SUPPORTED_VERSION_MAX 10`, `NODE_VERSION_IS_RELEASE 0` as version stamps in version-relevant docs | Source-of-truth from `src/node_version.h` |
| DO use the existing `@node-core/doc-kit` v1.0.2 pipeline | Pinned in `tools/doc/package.json` |


## 0.10 References

### 0.10.1 Files and Folders Examined Across the Codebase

The following catalog enumerates every file and folder examined by the Blitzy platform during the analysis phase that produced this Agent Action Plan. The repository was extracted from `/tmp/blitzy/Lakshya_Nested_Folder/07-May-26-Br1_625df4/Vs_Repo-main.zip` to `/tmp/nodejs_extract/Vs_Repo-main/` for inspection. Within the technical-specification document, retrieved sections are listed by heading.

**Repository Root and Top-Level Files:**

- `README.md` (repository root — placeholder)
- `Vs_Repo-main/` (extracted Node.js source tree)
- `Vs_Repo-main/README.md`
- `Vs_Repo-main/BUILDING.md`
- `Vs_Repo-main/CONTRIBUTING.md`
- `Vs_Repo-main/Makefile`
- `Vs_Repo-main/.editorconfig`
- `Vs_Repo-main/eslint.config.mjs`
- `Vs_Repo-main/onboarding.md`

**Source Library Folders:**

- `Vs_Repo-main/lib/` (54 public modules including `_http_agent.js`, `_http_client.js`, `_http_common.js`, `_http_incoming.js`, `_http_outgoing.js`, `_http_server.js`, `_tls_common.js`, `_tls_wrap.js`, `assert.js`, `async_hooks.js`, `buffer.js`, `child_process.js`, `cluster.js`, `console.js`, `constants.js`, `crypto.js`, `dgram.js`, `diagnostics_channel.js`, `dns.js`, `domain.js`, `events.js`, `fs.js`, `http.js`, `http2.js`, `https.js`, `inspector.js` and 28 additional modules)
- `Vs_Repo-main/lib/internal/` (referenced as containing 80+ module groups)

**Native Source Folders:**

- `Vs_Repo-main/src/` (453 C++ source files)
- `Vs_Repo-main/src/README.md` (C++ codebase orientation)

**Test Folders:**

- `Vs_Repo-main/test/` (top-level listing showing `abort`, `addons`, `async-hooks`, `benchmark`, `cctest`, `client-proxy`, `common`, `doctool`, `embedding`, `es-module`, `fixtures`, `fuzzers`, `internet`, `js-native-api`, `known_issues`, `message`, `module-hooks`, `node-api`, `nop`, `overlapped-checker`, `parallel`, `pseudo-tty`, `pummel`, `report`, `sea`, `sequential`, `sqlite`)
- `Vs_Repo-main/test/README.md`

**Documentation Folders:**

- `Vs_Repo-main/doc/`
- `Vs_Repo-main/doc/README.md` (Node.js documentation style guide — contents inspected)
- `Vs_Repo-main/doc/api/` (68 files — full filename listing inspected)
- `Vs_Repo-main/doc/api/index.md`
- `Vs_Repo-main/doc/api/documentation.md`
- `Vs_Repo-main/doc/api/test.md` (sampled for API documentation pattern)
- `Vs_Repo-main/doc/contributing/` (45 files — full filename listing inspected)
- `Vs_Repo-main/doc/contributing/api-documentation.md`
- `Vs_Repo-main/doc/contributing/writing-docs.md`
- `Vs_Repo-main/doc/contributing/writing-tests.md`
- `Vs_Repo-main/doc/contributing/pull-requests.md`
- `Vs_Repo-main/doc/contributing/maintaining/` (sub-folder noted)

**Build and Tooling Folders:**

- `Vs_Repo-main/tools/`
- `Vs_Repo-main/tools/doc/`
- `Vs_Repo-main/tools/doc/README.md`
- `Vs_Repo-main/tools/doc/package.json` (`@node-core/doc-kit` v1.0.2)

**Tech Specification Sections Retrieved (for context, not authored):**

- `1.1 Executive Summary` — project overview, version constants, license
- `1.2 System Overview` — release landscape, version identifiers, layered architecture overview
- `1.3 Scope` — in-scope and out-of-scope catalogs, supported platforms
- `2.1 Feature Catalog` — F-001 through F-013 with source paths
- `3.1 Programming Languages` — language inventory and counts
- `3.7 Technology Stack Summary` — consolidated technology inventory
- `5.1 HIGH-LEVEL ARCHITECTURE` — 5-layer architectural model
- `6.6 Testing Strategy` — comprehensive testing strategy, CI workflows, coverage gates
- `8.10 Code Quality Toolchain` — lint and coverage tooling

**Setup Discovery:**

- Searched `find / -name ".blitzyignore"` — no `.blitzyignore` files found in the system
- Searched `/tmp/environments_files` — directory does not exist (no user-attached files)

### 0.10.2 User-Provided Attachments

| Attachment | Summary |
|---|---|
| (none) | The user attached 0 environments and 0 files. The "List of environment variables" and "List of secrets" provided by the user were both empty arrays. |

The only user-supplied artifacts beyond the prompt text are:

- **Implementation rules** — A single rule with name "Document code" and content "Test" (captured verbatim in 0.9.1)
- **Setup instructions** — None provided
- **Environment variables** — None provided (empty list `[]`)
- **Secrets** — None provided (empty list `[]`)

### 0.10.3 Figma Frames and URLs

| Frame Name | URL | Description |
|---|---|---|
| (none) | (none) | No Figma frames, screens, or design references were provided by the user. The Node.js runtime has no UI surface (per tech-spec §7) and the documentation task is purely textual / diagrammatic. |

### 0.10.4 External References Cited Within the Repository

The following external references appear in the repository's existing documentation and are noted here for traceability. The Blitzy platform did not need to fetch these externally — the repository inspection sufficed for this task.

| Reference | Source File in Repo |
|---|---|
| OpenJS Foundation governance | `Vs_Repo-main/GOVERNANCE.md` (referenced in tech spec §1.1.3) |
| HackerOne bug bounty program | `Vs_Repo-main/SECURITY.md` |
| `@node-core/doc-kit` upstream project | `Vs_Repo-main/tools/doc/README.md` and `Vs_Repo-main/doc/contributing/api-documentation.md` |
| Node.js release schedule | `Vs_Repo-main/CHANGELOG.md` (referenced in tech spec §1.2.1) |
| WHATWG Fetch / Streams standards | Referenced in tech spec §2.1.4 (F-013) |
| W3C Web Crypto standard | Referenced in tech spec §2.1.4 (F-013) |
| Stack Overflow Developer Survey | Referenced in tech spec §1.1.4 (general context, not actionable) |

### 0.10.5 Repository Inspection Path Summary

| Inspection | Result |
|---|---|
| Repository root via `get_source_folder_contents("")` | Confirmed minimal root with single child `README.md`; actual codebase packaged inside `Vs_Repo-main.zip` |
| ZIP extraction via `python3 -c "import zipfile..."` | Successfully extracted `Vs_Repo-main.zip` to `/tmp/nodejs_extract/Vs_Repo-main/` |
| Directory enumeration via `ls` and `find` | Verified file counts and directory structure across `lib/`, `src/`, `test/`, `doc/`, `tools/` |
| Targeted file inspection via `cat` and `head` | Verified contents of `Makefile` (doc/test/lint targets), `tools/doc/package.json` (doc-kit version), `doc/README.md` (style guide), `doc/api/index.md`, `doc/api/documentation.md`, `doc/api/test.md`, `doc/contributing/api-documentation.md`, `doc/contributing/writing-docs.md`, `doc/contributing/writing-tests.md`, `BUILDING.md` (build/test/doc commands), `eslint.config.mjs`, `.editorconfig`, root `README.md` |
| `.blitzyignore` enforcement | Verified that no `.blitzyignore` file exists in the repository or environment; no inspection exclusions were required |


