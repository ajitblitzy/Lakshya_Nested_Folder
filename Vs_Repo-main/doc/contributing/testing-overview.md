# Testing overview

> Scope: This document indexes the **testing** workflows for the Node.js project. It complements the
> [Development overview](development-overview.md), which covers the **development** workflows
> (building, configuring, debugging, code style, dependency maintenance, releases). Concerns that
> are exclusively about producing or modifying source code, build configuration, or release
> artifacts are intentionally deferred to the development overview. Test execution, coverage,
> linting, benchmarks, CI gates, and security scanning are covered here.

This page synthesizes every testing-related artifact in the repository into one navigable index.
The existing detailed contributor guides (e.g., [Writing tests](writing-tests.md),
[Static analysis](static-analysis.md),
[Writing and running benchmarks](writing-and-running-benchmarks.md)) are preserved as authoritative
deep-dives. This document links out to those guides rather than duplicating their content.

The version of Node.js documented here is **v26.0.0-pre** (`NODE_MAJOR_VERSION 26`,
`NODE_MODULE_VERSION 144`, `NODE_API_SUPPORTED_VERSION_MAX 10`, `NODE_VERSION_IS_RELEASE 0`), as
recorded in `src/node_version.h`.

## Table of contents

* [Test corpus](#test-corpus)
* [Running tests](#running-tests)
* [Built-in test runner](#built-in-test-runner)
* [Benchmarks](#benchmarks)
* [Linting](#linting)
* [Coverage](#coverage)
* [CI workflows](#ci-workflows)
* [Security testing](#security-testing)
* [Web Platform Tests](#web-platform-tests)
* [Cross-references](#cross-references)

## Test corpus

The Node.js test suite lives under `test/` and consists of approximately 11,060 entries (test
files, fixtures, and supporting assets) distributed across 37 top-level category directories. The
largest categories are:

| Category              | Approximate entry count | Purpose                                       |
| --------------------- | ----------------------- | --------------------------------------------- |
| `test/parallel/`      | 4,102                   | Parallel-safe functional tests run by default |
| `test/fixtures/`      | 5,006                   | Static fixtures shared across tests           |
| `test/sequential/`    | 121                     | Tests requiring serialized execution          |
| `test/es-module/`     | 226                     | ECMAScript module loader tests                |
| `test/addons/`        | 226                     | Native addon integration tests                |
| `test/js-native-api/` | 176                     | Node-API tests written in C and JavaScript    |
| `test/node-api/`      | 131                     | Internal Node-API tests                       |
| `test/module-hooks/`  | 113                     | Module customization hooks tests              |
| `test/test-runner/`   | 109                     | Built-in `node --test` runner tests           |
| `test/async-hooks/`   | 101                     | `async_hooks` API tests                       |
| `test/wasi/`          | 87                      | WebAssembly System Interface tests            |
| `test/pseudo-tty/`    | 82                      | Tests that require a pseudo-TTY               |
| `test/wpt/`           | 61                      | Web Platform Tests harness                    |
| `test/sea/`           | 39                      | Single executable application tests           |
| `test/cctest/`        | 36                      | C++ unit tests linked via the `cctest` binary |

The remaining categories under `test/` (`abort/`, `benchmark/`, `client-proxy/`, `common/`,
`doctool/`, `embedding/`, `fuzzers/`, `internet/`, `known_issues/`, `message/`, `nop/`,
`overlapped-checker/`, `pummel/`, `report/`, `sqlite/`, `system-ca/`, `test426/`, `testpy/`,
`tick-processor/`, `tools/`, `v8-updates/`, `wasm-allocation/`) cover specialized scenarios such
as crash recovery, internet-dependent integration, fuzzing harnesses, and tooling self-tests.

For a comprehensive overview of the directory layout, see the orientation file at `test/README.md`
and the dedicated guide [Writing tests](writing-tests.md).

## Running tests

The repository's `Makefile` exposes the canonical entry points for running tests. The most
commonly used targets are:

```bash
# Build node and run the default test suite plus build the docs
make test

# Build node and run the default tests without rebuilding docs
make test-only

# Run tests with coverage instrumentation enabled
make test-cov

# Run only the C++ unit tests
make cctest

# Run JavaScript tests, excluding cctest
make jstest

# Run the documentation build, lint, and verification
make test-doc -j

# Run the doc tests in CI mode
make test-doc-ci

# Run tests against a debug build
make test-debug

# Run all tests against both Debug and Release builds
make test-all

# Run only the Web Platform Tests
make test-wpt

# Run only internet-dependent tests
make test-internet

# Run the known-issues smoke tests
make test-known-issues
```

For Windows, the equivalent targets are exposed by `vcbuild.bat`:

```text
vcbuild test
vcbuild test-doc
```

For finer-grained control over which tests run, the repository ships `tools/test.py` — a Python
test driver inherited from V8. Common invocation patterns are:

```bash
# Run a single test file
tools/test.py test/parallel/test-stream2-transform.js

# Run every test that matches a subsystem (resolves under test/parallel/, test/sequential/, ...)
tools/test.py child-process

# Run every test in a specific directory
tools/test.py test/message

# Use a glob to filter tests
tools/test.py "test/parallel/test-stream-*"
tools/test.py "test/*/test-inspector-*"

# Print the full set of CLI options
tools/test.py --help
```

`tools/test.py` discovers tests by walking the `test/` tree, executes each test against the freshly
built `out/Release/node` binary (or `out/Debug/node` when `--mode=debug` is passed), and reports
results in the project's TAP-style format. The test runner is parallelized by default and prints a
final summary listing passing, failing, skipped, and timed-out tests.

For background on how new tests should be written, see [Writing tests](writing-tests.md).

## Built-in test runner

In addition to the project's `tools/test.py` harness, Node.js ships a **built-in test runner** that
is exposed to all users of the runtime. It is documented as feature **F-008** and is the canonical
test-authoring API for application code as well as a subset of internal tests under
`test/test-runner/` (109 files).

<!--lint disable fenced-code-flag-->

```text
node --test  ->  test runner  ->  test discovery
                       |                |
                       |          (1) Walk specified file globs
                       |                |
                       |     <-- Test plan (suites + tests)
                       |
                       v
                  test execution
                       |
                       | (2) Run each test in isolation
                       |
                       v
              coverage harness  (optional)
                       |
                       | (3) --experimental-test-coverage
                       |     instrumentation
                       v
                  TAP reporter
                       |
                       | (4) TAP output  (or alternate reporter)
                       v
                  node --test (caller)
```

<!--lint enable fenced-code-flag-->

The runner is exposed via `node:test` (covered in [`../api/test.md`](../api/test.md)) with
assertions delegated to `node:assert` (covered in [`../api/assert.md`](../api/assert.md)). Common
invocations:

```bash
# Run every test file matching the default discovery pattern
node --test

# Run a specific file or glob
node --test test/foo.js
node --test "test/**/*.test.js"

# Enable coverage and emit the LCOV reporter alongside TAP
node --test --experimental-test-coverage --test-reporter=tap --test-reporter=lcov
```

For details on writing tests using the built-in runner, see [`../api/test.md`](../api/test.md).
For the assertion library, see [`../api/assert.md`](../api/assert.md).

## Benchmarks

Performance benchmarks live under `benchmark/`. The directory contains 591 entries spanning all
subsystems (HTTP, streams, buffers, crypto, child processes, file system, etc.). The Make targets
for running benchmarks are documented in detail in
[Writing and running benchmarks](writing-and-running-benchmarks.md).

The two foundational entry points are:

```bash
# Build the addons used by benchmark/napi/
make bench-addons-build

# Remove benchmark addon artifacts
make bench-addons-clean
```

Individual benchmarks are typically run as:

```bash
# Run an entire subsystem
node benchmark/run.js <subsystem>

# Run a single benchmark file
node benchmark/<subsystem>/<file>.js
```

For comparing two builds (e.g., a baseline against a feature branch), the
[Writing and running benchmarks](writing-and-running-benchmarks.md) guide documents the
`benchmark/compare.js` and `benchmark/compare.R` toolchain.

## Linting

The project lints across four languages. Configuration is co-located with the rules of each tool:

* **JavaScript and Markdown** — ESLint with the `@eslint/markdown` plugin. The root configuration
  lives in `eslint.config.mjs`, with per-tree partials at `doc/eslint.config_partial.mjs`,
  `test/eslint.config_partial.mjs`, and `benchmark/eslint.config_partial.mjs`. Run with
  `make lint-js`, `make lint-md`, or auto-fix via `make lint-js-fix`.
* **C++** — `clang-format` (formatting) and `cpplint.py` (linting). Configuration lives in
  `.clang-format` and `.cpplint`. The `.cpplint` file pins `set noparent` and `linelength=80`,
  and disables filters such as `-build/c++17`, `-build/include_alpha`, and `-legal/copyright`.
  Run with `make lint-cpp` or auto-format via `make format-cpp`.
* **Python** — `ruff`. Configuration lives in `pyproject.toml` (`target-version = "py310"`,
  `line-length = 172`). Run with `make lint-py`, or auto-fix via `make lint-py-fix` and
  `make lint-py-fix-unsafe`.
* **YAML** — `yamllint`. Configuration lives in `.yamllint.yaml`. Run with `make lint-yaml`.

Run the entire lint suite at once with:

```bash
make lint
```

Or in CI mode (the variant invoked by `linters.yml`):

```bash
make lint-ci
```

For Markdown-only validation (the canonical pre-commit doc check):

```bash
make lint-md
```

For deep-dive coverage of the C/C++ static analysis story, see
[Static analysis](static-analysis.md).

## Coverage

Code coverage is collected via `nyc` (JavaScript / TypeScript) and `gcov` (C++), aggregated by the
project's coverage Make targets, and uploaded to Codecov for visualization.

Coverage thresholds are pinned in `.nycrc` at the repository root:

| Metric     | Threshold |
| ---------- | --------- |
| Lines      | `95`      |
| Branches   | `93`      |
| Statements | `95`      |

The `.nycrc` file also instructs `nyc` to exclude `coverage/**`, `test/**`, `tools/**`,
`benchmark/**`, and `deps/**` from coverage measurement, and writes intermediate output to
`coverage/tmp/`. The reporters configured by default are `html`, `text`, and `cobertura`.

Codecov configuration is captured in `codecov.yml`:

* `layout: diff, files` for inline pull-request comments
* `require_changes: true` (no comment if coverage is unchanged)
* `after_n_builds: 2` to wait for both the Linux and Linux-without-intl coverage runs before
  posting (Windows coverage is currently disabled per the repository note)
* `project: off`, `patch: off` — coverage diff is informational, not blocking

Run the coverage suite locally with:

```bash
# Build with coverage instrumentation, run tests, and produce reports
make coverage

# Just the JavaScript portion
make coverage-run-js

# Clean coverage artifacts before a fresh run
make coverage-clean
```

Coverage artifacts are produced under `coverage/` and `out/Release/obj.target/`. The HTML report
is written to `coverage/index.html`.

## CI workflows

Continuous integration runs are orchestrated by GitHub Actions. The repository ships **37**
workflow files under `.github/workflows/`. They fall into the following categories:

<!--lint disable fenced-code-flag-->

```text
+--------------------+        +-----------------------------------------+
| Triggers           |        | Testing workflows                       |
|  push to main      | -----> |   test-linux.yml, test-macos.yml,       |
|  pull_request      | -----> |   test-shared.yml, test-internet.yml,   |
|                    |        |   daily.yml                             |
|                    |        +-----------------------------------------+
|                    |        +-----------------------------------------+
|                    |        | Coverage workflows                      |
|  push to main      | -----> |   coverage-linux.yml,                   |
|  pull_request      | -----> |   coverage-linux-without-intl.yml,      |
|                    |        |   coverage-windows.yml                  |
|                    |        +-----------------------------------------+
|                    |        +-----------------------------------------+
|                    |        | Linting workflows                       |
|  push to main      | -----> |   linters.yml, commit-lint.yml,         |
|  pull_request      | -----> |   lint-release-proposal.yml,            |
|                    |        |   license-builder.yml                   |
|                    |        +-----------------------------------------+
|                    |        +-----------------------------------------+
|                    |        | Security workflows                      |
|  schedule (cron)   | -----> |   codeql.yml, scorecard.yml             |
|                    |        +-----------------------------------------+
|                    |        +-----------------------------------------+
|                    |        | Maintenance workflows                   |
|                    |        |   update-v8.yml, update-openssl.yml,    |
|                    |        |   update-wpt.yml, daily-wpt-fyi.yml,    |
|                    |        |   timezone-update.yml, tools.yml,       |
|  schedule (cron)   | -----> |   close-stale-*.yml,                    |
|                    |        |   find-inactive-*.yml,                  |
|                    |        |   label-*.yml, notify-on-*.yml,         |
|                    |        |   comment-labeled.yml,                  |
|                    |        |   commit-queue.yml,                     |
|                    |        |   auto-start-ci.yml                     |
|                    |        +-----------------------------------------+
|                    |        +-----------------------------------------+
|                    |        | Release workflows                       |
|  workflow_dispatch | -----> |   create-release-proposal.yml,          |
|  release           | -----> |   major-release.yml, post-release.yml,  |
|                    |        |   build-tarball.yml, doc.yml            |
+--------------------+        +-----------------------------------------+
```

<!--lint enable fenced-code-flag-->

Brief summary of each category:

* **Testing workflows** — execute the test suite on Linux, macOS, and shared runners, with daily
  long-running variants (`daily.yml`) and an internet-dependent suite (`test-internet.yml`).
* **Coverage workflows** — run `make coverage` on Linux (with and without ICU) and Windows;
  results are uploaded to Codecov per the `codecov.yml` configuration.
* **Security workflows** — run **CodeQL** SAST daily (`codeql.yml`) and the OSSF **Scorecard**
  project-health check (`scorecard.yml`).
* **Linting workflows** — run `make lint-ci` (`linters.yml`), validate commit messages
  (`commit-lint.yml`), validate release-proposal pull requests (`lint-release-proposal.yml`), and
  rebuild the third-party license bundle (`license-builder.yml`).
* **Release workflows** — produce release proposals (`create-release-proposal.yml`,
  `major-release.yml`), run post-release validation (`post-release.yml`), build source tarballs
  (`build-tarball.yml`), and publish API docs (`doc.yml`).
* **Maintenance workflows** — automate dependency updates (V8, OpenSSL, WPT, time zones), prune
  stale issues and pull requests, detect inactive collaborators, label PRs and flaky tests, and
  run the commit queue (`commit-queue.yml`, `auto-start-ci.yml`).

The full inventory is browseable at `.github/workflows/`, and the human-friendly documentation for
the build/CI infrastructure is in [Pull requests](pull-requests.md) and
[Collaborator guide](collaborator-guide.md).

## Security testing

Security testing is layered:

* **CodeQL** — static analysis driven by `.github/workflows/codeql.yml`. It runs daily and on
  pushes to security-relevant branches, scanning JavaScript, C/C++, and Python sources for known
  vulnerability patterns. Results are surfaced through the GitHub security tab.
* **OSSF Scorecard** — supply-chain security scoring via `.github/workflows/scorecard.yml`. It
  runs on a weekly schedule and emits a Scorecard report consumable by downstream auditors.
* **Coverity** — manual scanning of C/C++ sources documented in
  [Static analysis](static-analysis.md). Authorized collaborators can sign up for emails on new
  defects.
* **HackerOne** — coordinated disclosure for security vulnerabilities. Process and severity matrix
  are described in [`../../SECURITY.md`](../../SECURITY.md) and
  [Security release process](security-release-process.md).
* **Security model strategy** — long-term threat-model evolution is captured in
  [Security model strategy](security-model-strategy.md).

For the security stewarding lifecycle, see
[Security steward on/off-boarding](security-steward-on-off-boarding.md).

## Web Platform Tests

The Web Platform Tests (WPT) suite under `test/wpt/` validates Node.js compliance with
web-platform APIs (`fetch`, Streams, Web Crypto, URL, etc.). Two workflows back the WPT
integration:

* `daily-wpt-fyi.yml` — runs the WPT suite daily and uploads results to wpt.fyi for cross-engine
  comparison.
* `update-wpt.yml` — automatically vendors the latest upstream WPT manifest and tests into
  `test/wpt/`, opening a pull request with the changes.

To run WPT locally:

```bash
make test-wpt
```

The suite is also covered by `tools/test.py wpt` (or `tools/test.py "test/wpt/test-*"`).

## Cross-references

The following documents extend the topics summarized here:

* Detailed guide for authoring tests: [Writing tests](writing-tests.md)
* Benchmark authoring and execution:
  [Writing and running benchmarks](writing-and-running-benchmarks.md)
* C/C++ static analysis (Coverity): [Static analysis](static-analysis.md)
* Built-in test runner reference: [`../api/test.md`](../api/test.md)
* Built-in assertion library: [`../api/assert.md`](../api/assert.md)
* Build and platform requirements: [`../../BUILDING.md`](../../BUILDING.md)
* Project contribution policy: [`../../CONTRIBUTING.md`](../../CONTRIBUTING.md)
* Project security policy: [`../../SECURITY.md`](../../SECURITY.md)
* Companion development workflow index: [Development overview](development-overview.md)
* Pull-request workflow that gates CI runs: [Pull requests](pull-requests.md)
* Collaborator-side handling of tests and CI: [Collaborator guide](collaborator-guide.md)
* Commit-queue mechanics: [Commit queue](commit-queue.md)
* Per-dependency maintainers (e.g., V8, OpenSSL): [`maintaining/`](maintaining/)
