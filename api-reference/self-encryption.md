# Self Encryption

A file content self-encryptor that provides convergent encryption on file-based data. It produces a `DataMap` type and several chunks of encrypted data. Each chunk is up to 4MB in size and has an index and a name (SHA3-256 hash of the content), allowing chunks to be self-validating.

# Quick Start Guide

## Installation

{% tabs %}
{% tab title="Rust" %}
```console
cargo add self_encryption
```
{% endtab %}

{% tab title="Python" %}
```bash
pip install self-encryption
```
{% endtab %}
{% endtabs %}

{% tab title="Node.js" %}
```console
npm install @withautonomi/self-encryption
```
{% endtab %}
{% endtabs %}

## Simple Example

{% tabs %}
{% tab title="Rust" %}
```rust
use self_encryption::{bytes::Bytes, decrypt, encrypt};

fn main() -> self_encryption::Result<()> {
    // Encrypt bytes in memory
    let data = Bytes::from("Small data to encrypt");
    let (data_map, encrypted_chunks) = encrypt(data.clone())?;

    // Decrypt using the data map and chunks
    let decrypted = decrypt(&data_map, &encrypted_chunks)?;

    // Original data and decrypted data will be the same
    assert_eq!(data, decrypted);

    Ok(())
}

```
{% endtab %}

{% tab title="Python" %}
```python
from self_encryption import encrypt, decrypt

# Encrypt bytes in memory
data = b"Small data to encrypt"
data_map, encrypted_chunks = encrypt(data)

# Decrypt using the data map and chunks
decrypted = decrypt(data_map, encrypted_chunks)
```
{% endtab %}

{% tab title="Node.js" %}
```ts
import { encrypt, decrypt } from '@withautonomi/self-encryption'
import assert from 'assert'

const data = Buffer.from("Hello, World!");
const { dataMap, chunks } = encrypt(data)
const dataDecrypted = decrypt(dataMap, chunks)
assert(data.equals(dataDecrypted))
```
{% endtab %}
{% endtabs %}

## Core Concepts

### DataMap

The data map holds information required to recover the content of the encrypted data. It contains a vector of chunk 'infos', a list of file's chunk hashes. Only data larger than 3 bytes can be self-encrypted.

### Chunk Sizes

* `MIN_CHUNK_SIZE`: 1 byte
* `MAX_CHUNK_SIZE`: Defaults to 1 MiB but Autonomi sets this to 4 MiB
* `MIN_ENCRYPTABLE_BYTES`: 3 bytes

## Streaming Operations

Some functions use streaming to reduce the memory footprint of the encryption operation. These functions take callbacks that store or retrieve chunks.

### Streaming File Encryption

An example is to use an in-memory data type, like the `HashMap` in Rust:

{% tabs %}
{% tab title="Rust" %}
```rust
use self_encryption::{
    Result, XorName, bytes::Bytes, streaming_decrypt_from_storage,
    streaming_encrypt_from_file,
};
use std::{
    collections::HashMap,
    path::Path,
    sync::{Arc, Mutex},
};

let storage = Arc::new(Mutex::new(HashMap::new()));

let storage_clone = Arc::clone(&storage);
let store = move |hash: XorName, content: Bytes| -> Result<()> {
    // Store encrypted chunk into the map by its hash.
    let _ = storage_clone.lock().unwrap().insert(hash, content);
    Ok(())
};

// Encrypt the data that lives in the file and store it into the `HashMap`.
let file_path = Path::new("Cargo.toml");
let data_map = streaming_encrypt_from_file(file_path, store).unwrap();

let fetch = |hashes: &[XorName]| -> Result<Vec<Bytes>> {
    Ok(hashes
        .iter()
        // Take the hashes and map them to their encrypted contents
        .map(|hash| {
            storage
                .lock()
                .expect("lock should not be poisoned")
                .get(hash)
                .expect("hash should be in storage")
                .clone()
        })
        .collect())
};
streaming_decrypt_from_storage(
    &data_map,
    Path::new("Cargo.toml.decrypted"),
    fetch,
)
.unwrap();

// Verify that the decrypted file matches the original
assert_eq!(
    std::fs::read(file_path).unwrap(),
    std::fs::read("Cargo.toml.decrypted").unwrap()
);
```
{% endtab %}
{% endtabs %}
