    This tool is designed to search for private and public keys contained in binary files for ED448, ED25519, and SECP256K1. It can also extract certificates in DER format and provides an analysis of the cryptographic material contained in the file.<br>
    <br>
    Command Line Options:<br>
    <br>
    -i path|filename             Specifies the path or binary file to load (multiple files or folders accepted).<br>
    -r                           Enables recursive directory scanning.<br>
    -th number                   Sets the number of threads to use.<br>
    +-ed25519 ed448 secp256k1    Enables or disables specific algorithms (ED25519, ED448, SECP256K1).<br>
    -no_cross                    Disables cross scan, preventing the search for private/public keys in different files.<br>
    -analyze                     Searches for crypto materials within the file.<br>
    -analyze_only                Searches for crypto materials within the file only (no keys search).<br>
    -certs                       Searches for and extracts certificates.<br>
    -certs_only                  Searches for and extracts certificates only (no keys search).<br>
