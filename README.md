# KeyFinder — Cryptographic Key Search Tool

by Luigi Origa

KeyFinder searches binary files and memory dumps for cryptographic keys, secrets, and artifacts. It identifies asymmetric private/public key pairs, symmetric key schedules expanded in memory, X.509 certificates, and can attempt trial decryption of encrypted files using candidate keys extracted from source binaries.

## Command Line Reference

```
KeyFinder.exe [options] [+algo...] [-algo...]
```

### General

```
--help, --h              this help
--i path|filename        path or binary to load (key source, repeatable)
--th number              number of threads (default: auto)
--r                      scan directories recursively
```

Also accepted: `-h`, `-?`

### Scan Modes (independent of algorithm selection)

```
--analyze                search for common crypto constants
--certs                  search/extract DER/PEM X509, PKCS#7, CSR
--deep_scan              deep scan for keys, secrets and crypto artifacts
```

### Algorithm Selection

`+name` enables, `-name` disables. No algorithms are enabled by default.

```
Asymmetric key search (private/public keys):
  +asym                    enable all asymmetric algorithms
  names: ed25519 ed448 secp256k1 secp256r1 secp384r1 secp521r1
         x25519 x448
         bp256r1 bp384r1 bp512r1 (Brainpool curves)
         dsa1024 dsa2048 dsa3072
         rsa128 rsa256 rsa512 rsa1024 rsa2048 rsa4096

Symmetric key schedule search (expanded keys in memory):
  +sym                     enable all symmetric algorithms
  names: aes des camellia sm4 serpent idea twofish seed
         cast128 aria rc5rc6 chacha20
```

Use `-name` after `+asym` or `+sym` to exclude specific algorithms.
Example: `+asym -rsa128 -rsa256` (all asymmetric except RSA-128 and RSA-256)

### Trial Decrypt (requires `--e` and at least one `+sym` algorithm)

```
--e path|filename        encrypted file target (repeatable)
--mode ecb|cbc|ctr       cipher mode (default: ecb)
--iv hex                 IV for CBC (32 hex = 16 bytes)
                         or nonce for CTR (24 hex = 12 bytes)
--ctr_start number       initial counter value for CTR (default: 0)
--validate_arm           post-filter: validate ARM Cortex-M vector table
```

Supported ciphers for trial decrypt:

| Mode | Ciphers |
|------|---------|
| ECB | AES-128, AES-256, Camellia-128/256, SM4, Serpent-256, Twofish-256, ARIA-128/256, SEED, DES, 3DES, IDEA, CAST-128 |
| CBC | AES-128, AES-256, Camellia-128/256, SM4, Serpent-256, Twofish-256, ARIA-128/256, SEED |
| CTR | AES-128, AES-256 |

Note: CBC requires 16-byte block ciphers (DES, 3DES, IDEA, CAST-128 have 8-byte blocks and are excluded). CTR is implemented only for AES. RC5/RC6 and ChaCha20 are detected as key schedules but do not support trial decrypt.

### Advanced Decrypt

```
--xor_compare file       XOR-compare encrypted files (use 2+, repeatable)
--partial_key pattern    brute-force key with '?' wildcards (max 40 bits)
--kdf sha256|hmac|cmac|hkdf  derive key via KDF before trial decrypt (*)
--kdf_constant hex       constant/info parameter for KDF (*)
```

(*) KDF is currently disabled — the required library (kdf/hkdf.h) is not yet implemented. The argument is accepted but prints a warning and does nothing.

### Filters and Options

```
--no_cross               disable cross-file key comparison
--no_gpu                 disable GPU acceleration
--ignore_excluded        ignore excluded extensions
--xe ext                 exclude extension (repeatable)
--xd dir                 exclude directory (repeatable)
--clean_db               remove all sidecar/cache files
```

## Supported Algorithms

### Asymmetric (20 algorithms)

| Family | Algorithms | Private Key Size | Public Key Size | Cross-Compare |
|--------|-----------|------------------|-----------------|---------------|
| EdDSA | ED25519 | 32 bytes | 32 bytes | yes |
| EdDSA | ED448 | 57 bytes | 57 bytes | yes |
| ECDSA | SECP256K1, SECP256R1 | 32 bytes | 32+32 bytes (X,Y) | yes |
| ECDSA | SECP384R1 | 48 bytes | 48+48 bytes (X,Y) | yes |
| ECDSA | SECP521R1 | 66 bytes | 66+66 bytes (X,Y) | yes |
| X-DH | X25519 | 32 bytes | 32 bytes | yes |
| X-DH | X448 | 56 bytes | 56 bytes | yes |
| Brainpool | BP256R1 | 32 bytes | 32+32 bytes (X,Y) | yes |
| Brainpool | BP384R1 | 48 bytes | 48+48 bytes (X,Y) | yes |
| Brainpool | BP512R1 | 64 bytes | 64+64 bytes (X,Y) | yes |
| DSA | DSA-1024 | 20 bytes | 128 bytes (y = g^x mod p) | yes |
| DSA | DSA-2048 | 32 bytes | 256 bytes (y = g^x mod p) | yes |
| DSA | DSA-3072 | 32 bytes | 384 bytes (y = g^x mod p) | yes |
| RSA | RSA-128 (128-bit) | P=8B, Q=8B | N=16 bytes | no (*) |
| RSA | RSA-256 (256-bit) | P=16B, Q=16B | N=32 bytes | no (*) |
| RSA | RSA-512 (512-bit) | P=32B, Q=32B | N=64 bytes | no (*) |
| RSA | RSA-1024 (1024-bit) | P=64B, Q=64B | N=128 bytes | no (*) |
| RSA | RSA-2048 (2048-bit) | P=128B, Q=128B | N=256 bytes | no (*) |
| RSA | RSA-4096 (4096-bit) | P=256B, Q=256B | N=512 bytes | no (*) |

(*) RSA does not have cross-file comparison. RSA search finds primes P,Q within each file and checks if N=P*Q matches a public key pattern in the same file.

All asymmetric algorithms are **disabled by default**. Enable with `+asym` or `+name`.

### Symmetric (12 algorithms)

| Algorithm | Key Schedule Detection | Trial Decrypt |
|-----------|----------------------|---------------|
| AES | yes (128/256-bit key schedule) | ECB, CBC, CTR |
| DES | yes (56-bit + 16 subkeys) | ECB only |
| 3DES | — (detected via DES) | ECB only |
| Camellia | yes (128-bit key schedule) | ECB, CBC |
| SM4 | yes (128-bit, 32 round keys) | ECB, CBC |
| Serpent | yes (128/256-bit, 33 round keys) | ECB, CBC |
| IDEA | yes (128-bit, 52 subkeys) | ECB only |
| Twofish | yes (128/256-bit, 40 subkeys) | ECB, CBC |
| SEED | yes (128-bit, 16 round keys) | ECB, CBC |
| CAST-128 | yes (128-bit, 32 subkeys) | ECB only |
| ARIA | yes (128-bit, 13 round keys) | ECB, CBC |
| RC5/RC6 | yes (RC5-32/12 + RC6-32/20) | no |
| ChaCha20 | yes ("expand 32-byte k" constant) | no |

All symmetric algorithms are **disabled by default**. Enable with `+sym` or `+name`.

## Features

- **Multi-threaded CPU** — all key search and trial decrypt operations are parallelized
- **GPU acceleration (OpenCL)** — ED25519, ED448, SECP curves, X25519, X448, Brainpool, DSA, RSA primes, AES key schedule search, partial key brute-force
- **Cross-file comparison** — finds private key in file A matching public key in file B (all asymmetric except RSA)
- **SHA-256 deduplication** — skips files with identical content; caches completed algorithms per-file
- **Trial decrypt** — sliding-window key extraction with automatic variant generation (original, reversed, endian-swapped)
- **Certificate extraction** — DER/PEM X.509, PKCS#7/CMS SignedData, PKCS#10 CSR, PKCS#8/PKCS#1 private keys

## Examples

```bash
# Search all asymmetric keys in a firmware dump
KeyFinder.exe --i firmware.bin +asym

# Recursive scan with all algorithms and 8 threads
KeyFinder.exe --i C:\dumps --r +asym +sym --th 8

# Specific algorithms only
KeyFinder.exe --i file1.bin --i file2.bin +rsa2048 +secp256r1

# All asymmetric except small RSA
KeyFinder.exe --i firmware.bin +asym -rsa128 -rsa256

# Certificate and constant analysis
KeyFinder.exe --i C:\dumps --r --certs --analyze

# Deep scan, CPU only
KeyFinder.exe --i target.bin --deep_scan --no_gpu

# Exclude extensions and directories
KeyFinder.exe --i C:\dumps --r +asym --xe log --xd temp

# Clean sidecar/cache files
KeyFinder.exe --i C:\dumps --r --clean_db
```

### Trial Decrypt

```bash
# ECB (default)
KeyFinder.exe --i keydump.bin --e encrypted.bin +aes

# CBC with IV
KeyFinder.exe --i dump.bin --e firmware.bin +aes --mode cbc --iv 00112233445566778899AABBCCDDEEFF

# CTR with nonce (AES only)
KeyFinder.exe --i memory.bin --e payload.bin +aes --mode ctr --iv 00112233445566778899AABB --ctr_start 1

# All symmetric ciphers
KeyFinder.exe --i source.bin --e target.bin +sym

# Partial key brute-force
KeyFinder.exe --e encrypted.bin --partial_key 0123456789ABCDEF???????????????? +aes

# XOR-compare (detect key reuse)
KeyFinder.exe --xor_compare file1.enc --xor_compare file2.enc

# ARM firmware validation
KeyFinder.exe --i dump.bin --e firmware.enc +aes --validate_arm
```

## Sidecar Files

KeyFinder caches results alongside scanned files:

| Extension | Content |
|-----------|---------|
| `.sha256` | File hash + completed algorithm list |
| `.P8` `.P16` `.P20` `.P32` `.P48` `.P56` `.P57` `.P64` `.P66` `.P128` `.P256` `.P512` | Raw pattern candidates |
| `.KED25519` `.KED448` `.KSECP256K1` `.KSECP256R1` `.KSECP384R1` `.KSECP521R1` `.KX25519` `.KX448` `.KBP256R1` `.KBP384R1` `.KBP512R1` `.KDSA1024` `.KDSA2048` `.KDSA3072` | Computed key pairs |
| `.KRSA128` `.KRSA256` `.KRSA512` `.KRSA1024` `.KRSA2048` `.KRSA4096` | RSA key pairs |
| `.M8` `.M16` `.M32` `.M64` `.M128` `.M256` `.M512` | RSA prime candidates (0 bytes = no primes found) |

Use `--clean_db` to remove all sidecar files.
