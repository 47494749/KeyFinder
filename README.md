# Keys Search Tool by Luigi Origa

This tool is designed to search for cryptographic keys and secrets contained in binary files. It supports elliptic curve algorithms (ED25519, ED448, SECP256K1, SECP256R1, SECP384R1, SECP521R1), RSA private keys (128 to 4096 bit), AES key schedules (128/256), and ECDH key pairs. It can also extract certificates in DER format, perform deep scans for crypto artifacts, and provides an analysis of the cryptographic material contained in the file. Supports multi-threaded CPU search and GPU acceleration via OpenCL.

## Command Line Options

- `-h`
  - Shows help.
- `-i path|filename`
  - Specifies the path or binary file to load (multiple `-i` accepted).
- `-r`
  - Enables recursive directory scanning.
- `-th number`
  - Sets the number of threads to use.
- `+-ed25519 ed448 secp256k1 secp256r1 secp384r1 secp521r1 aes rsa128 rsa256 rsa512 rsa1024 rsa2048 rsa4096`
  - Enables (`+`) or disables (`-`) specific algorithms.
  - **Default ON:** ed25519, ed448, aes
  - **Default OFF:** secp256k1, secp256r1, secp384r1, secp521r1, rsa128, rsa256, rsa512, rsa1024, rsa2048, rsa4096
- `-no_cross`
  - Disables cross scan (searching for matching private/public keys across different files).
- `-no_gpu`
  - Disables GPU (OpenCL) acceleration.
- `-analyze`
  - Searches for common cryptographic constants within the file.
- `-analyze_only`
  - Searches for common constants only (no key search).
- `-certs`
  - Searches for and extracts certificates.
- `-certs_only`
  - Searches for and extracts certificates only (no key search).
- `-deep_scan`
  - Deep scan for keys, secrets and crypto artifacts.
- `-deep_scan_only`
  - Deep scan only (no key search).
- `-clean_db`
  - Removes all pattern/key/prime sidecar files.
- `-ignore_excluded`
  - Ignores excluded file extensions.
- `-xe ext`
  - Excludes a file extension (can use multiple `-xe`).
- `-xd dir`
  - Excludes a directory (can use multiple `-xd`).

## Default Excluded Extensions

`txt`, `jpg`, `bmp`, `ico`, `xls`, `doc`, `hex`, `bak`, `idb`, `idc`, `py`

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
