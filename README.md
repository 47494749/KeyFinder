# KeyFinder — Cryptographic Key Search Tool

by Luigi Origa

KeyFinder searches binary files and memory dumps for cryptographic keys, secrets, and artifacts. It identifies asymmetric private/public key pairs, symmetric key schedules expanded in memory, X.509 certificates, and can attempt trial decryption of encrypted files using candidate keys extracted from source binaries.

## Supported Algorithms

### Asymmetric Key Search (private/public key pairs)

| Family | Algorithms |
|--------|-----------|
| EdDSA | ED25519, ED448 |
| ECDSA / ECDH | SECP256K1, SECP256R1, SECP384R1, SECP521R1 |
| X-DH | X25519, X448 |
| Brainpool | BP256R1, BP384R1, BP512R1 |
| DSA | DSA-1024, DSA-2048, DSA-3072 |
| RSA | RSA-128, RSA-256, RSA-512, RSA-1024, RSA-2048, RSA-4096 |

### Symmetric Key Schedule Search (expanded keys in memory)

AES, DES, Camellia, SM4, Serpent, IDEA, Twofish, SEED, CAST-128, ARIA, RC5/RC6, ChaCha20

### Trial Decrypt (brute-force decryption with candidate keys)

| Mode | Supported Ciphers |
|------|-------------------|
| ECB | AES-128, AES-256, Camellia-128/256, SM4, Serpent-256, Twofish-256, ARIA-128/256, SEED, DES, 3DES, IDEA, CAST-128 |
| CBC | AES-128, AES-256, Camellia-128/256, SM4, Serpent-256, Twofish-256, ARIA-128/256, SEED (16-byte block ciphers) |
| CTR | AES-128, AES-256 |

## Command Line Reference

### General

| Option | Description |
|--------|-------------|
| `-h`, `-?` | Show help |
| `-i path\|filename` | Path or binary to load as key source (repeatable) |
| `-th number` | Number of threads (default: auto-detect) |
| `-r` | Scan directories recursively |

### Trial Decrypt

| Option | Description |
|--------|-------------|
| `-e path\|filename` | Encrypted file to trial-decrypt (repeatable) |
| `-mode ecb\|cbc\|ctr` | Cipher mode for trial decrypt (default: ecb) |
| `-iv hex` | IV for CBC (32 hex chars = 16 bytes) or nonce for CTR (24 hex chars = 12 bytes) |
| `-ctr_start number` | Initial counter value for CTR mode (default: 0) |

### Algorithm Toggles

`+name` enables an algorithm, `-name` disables it.

**Default enabled:** ed25519, ed448

| Toggle | Description |
|--------|-------------|
| `+asym` / `-asym` | Enable / disable all asymmetric algorithms |
| `+sym` / `-sym` | Enable / disable all symmetric algorithms |

**Asymmetric names:** `ed25519` `ed448` `secp256k1` `secp256r1` `secp384r1` `secp521r1` `x25519` `x448` `bp256r1` `bp384r1` `bp512r1` `dsa1024` `dsa2048` `dsa3072` `rsa128` `rsa256` `rsa512` `rsa1024` `rsa2048` `rsa4096`

**Symmetric names:** `aes` `des` `camellia` `sm4` `serpent` `idea` `twofish` `seed` `cast128` `aria` `rc5rc6` `chacha20`

### Scan Modes

| Option | Description |
|--------|-------------|
| `-analyze` | Search for common cryptographic constants |
| `-analyze_only` | Search for constants only (no key search) |
| `-certs` | Search and extract DER X.509 certificates |
| `-certs_only` | Extract certificates only (no key search) |
| `-deep_scan` | Deep scan for keys, secrets, and crypto artifacts |
| `-deep_scan_only` | Deep scan only (no key search) |

### Filters and Options

| Option | Description |
|--------|-------------|
| `-no_cross` | Disable cross-file key comparison |
| `-no_gpu` | Disable GPU (OpenCL) acceleration |
| `-ignore_excluded` | Ignore excluded file extensions |
| `-xe ext` | Exclude a file extension (repeatable) |
| `-xd dir` | Exclude a directory (repeatable) |
| `-clean_db` | Remove all sidecar/cache files |

## Default Excluded Extensions

`txt`, `jpg`, `bmp`, `ico`, `xls`, `doc`, `hex`, `bak`, `idb`, `idc`, `py`

## Features

- **Multi-threaded CPU** — all key search and trial decrypt operations are parallelized
- **GPU acceleration** — OpenCL-based search for ED25519, ED448, SECP, RSA primes, AES key schedules, and more
- **Cross-file comparison** — finds matching private/public key pairs across multiple files
- **SHA-256 deduplication** — skips files with identical content; caches completed algorithms per-file
- **Trial decrypt** — sliding-window key extraction from source files with automatic variant generation (original, reversed, endian-swapped)

## Examples

```
KeyFinder -i firmware.bin
KeyFinder -i C:\dumps -r -th 8
KeyFinder -i file1.bin -i file2.bin +rsa2048 +secp256r1
KeyFinder -i firmware.bin -aes -ed448
KeyFinder -i C:\dumps -r -certs -analyze
KeyFinder -i target.bin -deep_scan -no_gpu
KeyFinder -i C:\dumps -r -xe log -xd temp
```

### Trial Decrypt Examples

```
# ECB mode (default) — try all symmetric keys from source against encrypted file
KeyFinder -i keydump.bin -e encrypted.bin +aes

# CBC mode — requires 16-byte IV
KeyFinder -i tcm_dump.bin -e firmware.bin +aes -mode cbc -iv 00112233445566778899AABBCCDDEEFF

# CTR mode — requires 12-byte nonce and optional start counter
KeyFinder -i memory.bin -e payload.bin +aes -mode ctr -iv 00112233445566778899AABB -ctr_start 1

# Multiple ciphers in ECB trial decrypt
KeyFinder -i source.bin -e target.bin +sym
```
