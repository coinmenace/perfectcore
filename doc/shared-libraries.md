Shared Libraries
================

## perfectcoinconsensus

The purpose of this library is to make the verification functionality that is critical to PerfectCoin's consensus available to other applications, e.g. to language bindings.

### API

The interface is defined in the C header `perfectcoinconsensus.h` located in  `src/script/perfectcoinconsensus.h`.

#### Version

`perfectcoinconsensus_version` returns an `unsigned int` with the the API version *(currently at an experimental `0`)*.

#### Script Validation

`perfectcoinconsensus_verify_script` returns an `int` with the status of the verification. It will be `1` if the input script correctly spends the previous output `scriptPubKey`.

##### Parameters
- `const unsigned char *scriptPubKey` - The previous output script that encumbers spending.
- `unsigned int scriptPubKeyLen` - The number of bytes for the `scriptPubKey`.
- `const unsigned char *txTo` - The transaction with the input that is spending the previous output.
- `unsigned int txToLen` - The number of bytes for the `txTo`.
- `unsigned int nIn` - The index of the input in `txTo` that spends the `scriptPubKey`.
- `unsigned int flags` - The script validation flags *(see below)*.
- `perfectcoinconsensus_error* err` - Will have the error/success code for the operation *(see below)*.

##### Script Flags
- `perfectcoinconsensus_SCRIPT_FLAGS_VERIFY_NONE`
- `perfectcoinconsensus_SCRIPT_FLAGS_VERIFY_P2SH` - Evaluate P2SH ([BIP16](https://github.com/perfectcoin/bips/blob/master/bip-0016.mediawiki)) subscripts
- `perfectcoinconsensus_SCRIPT_FLAGS_VERIFY_DERSIG` - Enforce strict DER ([BIP66](https://github.com/perfectcoin/bips/blob/master/bip-0066.mediawiki)) compliance

##### Errors
- `perfectcoinconsensus_ERR_OK` - No errors with input parameters *(see the return value of `perfectcoinconsensus_verify_script` for the verification status)*
- `perfectcoinconsensus_ERR_TX_INDEX` - An invalid index for `txTo`
- `perfectcoinconsensus_ERR_TX_SIZE_MISMATCH` - `txToLen` did not match with the size of `txTo`
- `perfectcoinconsensus_ERR_DESERIALIZE` - An error deserializing `txTo`

### Example Implementations
- [NPerfectCoin](https://github.com/NicolasDorier/NPerfectCoin/blob/master/NPerfectCoin/Script.cs#L814) (.NET Bindings)
- [node-libperfectcoinconsensus](https://github.com/bitpay/node-libperfectcoinconsensus) (Node.js Bindings)
- [java-libperfectcoinconsensus](https://github.com/dexX7/java-libperfectcoinconsensus) (Java Bindings)
- [perfectcoinconsensus-php](https://github.com/Bit-Wasp/perfectcoinconsensus-php) (PHP Bindings)
