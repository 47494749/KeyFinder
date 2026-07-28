# KeyFinder Changelog

## V1.08 (2026-07-28)

### Trial Decrypt — CBC and CTR Mode Support
- Added `-mode ecb|cbc|ctr` to select cipher mode for trial decrypt
- Added `-iv <hex>` for CBC initialization vector (16 bytes) or CTR nonce (12 bytes)
- Added `-ctr_start <number>` for CTR initial counter value
- CBC mode supported for all 16-byte block ciphers (AES, Camellia, SM4, Serpent, Twofish, ARIA, SEED)
- CTR mode supported for AES-128 and AES-256
- GPU OpenCL kernel prepared for CBC/CTR trial decrypt acceleration

### Trial Decrypt — Multi-cipher support
- Added trial decrypt support for: Camellia-128/256, SM4, Serpent-256, Twofish-256, ARIA-128/256, SEED, DES, 3DES, IDEA, CAST-128
- Previously only AES-128/256 ECB was supported

### Bug Fixes
- Fixed 3DES EDE decrypt using wrong key order (dead code removed)
- Fixed `NumToHex` redefinition conflict between myDisk.h and tools/compat.h
- Removed unused `symmetric/aes.h` include from trialDecrypt.cpp

### CLI
- Reorganized help output into sections (General, Trial Decrypt, Algorithms, Scan Modes, Filters)
- Added `-?` alias for `-h` in help documentation
- Documented trial decrypt supported ciphers per mode in help text

---

## V1.07 (2026-05-04)

### New Algorithms
- X25519 key pair search (CPU + GPU)
- X448 key pair search (CPU + GPU)
- Brainpool curves: BP256R1, BP384R1, BP512R1 (CPU + GPU)
- DSA-1024, DSA-2048, DSA-3072 key search (CPU + GPU)
- ChaCha20 key schedule detection

### Algorithm Groups
- Added `+asym` / `-asym` toggle for all asymmetric algorithms
- Added `+sym` / `-sym` toggle for all symmetric algorithms
- Added individual toggles for all new algorithms

### GPU Enhancements
- New OpenCL kernels: X25519, X448, BP256R1, BP384R1, BP512R1, Serpent, Twofish, Camellia, ARIA, SM4
- GPU kernel cache system (`%APPDATA%\KeyFinder\kernels`) with binary caching per device
- All GPU kernel versions bumped to 1.3.0+

### Test Infrastructure
- Test vector verification for all new algorithms
- Fixed incorrect test vectors for ED25519, ED448, SECP curves

---

## V1.05 (2026-04-19)

### Symmetric Key Schedule Search
- AES-128/256 expanded key schedule detection (CPU + GPU)
- DES, Camellia, SM4, Serpent, IDEA, Twofish, SEED, CAST-128, ARIA, RC5/RC6 key schedule detection

### Deep Scan
- Added `-deep_scan` / `-deep_scan_only` for structural crypto artifact detection
- PGP key packet validation
- SSH key format detection
- Entropy-based secret detection

### Analysis Enhancements
- Added 12 new crypto constant tables: MD5, DES S-boxes, ChaCha20, SM3, SM4, Keccak, etc.
- Improved certificate extraction with reduced false positives

### Key Report
- End-of-run summary report with per-file and total key counts
- SHA-256 file deduplication to avoid re-scanning identical content
- Per-algorithm completion caching via `.sha256` sidecar files

### RSA
- Fixed `std::bad_alloc` in RSA pair generation for large files
- N-PHI pair search method (find RSA keys without needing P,Q primes)
- Empty marker file caching for unsuccessful RSA prime searches

### GPU
- SECP256K1, SECP256R1, SECP384R1, SECP521R1 GPU acceleration
- Precomputed G_TABLE optimization for all SECP OpenCL kernels
- Fixed SECP521R1 GPU kernel performance issue
- Fixed address space conversion errors in SECP kernels

---

## V1.04 (2026-04-02)

### Core
- ED25519 private/public key pair search (CPU + GPU)
- ED448 private/public key pair search (CPU + GPU)
- SECP256K1, SECP256R1, SECP384R1, SECP521R1 key pair search
- RSA-128 through RSA-4096 prime factor search
- Cross-file key comparison (`-no_cross` to disable)
- Certificate extraction (`-certs`, `-certs_only`)
- Crypto constant analysis (`-analyze`, `-analyze_only`)
- Multi-threaded CPU with configurable thread count (`-th`)
- GPU acceleration via OpenCL for ED25519, ED448
- Recursive directory scanning (`-r`)
- Extension and directory exclusion (`-xe`, `-xd`)
- Sidecar database cleanup (`-clean_db`)
