# CHANGELOG for crypton

## Unreleased

`Crypto.PubKey.MLKEM` and `Crypto.PubKey.MLDSA`: ML-KEM and ML-DSA, the
post-quantum key encapsulation and signature schemes of FIPS 203 and FIPS
204, through mlkem-native and mldsa-native.

[All changes](https://github.com/kazu-yamamoto/crypton/compare/crypton-v2.1.7...main)

## 2.1.7

RSA-PSS verification was accepting an encoding RFC 8017 says to refuse.  It
is a conformance fault rather than a forgery: producing such a signature takes
the private key, so a third party holding a valid signature cannot turn it
into one of these.

[All changes](https://github.com/kazu-yamamoto/crypton/compare/crypton-v2.1.6...crypton-v2.1.7)

## 2.1.6

A PBKDF2 output length of zero no longer aborts the process, and
`Crypto.PubKey.ECC.P256.scalarInv` no longer loops forever on zero.  The
`license:` field now says what the tree holds, `BSD-3-Clause AND MIT AND
ISC`, and `license-files:` lists all five texts.  **Nothing is required of a
user that was not required before**; the field was simply incomplete.

[All changes](https://github.com/kazu-yamamoto/crypton/compare/crypton-v2.1.5...crypton-v2.1.6)

## 2.1.5

2.1.3 and 2.1.4 cannot be built with GCC 14 or newer.  This release is that
fix.  AES-GCM is also about a quarter faster on AArch64.

[All changes](https://github.com/kazu-yamamoto/crypton/compare/crypton-v2.1.4...crypton-v2.1.5)

## 2.1.4

2.1.3 could not be built from Hackage in the default configuration, and is
deprecated there.  This release is that fix, and two more: the C builds with
gcc before 13 on AArch64 again, and SHA-256 on AArch64 is back to the speed
of 2.1.2, which 2.1.3 had cut to a fifth.  It is deprecated on Hackage in
turn, as it does not build with GCC 14 or newer; use 2.1.5 or later.

[All changes](https://github.com/kazu-yamamoto/crypton/compare/crypton-v2.1.3...crypton-v2.1.4)

## 2.1.3

Deprecated on Hackage, as its tarball does not build; use 2.1.5 or later.  A
hardening release.  One-call AES-GCM and ChaCha20-Poly1305 decryption accepted
a tag of no bytes, which skipped authentication; a tag must now be 4 to 16
bytes for AES-GCM and exactly 16 for ChaCha20-Poly1305, so a truncated
ChaCha20-Poly1305 tag is refused.  `Crypto.Cipher.AES.GCM` refuses an empty
nonce, which gave the authentication key away.  A message of 4 GiB or more was
silently truncated on its way to the C; the streaming ciphers now process all
of it and the one-call AEADs and AES modes refuse it.  The arithmetic used
without GMP no longer overruns a buffer, two threads setting up AES keys no
longer race, and the three cabal flag settings that were broken work again.

RSA on AArch64, Ed25519 signing and key generation, and P-256 ECDSA
verification are faster, and AES-GCM uses 512-bit VAES where x86-64 has it.
The C now runs under the sanitizers and a constant-time check in CI, on
32-bit and big-endian machines too, and key material is wiped where the
compiler cannot skip it.

[All changes](https://github.com/kazu-yamamoto/crypton/compare/crypton-v2.1.2...crypton-v2.1.3)

## 2.1.2

Faster only; no API change.  P-256, P-384, P-521, X25519 and, on x86-64, RSA
go through AWS's s2n-bignum, hand-written assembly carrying machine-checked
proofs, which makes them up to nine times faster depending on the curve and
the operation.  AES-GCM uses the wide VAES instructions where x86-64 has them.

[All changes](https://github.com/kazu-yamamoto/crypton/compare/crypton-v2.1.1...crypton-v2.1.2)

## 2.1.1

New functions only; existing code is unaffected.  `Crypto.PubKey.ECDSA` gains
RFC 6979 deterministic nonces, `Crypto.Cipher.ChaCha.Poly1305` does a whole
ChaCha20-Poly1305 message in one call, and `Crypto.Cipher.AES.GCM` gains
`decryptWithTag` for protocols that carry the tag apart from the ciphertext.

[All changes](https://github.com/kazu-yamamoto/crypton/compare/crypton-v2.1.0...crypton-v2.1.1)

## 2.1.0

AES-GCM is several times faster on short messages, the packet sizes QUIC and
TLS send, on x86-64 and AArch64.  New are `Crypto.Cipher.AES.GCM` for many
messages under one key, `encryptWithMask` for QUIC header protection, and
Skein with the digest size as a type parameter.  The Broadwell crash fixed in
2.0.1 is fixed here too.

One change breaks callers: `Crypto.Cipher.ChaChaPoly1305.initialize` and
`initializeX` take a checked `Key`, built with `key`, and return a `State`
rather than a `CryptoFailable State`.

[All changes](https://github.com/kazu-yamamoto/crypton/compare/crypton-v2.0.0...crypton-v2.1.0)

## 2.0.1

A maintenance release from the 2.0 branch with one fix: SHA-512 and ChaCha20
crashed on Intel processors from Broadwell on.

[All changes](https://github.com/kazu-yamamoto/crypton/compare/crypton-v2.0.0...crypton-v2.0.1)

## 2.0.0

A large release, faster and harder to attack.  AES, GHASH, SHA-1, SHA-2,
SHA-3, ChaCha20 and Poly1305 use the processor's instructions on x86-64 and
AArch64, several through the CRYPTOGAMS assembly OpenSSL uses; DES, Camellia,
Twofish and Blowfish run in C rather than Haskell; and point multiplication on
every prime curve runs in C.  RSA, DSA, ECDSA and the OTP checks no longer let
a secret decide how long they take, `expSafe` hides its exponent again, and
showing a private key no longer prints it (`Crypto.Debug` prints one on
purpose).  `Crypto.PubKey.ElGamal` is exposed.

Upgrading can break code in three ways.  Input that used to be accepted is now
refused: a value at or above an RSA or Rabin modulus, a signature of the wrong
length or out of range, a digest too short for HOTP's dynamic truncation, a
non-canonical Ed25519 signature, a DH or ECDH peer value that fails
validation, an ECC public point outside the prime-order subgroup, a PKCS#7
block size outside 1..255, and block cipher input that is not a whole number
of blocks.  A refused KDF, Argon2 or bcrypt parameter, and a peer value
`getShared` refuses, is reported as a `CryptoError` rather than an
`ErrorCall`; `CryptoError_ParameterInvalid` is appended to `CryptoError`, and
`tryGetShared` is added beside `getShared`.  `Crypto.MAC.Poly1305.initialize`
and `auth` take a checked `Key`, built with `key`, so `initialize` can no
longer fail.

The eighteen curves over a binary field in `Crypto.ECC.Simple.Types` are
deprecated.  They are obsolete, they are the curves whose cofactor is not 1,
and they will go in a later major version.  Prefer a prime curve, or X25519.

[All changes](https://github.com/kazu-yamamoto/crypton/compare/crypton-v1.1.5...crypton-v2.0.0)

## 1.1.5

* fix(aead): reject undersized tags
  [#80](https://github.com/kazu-yamamoto/crypton/pull/80)
* fix(aes): refuse a zero-length AES-GCM IV
  [#79](https://github.com/kazu-yamamoto/crypton/pull/79)
* fix(p256): prevent crashes when validating valid points
  [#78](https://github.com/kazu-yamamoto/crypton/pull/78)
* feat(asn1): add SHA-3 HashAlgorithmASN1 instances for PKCS#1 v1.5
  [#77](https://github.com/kazu-yamamoto/crypton/pull/77)
* OCB3 conformance
  [#76](https://github.com/kazu-yamamoto/crypton/pull/76)

## 1.1.4

* Generic instance for RSA PublicKey and PrivateKey

## 1.1.3

* Ensure that `pointAdd` in `PubKey.ECC.P256` treats the point at infinity as the additive identity.
  [#73](https://github.com/kazu-yamamoto/crypton/pull/73)

## 1.1.2

* Preparing `ram` v0.22.
* Generalizing RSA encrypt/decrypt to manipulate ScrubbedBytes directly.

## 1.1.1

* On iOS, ScrubbedBytes based hashing is used for seedNew. On other
  plateforms, entropy is used directly as used to be.
  [#71](https://github.com/kazu-yamamoto/crypton/pull/71)

## 1.1.0

* Removing "basement" and "memory".
  [#67](https://github.com/kazu-yamamoto/crypton/pull/67)


## 1.0.7

* Stop depending on basement, use upstream dependencies instead
* Stop transitively depending on basement by depending on ram.

## 1.0.6

* Fix test failures on less common 64-bit arches.
  [#65](https://github.com/kazu-yamamoto/crypton/pull/65)

## 1.0.5

* Setter/Getter for ChaCha counter.
  [#63](https://github.com/kazu-yamamoto/crypton/pull/63)
* Add simple interface to generate full blocks
  [#60](https://github.com/kazu-yamamoto/crypton/pull/60)
* Avoid `ghc-prim` dependency.
  [#61](https://github.com/kazu-yamamoto/crypton/pull/61)

## 1.0.4

* Ed448.sign: avoid extra re-derive of public key.
  [#48](https://github.com/kazu-yamamoto/crypton/pull/48)

## 1.0.3

* Make sign of Ed25519/Ed448 safer. The public key parameter is
  ignored and its public key is generated from the secret key
  parameter to prevent Double Public Key Signing Function Oracle
  Attack.
  [#47](https://github.com/kazu-yamamoto/crypton/pull/47)

## 1.0.2

* Deterministic Nonce Generation for ECDSA
  [#46](https://github.com/kazu-yamamoto/crypton/pull/46)
* ECDSA Signature Normalization.
  [#45](https://github.com/kazu-yamamoto/crypton/pull/45)
* Add Full Test Suite from RFC 6979.
  [#44](https://github.com/kazu-yamamoto/crypton/pull/44)
* ECDSA with Public Key Recovery.
  [#43](https://github.com/kazu-yamamoto/crypton/pull/43)
* Providing necessary features for HPKE.
  [#42](https://github.com/kazu-yamamoto/crypton/pull/42)

## 1.0.1

* Update decaf library.
  [#38](https://github.com/kazu-yamamoto/crypton/pull/38)
* Add TypeOperators language extension to EdDSA.hs.
  [#36](https://github.com/kazu-yamamoto/crypton/pull/36)

## 1.0.0

* Versions follow the standard version policy.
* Removing pthread stuff.
  [#32](https://github.com/kazu-yamamoto/crypton/pull/32)

## 0.34

* Hashing getRandomBytes before using as Seed for ChaChaDRG
  [#24](https://github.com/kazu-yamamoto/crypton/pull/24)
* Add support for XChaCha and XChaChaPoly1305
  [#18](https://github.com/kazu-yamamoto/crypton/pull/18)
* Strict byteArray of IV c
  [#16](https://github.com/kazu-yamamoto/crypton/pull/16)

## 0.33

* Add "crypton_" prefix to the final C symbols.
  [#9](https://github.com/kazu-yamamoto/crypton/pull/9)

## 0.32

* All C symbols now have the "crypton_" prefix.
  [#7](https://github.com/kazu-yamamoto/crypton/pull/7)
  [#8](https://github.com/kazu-yamamoto/crypton/pull/8)

## 0.31

* Crypton is forked from cryptonite with the original authors permission.
* Ignoring exceptons from hClose to read the next entropy
  [#1](https://github.com/kazu-yamamoto/crypton/pull/1)
* Enabling the support_pclmuldq flag by default.

## 0.30

* Fix some C symbol blake2b prefix to be cryptonite_ prefix (fix mixing with other C library)
* add hmac-lazy
* Fix compilation with GHC 9.2
* Drop support for GHC8.0, GHC8.2, GHC8.4, GHC8.6

## 0.29

* advance compilation with gmp breakage due to change upstream
* Add native EdDSA support

## 0.28

* Add hash constant time capability
* Prevent possible overflow during hashing by hashing in 4GB chunks

## 0.27

* Optimise AES GCM and CCM
* Optimise P256R1 implementation
* Various AES-NI building improvements
* Add better ECDSA support
* Add XSalsa derive
* Implement square roots for ECC binary curve
* Various tests and benchmarks

## 0.26

* Add Rabin cryptosystem (and variants)
* Add bcrypt_pbkdf key derivation function
* Optimize Blowfish implementation
* Add KMAC (Keccak Message Authentication Code)
* Add ECDSA sign/verify digest APIs
* Hash algorithms with runtime output length
* Update blake2 to latest upstream version
* RSA-PSS with arbitrary key size
* SHAKE with output length not divisible by 8
* Add Read and Data instances for Digest type
* Improve P256 scalar primitives
* Fix hash truncation bug in DSA
* Fix cost parsing for bcrypt
* Fix ECC failures on arm64
* Correction to PKCS#1 v1.5 padding
* Use powModSecInteger when available
* Drop GHC 7.8 and GHC 7.10 support, refer to pkg-guidelines
* Optimise GCM mode
* Add little endian serialization of integer

## 0.25

* Improve digest binary conversion efficiency
* AES CCM support
* Add MonadFailure instance for CryptoFailable
* Various misc improvements on documentation
* Edwards25519 lowlevel arithmetic support
* P256 add point negation
* Improvement in ECC (benchmark, better normalization)
* Blake2 improvements to context size
* Use gauge instead of criterion
* Use haskell-ci for CI scripts
* Improve Digest memory representation to be 2 less Ints and one less boxing
  moving from `UArray` to `Block`

## 0.24

* Ed25519: generateSecret & Documentation updates
* Repair tutorial
* RSA: Allow signing digest directly
* IV add: fix overflow behavior
* P256: validate point when decoding
* Compilation fix with deepseq disabled
* Improve Curve448 and use decaf for Ed448
* Compilation flag blake2 sse merged in sse support
* Process unaligned data better in hashes and AES, on architecture needing alignment
* Drop support for ghc 7.6
* Add ability to create random generator Seed from binary data and
  loosen constraint on ChaChaDRG seed from ByteArray to ByteArrayAccess.
* Add 3 associated types with the HashAlgorithm class, to get
  access to the constant for BlockSize, DigestSize and ContextSize at the type level.
  the related function that this replaced will be deprecated in later release, and
  eventually removed.

API CHANGES:

* Improve ECDH safety to return failure for bad inputs (e.g. public point in small order subgroup).
  To go back to previous behavior you can replace `ecdh` by `ecdhRaw`. It's recommended to
  use `ecdh` and handle the error appropriately.
* Users defining their own HashAlgorithm needs to define the
  HashBlockSize, HashDigest, HashInternalContextSize associated types

## 0.23

* Digest memory usage improvement by using unpinned memory
* Fix generateBetween to generate within the right bounds
* Add pure Twofish implementation
* Fix memory allocation in P256 when using a temp point
* Consolidate hash benchmark code
* Add Nat-length Blake2 support (GHC > 8.0)
* Update tutorial

## 0.22

* Add Argon2 (Password Hashing Competition winner) hash function
* Update blake2 to latest upstream version
* Add extra blake2 hashing size
* Add faster PBKDF2 functions for SHA1/SHA256/SHA512
* Add SHAKE128 and SHAKE256
* Cleanup prime generation, and add tests
* Add Time-based One Time Password (TOTP) and HMAC-based One Time Password (HOTP)
* Rename Ed448 module name to Curve448, old module name still valid for now

## 0.21

* Drop automated tests with GHC 7.0, GHC 7.4, GHC 7.6. support dropped, but probably still working.
* Improve non-aligned support in C sources, ChaCha and SHA3 now probably work on arch without support for unaligned access. not complete or tested.
* Add another ECC framework that is more flexible, allowing different implementations to work instead of
  the existing Pure haskell NIST implementation.
* Add ECIES basic primitives
* Add XSalsa20 stream cipher
* Process partial buffer correctly with Poly1305

## 0.20

* Fixed hash truncation used in ECDSA signature & verification (Olivier Chéron)
* Fix ECDH when scalar and coordinate bit sizes differ (Olivier Chéron)
* Speed up ECDSA verification using Shamir's trick (Olivier Chéron)
* Fix rdrand on windows

## 0.19

* Add tutorial (Yann Esposito)
* Derive Show instance for better interaction with Show pretty printer (Eric Mertens)

## 0.18

* Re-used standard rdrand instructions instead of bytedump of rdrand instruction
* Improvement to F2m, including lots of tests (Andrew Lelechenko)
* Add error check on salt length in bcrypt

## 0.17

* Add Miyaguchi-Preneel construction (Kei Hibino)
* Fix buffer length in scrypt (Luke Taylor)
* build fixes for i686 and arm related to rdrand

## 0.16

* Fix basepoint for Ed448

* Enable 64-bit Curve25519 implementation

## 0.15

* Fix serialization of DH and ECDH

## 0.14

* Reduce size of SHA3 context instead of allocating all-size fit memory. save
  up to 72 bytes of memory per context for SHA3-512.
* Add a Seed capability to the main DRG, to be able to debug/reproduce randomized program
  where you would want to disable the randomness.
* Add support for Cipher-based Message Authentication Code (CMAC) (Kei Hibino)
* *CHANGE* Change the `SharedKey` for `Crypto.PubKey.DH` and `Crypto.PubKey.ECC.DH`,
  from an Integer newtype to a ScrubbedBytes newtype. Prevent mistake where the
  bytes representation is generated without the right padding (when needed).
* *CHANGE* Keep The field size in bits, in the `Params` in `Crypto.PubKey.DH`,
  moving from 2 elements to 3 elements in the structure.

## 0.13

* *SECURITY* Fix buffer overflow issue in SHA384, copying 16 extra bytes from
  the SHA512 context to the destination memory pointer leading to memory
  corruption, segfault. (Mikael Bung)

## 0.12

* Fix compilation issue with Ed448 on 32 bits machine.

## 0.11

* Truncate hashing correctly for DSA
* Add support for HKDF (RFC 5869)
* Add support for Ed448
* Extends support for Blake2s to 224 bits version.
* Compilation workaround for old distribution (RHEL 4.1)
* Compilation fix for AIX
* Compilation fix with AESNI and ghci compiling C source in a weird order.
* Fix example compilation, typo, and warning

## 0.10

* Add reference implementation of blake2 for non-SSE2 platform
* Add support\_blake2\_sse flag

## 0.9

* Quiet down unused module imports
* Move Curve25519 over to Crypto.Error instead of using Either String.
* Add documentation for ChaChaPoly1305
* Add missing documentation for various modules
* Add a way to create Poly1305 Auth tag.
* Added support for the BLAKE2 family of hash algorithms
* Fix endianness of incrementNonce function for ChaChaPoly1305

## 0.8

* Add support for ChaChaPoly1305 Nonce Increment (John Galt)
* Move repository to the haskell-crypto organisation

## 0.7

* Add PKCS5 / PKCS7 padding and unpadding methods
* Fix ChaChaPoly1305 Decryption
* Add support for BCrypt (Luke Taylor)

## 0.6

* Add ChaChaPoly1305 AE cipher
* Add instructions in README for building on old OSX
* Fix blocking /dev/random Andrey Sverdlichenko

## 0.5

* Fix all strays exports to all be under the cryptonite prefix.

## 0.4

* Add a System DRG that represent a referentially transparent of evaluated bytes
  while using lazy evaluation for future entropy values.

## 0.3

* Allow drgNew to run in any MonadRandom, providing cascading initialization
* Remove Crypto.PubKey.HashDescr in favor of just having the algorithm
  specified in PKCS15 RSA function.
* Fix documentation in cipher sub section (Luke Taylor)
* Cleanup AES dead functions (Luke Taylor)
* Fix Show instance of Digest to display without quotes similar to cryptohash
* Use scrubbed bytes instead of bytes for P256 scalar

## 0.2

* Fix P256 compilation and exactness, + add tests
* Add a raw memory number serialization capability (i2osp, os2ip)
* Improve tests for number serialization
* Improve tests for ECC arithmetics
* Add Ord instance for Digest (Nicolas Di Prima)
* Fix entropy compilation on windows 64 bits.

## 0.1

* Initial release

