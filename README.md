# Cryptographic Material Search Tool by Luigi Origa


This tool is designed to search for private and public keys contained in binary files for ED448, ED25519, and SECP256K1. It can also extract certificates in DER format and provides an analysis of the cryptographic material contained in the file.

## Command Line Options

- `-i path|filename`             
  - Specifies the path or binary file to load (multiple files or folders accepted).
- `-r`                           
  - Enables recursive directory scanning.
- `-th number`                   
  - Sets the number of threads to use.
- `+-ed25519 ed448 secp256k1 rsa128 rsa256 rsa512 rsa1024 rsa2048 rsa4096`    
  - Enables or disables specific algorithms (ED25519, ED448, SECP256K1, RSA...).
- `-no_cross`                    
  - Disables cross scan, preventing the search for private/public keys in different files.
- `-analyze`                     
  - Searches for crypto materials within the file.
- `-analyze_only`                
  - Searches for crypto materials within the file only (no keys search).
- `-certs`                       
  - Searches for and extracts certificates.
- `-certs_only`                  
  - Searches for and extracts certificates only (no keys search).
