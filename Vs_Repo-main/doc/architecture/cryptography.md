# Cryptography

> Feature: F-004 — Cryptography (`node:crypto`, `node:tls`, `node:webcrypto`)

This page deep-dives into the cryptography stack of Node.js v26.0.0-pre. It covers OpenSSL backing,
the difference between the traditional `node:crypto` API and the Web Crypto API
(`globalThis.crypto.subtle`), FIPS-compliant build mode, and the `src/crypto/` native layout (62
files). The version stamps are `NODE_MAJOR_VERSION 26`, `NODE_MODULE_VERSION 144`.

For an overview of the runtime layers, see [Architecture overview](overview.md). For the bundled
OpenSSL details and build flags, see [Dependencies](dependencies.md). For the network use of TLS,
see [Networking](networking.md). For the Web Standards alignment, see [Web standards](web-standards.md).

## OpenSSL backing

All cryptography in the runtime is OpenSSL-backed. The build toggle that enables this is declared
in `node.gyp`:

```text
'node_use_openssl%': 'true',
```

The bundled OpenSSL lives at `deps/openssl/`. To link a system OpenSSL instead, pass
`./configure --shared-openssl` (which sets `node_shared_openssl: true`). Both bundled and system
configurations are covered in [`../../BUILDING.md`](../../BUILDING.md).

The native bindings between JavaScript and OpenSSL are in `src/crypto/` (62 files). Their structural
breakdown is grouped below by algorithm family:

| Subject                         | Source                                                                          |
| ------------------------------- | ------------------------------------------------------------------------------- |
| Symmetric ciphers (AES)         | `src/crypto/crypto_aes.cc`, `src/crypto/crypto_aes.h`                           |
| Symmetric ciphers (ChaCha20)    | `src/crypto/crypto_chacha20_poly1305.cc`, `src/crypto/crypto_chacha20_poly1305.h` |
| Symmetric cipher streams        | `src/crypto/crypto_cipher.cc`, `src/crypto/crypto_cipher.h`                     |
| Hashing (SHA family, BLAKE2)    | `src/crypto/crypto_hash.cc`, `src/crypto/crypto_hash.h`                         |
| Key derivation (PBKDF2)         | `src/crypto/crypto_pbkdf2.cc`, `src/crypto/crypto_pbkdf2.h`                     |
| Key derivation (scrypt)         | `src/crypto/crypto_scrypt.cc`, `src/crypto/crypto_scrypt.h`                     |
| Key derivation (HKDF)           | `src/crypto/crypto_hkdf.cc`, `src/crypto/crypto_hkdf.h`                         |
| Key derivation (argon2)         | `src/crypto/crypto_argon2.cc`, `src/crypto/crypto_argon2.h`                     |
| Asymmetric (RSA)                | `src/crypto/crypto_rsa.cc`, `src/crypto/crypto_rsa.h`                           |
| Asymmetric (DSA)                | `src/crypto/crypto_dsa.cc`, `src/crypto/crypto_dsa.h`                           |
| Asymmetric (Diffie-Hellman)     | `src/crypto/crypto_dh.cc`, `src/crypto/crypto_dh.h`                             |
| Asymmetric (EC and CFRG curves) | `src/crypto/crypto_ec.cc`, `src/crypto/crypto_ec.h`                             |
| Post-quantum (ML-KEM / Kyber)   | `src/crypto/crypto_kem.cc`, `src/crypto/crypto_kem.h`                           |
| Post-quantum (ML-DSA)           | `src/crypto/crypto_ml_dsa.cc`, `src/crypto/crypto_ml_dsa.h`                     |
| Signatures (signing/verifying)  | `src/crypto/crypto_sig.cc`, `src/crypto/crypto_sig.h`                           |
| Random (CSPRNG, prime gen)      | `src/crypto/crypto_random.cc`, `src/crypto/crypto_random.h`                     |
| TLS context                     | `src/crypto/crypto_tls.cc`, `src/crypto/crypto_tls.h`                           |
| X.509 certificates              | `src/crypto/crypto_x509.cc`, `src/crypto/crypto_x509.h`                         |
| Certificate signing requests    | `src/crypto/crypto_spkac.cc`, `src/crypto/crypto_spkac.h`                       |
| Top-level glue and binding      | `src/crypto/crypto_util.cc`, `src/crypto/crypto_common.cc`                      |
| Key handling and generation     | `src/crypto/crypto_keys.cc`, `src/crypto/crypto_keygen.cc`                      |

The CFRG family (Ed25519, Ed448, X25519, X448) is implemented in JavaScript at
`lib/internal/crypto/cfrg.js`; in C++ those curves are handled inside `src/crypto/crypto_ec.cc`
(key material) and `src/crypto/crypto_sig.cc` (signing/verifying), since OpenSSL treats them as
elliptic-curve algorithms.

## Two API surfaces

Node.js exposes cryptography through two parallel APIs that share the OpenSSL backend:

| API                                | Reference                          | Style                              |
| ---------------------------------- | ---------------------------------- | ---------------------------------- |
| **Node.js `crypto` (traditional)** | [`crypto.md`](../api/crypto.md)    | Sync, stream-based, Node-native    |
| **Web Crypto (`crypto.subtle`)**   | [`webcrypto.md`](../api/webcrypto.md) | Promise-based, W3C, cross-browser  |

Both surfaces are exposed through the same OpenSSL bindings; they differ in object model, error
shape, and availability. Web Crypto is part of the Web Standards alignment effort documented in
[Web standards](web-standards.md).

### `node:crypto` highlights

```mjs
import { createHash, randomBytes } from 'node:crypto';

const hash = createHash('sha256').update('input').digest('hex');
const nonce = randomBytes(16);
```

```cjs
const { createHash, randomBytes } = require('node:crypto');

const hash = createHash('sha256').update('input').digest('hex');
const nonce = randomBytes(16);
```

### Web Crypto highlights

```mjs
const data = new TextEncoder().encode('input');
const digest = await globalThis.crypto.subtle.digest('SHA-256', data);
```

```cjs
const data = new TextEncoder().encode('input');
globalThis.crypto.subtle.digest('SHA-256', data).then((digest) => { /* ... */ });
```

## Internal layout (`lib/internal/crypto/`)

The internal modules under `lib/internal/crypto/` host the JavaScript-side implementation of the
cryptography APIs. Notable entries include:

| File | Purpose |
| --- | --- |
| `lib/internal/crypto/aes.js` | AES key wrapping and Web Crypto AES algorithms |
| `lib/internal/crypto/argon2.js` | Argon2 KDF |
| `lib/internal/crypto/certificate.js` | X.509 certificate parsing |
| `lib/internal/crypto/cfrg.js` | CFRG (Ed25519, Ed448, X25519, X448) |
| `lib/internal/crypto/chacha20_poly1305.js` | ChaCha20-Poly1305 AEAD |
| `lib/internal/crypto/cipher.js` | Symmetric cipher streams |
| `lib/internal/crypto/diffiehellman.js` | Classic DH and ECDH |
| `lib/internal/crypto/ec.js` | NIST elliptic-curve algorithms |
| `lib/internal/crypto/hash.js` | Hash and HMAC streams |
| `lib/internal/crypto/hashnames.js` | Hash-name normalization between OpenSSL and Web Crypto |

Many additional files implement specific algorithms or shared utilities; the full list lives under
`lib/internal/crypto/` in the repository.

## TLS

The `node:tls` module surfaces OpenSSL-backed TLS for `node:net` sockets. Its public surface is at
[`../api/tls.md`](../api/tls.md), with relevant internal modules at `lib/_tls_common.js` and
`lib/_tls_wrap.js`. TLS use cases (HTTPS, HTTP/2 over TLS, custom TCP servers with TLS, mutual TLS)
are documented in [Networking](networking.md) and [`../api/https.md`](../api/https.md),
[`../api/http2.md`](../api/http2.md).

## FIPS-compliant builds

For deployments requiring NIST FIPS 140-3 compliance, the runtime can be built against the OpenSSL
FIPS provider. The complete build recipe (FIPS provider configuration, `enable_fips_include.py`
helper, runtime check via `node --enable-fips`) is in [`../../BUILDING.md`](../../BUILDING.md). The
maintainer-side details for the OpenSSL bundling are in
[`../contributing/maintaining/maintaining-openssl.md`](../contributing/maintaining/maintaining-openssl.md).

## OpenSSL update workflow

The OpenSSL bundled at `deps/openssl/` is automatically refreshed by the
`tools/dep_updaters/update-openssl.sh` script (invoked by the `update-openssl.yml` workflow). The
maintainer-side procedure is in
[`../contributing/maintaining/maintaining-openssl.md`](../contributing/maintaining/maintaining-openssl.md).
For an integrated view of all bundled dependencies, see [Dependencies](dependencies.md).

## Cross-references

* Architectural overview: [Architecture overview](overview.md)
* Feature catalog: [Feature catalog](features.md)
* Integration narrative: [Integration](integration.md)
* Bundled dependencies (OpenSSL details): [Dependencies](dependencies.md)
* Networking (TLS use, HTTPS, HTTP/2-over-TLS): [Networking](networking.md)
* Web Standards (`crypto.subtle`): [Web standards](web-standards.md)
* Public APIs:
  * [`../api/crypto.md`](../api/crypto.md)
  * [`../api/webcrypto.md`](../api/webcrypto.md)
  * [`../api/tls.md`](../api/tls.md)
  * [`../api/https.md`](../api/https.md)
* OpenSSL maintainer guide:
  [`../contributing/maintaining/maintaining-openssl.md`](../contributing/maintaining/maintaining-openssl.md)
* Build and platform requirements (FIPS recipe): [`../../BUILDING.md`](../../BUILDING.md)
* Project security policy: [`../../SECURITY.md`](../../SECURITY.md)
