# Client API

## Installation

Choose your preferred language:

{% tabs %}
{% tab title="Rust" %}
```bash
cargo add autonomi
```
{% endtab %}

{% tab title="Python" %}
```bash
pip install autonomi
```
{% endtab %}
{% endtabs %}

## Client Initialization

Initialize a client in read-only mode for browsing data, or with write capabilities for full access:

{% tabs %}
{% tab title="Rust" %}
```rust
use autonomi::{Client, ClientConfig};

// Initialize a local client
let client = Client::new_local().await?;

// Or initialize with configuration
let config = ClientConfig::default();
let client = Client::init_with_config(config).await?;
// Same as `Client::init`.
```
{% endtab %}

{% tab title="Python" %}
```python
from autonomi_client import Client

# Initialize a local client
client = await Client.init_local()

# Or initialize with custom config
config = ClientConfig.new()
config.network = Network(False)
client = await Client.init_with_config(config)
```
{% endtab %}
{% endtabs %}

## Core Data Types

Autonomi provides four fundamental data types that serve as building blocks for all network operations. For detailed information about each type, see the [Data Types Guide](../../core-concepts/data_types.md).

### 1. Chunk

Immutable, quantum-secure encrypted data blocks:

{% tabs %}
{% tab title="Rust" %}
```rust
use autonomi::Chunk;

// Store raw data as a chunk
let data = b"Hello, World!";
let chunk = client.store_chunk(data).await?;

// Retrieve chunk data
let retrieved = client.get_chunk(chunk.address()).await?;
assert_eq!(data, &retrieved[..]);

// Get chunk metadata
let metadata = client.get_chunk_metadata(chunk.address()).await?;
println!("Size: {}", metadata.size);
```
{% endtab %}

{% tab title="Python" %}
```python
from autonomi_client import Client, Wallet, Network

# Connect to the network
client = await Client.init_local()

# Wallet with private key that has funding on default local testnets
private_key = "0xac0974bec39a17e36ba4a6b4d238ff944bacb478cbed5efcae784d7bf4f2ff80"
wallet = Wallet.new_from_private_key(Network(True), private_key)

# Store raw data as a chunk
data = b"Hello, World!"
[cost, addr] = await client.chunk_put(data, PaymentOption.wallet(wallet))

# Retrieve chunk data
retrieved = await client.chunk_get(addr)
assert data == retrieved
```
{% endtab %}
{% endtabs %}

### 2. Pointer

Mutable references with version tracking:

{% tabs %}
{% tab title="Rust" %}
```rust
use autonomi::Pointer;

// Create a pointer to some data
let pointer = client.create_pointer(target_address).await?;

// Update pointer target
client.update_pointer(pointer.address(), new_target_address).await?;

// Resolve pointer to get current target
let target = client.resolve_pointer(pointer.address()).await?;

// Get pointer metadata and version
let metadata = client.get_pointer_metadata(pointer.address()).await?;
println!("Version: {}", metadata.version);
```
{% endtab %}

{% tab title="Python" %}
```python
from autonomi_client import Client, Network, Wallet, PaymentOption, SecretKey, PointerTarget, ChunkAddress, Pointer

# Wallet with private key that has funding on default local testnets
private_key = "0xac0974bec39a17e36ba4a6b4d238ff944bacb478cbed5efcae784d7bf4f2ff80"
wallet = Wallet.new_from_private_key(Network(True), private_key)

# Connect to the network
client = await Client.init_local()

# First, let's upload some data that we want to point to
target_data = b"Hello, I'm the target data!"
[cost, target_addr] = await client.data_put_public(target_data, PaymentOption.wallet(wallet))
print(f"Target data uploaded to: {target_addr}")

# Create a pointer target from the address
target = PointerTarget.new_chunk(ChunkAddress(target_addr))

# Create owner key pair
key = SecretKey()

# Estimate the cost of the pointer
cost = await client.pointer_cost(key.public_key())
print(f"pointer cost: {cost}")

# Create the pointer
pointer = Pointer(key, 0, target)
payment_option = PaymentOption.wallet(wallet)

# Create and store the pointer
pointer_addr = await client.pointer_put(pointer, payment_option)
print("Pointer stored successfully")

# Wait for the pointer to be stored by the network
await asyncio.sleep(1)

# Later, we can retrieve the pointer
pointer = await client.pointer_get(pointer_addr)
print(f"Retrieved pointer target: {pointer}")

# We can then use the target address to get the original data
retrieved_data = await client.data_get_public(pointer.target.hex)
print(f"Retrieved target data: {retrieved_data.decode()}")
```
{% endtab %}
{% endtabs %}

### 3. GraphEntry

Decentralized Graph structures for linked data:

{% tabs %}
{% tab title="Rust" %}
```rust
use autonomi::{
    client::{
        graph::{GraphEntry, GraphError},
        payment::PaymentOption,
    },
    Client,
};

let client = Client::init_local().await?;
let wallet = get_funded_wallet();

let key = bls::SecretKey::random();
let content = [0u8; 32];
let graph_entry = GraphEntry::new(&key, vec![], content, vec![]);

// estimate the cost of the graph_entry
let cost = client.graph_entry_cost(&key.public_key()).await?;
println!("graph_entry cost: {cost}");

// put the graph_entry
let payment_option = PaymentOption::from(&wallet);
client
    .graph_entry_put(graph_entry.clone(), payment_option)
    .await?;

// wait for the graph_entry to be replicated
tokio::time::sleep(tokio::time::Duration::from_secs(5)).await;

// check that the graph_entry is stored
let txs = client.graph_entry_get(&graph_entry.address()).await?;
assert_eq!(txs, graph_entry.clone());
```
{% endtab %}

{% tab title="Python" %}
```python
from autonomi_client import *
wallet = Wallet.new_from_private_key(Network(True), "0xac0974bec39a17e36ba4a6b4d238ff944bacb478cbed5efcae784d7bf4f2ff80")

# Connect to the network
client = await Client.init_local()

# Random key
key = SecretKey()
content = bytearray(32)
graph_entry = GraphEntry(key, [], content, [])

# estimate the cost of the graph_entry
cost = await client.graph_entry_cost(key.public_key())

# put the graph_entry
[cost, addr] = await client.graph_entry_put(graph_entry, PaymentOption.wallet(wallet))

# check that the graph_entry is stored
txs = await client.graph_entry_get(graph_entry.address())
assert txs == graph_entry
```
{% endtab %}
{% endtabs %}

### 4. ScratchPad

Unstructured data with CRDT properties:

{% tabs %}
{% tab title="Rust" %}
```rust
use autonomi::client::payment::PaymentOption;
use autonomi::scratchpad::ScratchpadError;
use autonomi::AttoTokens;
use autonomi::{
    client::scratchpad::{Bytes, Scratchpad},
    Client,
};

let client = Client::init_local().await?;
let wallet = todo!();

let key = bls::SecretKey::random();
let public_key = key.public_key();
let content = Bytes::from("Massive Array of Internet Disks");
let scratchpad = Scratchpad::new(&key, 42, &content, 0);

// estimate the cost of the scratchpad
let cost = client.scratchpad_cost(&public_key).await?;
println!("scratchpad cost: {cost}");

// put the scratchpad
let payment_option = PaymentOption::from(&wallet);
let (cost, addr) = client
    .scratchpad_put(scratchpad.clone(), payment_option)
    .await?;
assert_eq!(addr, *scratchpad.address());
println!("scratchpad put 1 cost: {cost}");

// wait for the scratchpad to be replicated
tokio::time::sleep(tokio::time::Duration::from_secs(5)).await;

// check that the scratchpad is stored
let got = client.scratchpad_get(&addr).await?;
assert_eq!(got, scratchpad.clone());
println!("scratchpad got 1");

// check that the content is decrypted correctly
let got_content = got.decrypt_data(&key)?;
assert_eq!(got_content, content);
```
{% endtab %}

{% tab title="Python" %}
```python
from autonomi_client import *

wallet = Wallet.new_from_private_key(Network(True), "0xac0974bec39a17e36ba4a6b4d238ff944bacb478cbed5efcae784d7bf4f2ff80")

# Connect to the network
client = await Client.init_local()

key = SecretKey()
content = b"Massive Array of Internet Disks"
scratchpad = Scratchpad(key, 42, content, 0)

# put the scratchpad
payment_option = PaymentOption.wallet(wallet)
(cost, addr) = await client.scratchpad_put(scratchpad, payment_option)
print(f"scratchpad address: {addr}")
print(f"scratchpad address: {scratchpad.address()}")
assert addr == scratchpad.address()

# check that the scratchpad is stored
got = await client.scratchpad_get(addr)
assert got == scratchpad

# check that the content is decrypted correctly
got_content = got.decrypt_data(key)
assert got_content == content
```
{% endtab %}
{% endtabs %}

<!-- ## Error Handling

Each language provides appropriate error handling mechanisms:

{% tabs %}
{% tab title="Rust" %}
```rust
use autonomi::error::{ChunkError, PointerError, GraphError, ScratchPadError};

// Handle chunk operations
match client.get_chunk(address).await {
    Ok(data) => process_data(data),
    Err(ChunkError::NotFound) => handle_missing(),
    Err(ChunkError::NetworkError(e)) => handle_network_error(e),
    Err(e) => handle_other_error(e),
}

// Handle pointer updates
match client.update_pointer(address, new_target).await {
    Ok(_) => println!("Update successful"),
    Err(PointerError::VersionConflict) => handle_conflict(),
    Err(e) => handle_other_error(e),
}
```
{% endtab %}

{% tab title="Python" %}
```python
from autonomi.errors import ChunkError, PointerError

# Handle chunk operations
try:
    data = client.get_chunk(address)
    process_data(data)
except ChunkError.NotFound:
    handle_missing()
except ChunkError.NetworkError as e:
    handle_network_error(e)
except Exception as e:
    handle_other_error(e)

# Handle pointer updates
try:
    client.update_pointer(address, new_target)
    print("Update successful")
except PointerError.VersionConflict:
    handle_conflict()
except Exception as e:
    handle_other_error(e)
```
{% endtab %}
{% endtabs %} -->

<!-- ## Advanced Usage

### Custom Types

{% tabs %}
{% tab title="Rust" %}
```rust
use serde::{Serialize, Deserialize};

#[derive(Serialize, Deserialize)]
struct MyData {
    field1: String,
    field2: u64,
}

// Store custom type in a scratchpad
let data = MyData {
    field1: "test".into(),
    field2: 42,
};
let pad = client.create_scratchpad(ContentType::Custom("MyData")).await?;
client.update_scratchpad(pad.address(), &data).await?;
```
{% endtab %}

{% tab title="Python" %}
```python
from dataclasses import dataclass

@dataclass
class MyData:
    field1: str
    field2: int

# Store custom type in a scratchpad
data = MyData(field1="test", field2=42)
pad = client.create_scratchpad(ContentType.CUSTOM("MyData"))
client.update_scratchpad(pad.address, data)
```
{% endtab %}
{% endtabs %}

### Encryption

{% tabs %}
{% tab title="Rust" %}
```rust
use autonomi::crypto::{encrypt_aes, decrypt_aes};

// Encrypt data before storage
let key = generate_aes_key();
let encrypted = encrypt_aes(data, &key)?;
let pad = client.create_scratchpad(ContentType::Encrypted).await?;
client.update_scratchpad(pad.address(), &encrypted).await?;

// Decrypt retrieved data
let encrypted = client.get_scratchpad(pad.address()).await?;
let decrypted = decrypt_aes(encrypted, &key)?;
```
{% endtab %}

{% tab title="Python" %}
```python
from autonomi.crypto import encrypt_aes, decrypt_aes

# Encrypt data before storage
key = generate_aes_key()
encrypted = encrypt_aes(data, key)
pad = client.create_scratchpad(ContentType.ENCRYPTED)
client.update_scratchpad(pad.address, encrypted)

# Decrypt retrieved data
encrypted = client.get_scratchpad(pad.address)
decrypted = decrypt_aes(encrypted, key)
```
{% endtab %}
{% endtabs %} -->

## Best Practices

1. **Data Type Selection**
   * Use Chunks for immutable data
   * Use Pointers for mutable references
   * Use GraphEntrys for ordered collections
   * Use ScratchPads for temporary data
2. **Error Handling**
   * Always handle network errors appropriately
   * Use type-specific error handling
   * Implement retry logic for transient failures
3. **Performance**
   * Use batch operations for multiple items
   * Consider chunking large data sets
   * Cache frequently accessed data locally
4. **Security**
   * Encrypt sensitive data before storage
   * Use secure key management
   * Validate data integrity

## Type System

{% tabs %}
{% tab title="Rust" %}
```rust
use autonomi::crypto::{encrypt_aes, decrypt_aes};

// Encrypt data before storage
let key = generate_aes_key();
let encrypted = encrypt_aes(data, &key)?;
let pad = client.create_scratchpad(ContentType::Encrypted).await?;
client.update_scratchpad(pad.address(), &encrypted).await?;

// Decrypt retrieved data
let encrypted = client.get_scratchpad(pad.address()).await?;
let decrypted = decrypt_aes(encrypted, &key)?;
```
{% endtab %}

{% tab title="Python" %}
```python
from typing import List, Optional, Union
from autonomi.types import Address, Data, Metadata

class Client:
    def store_chunk(self, data: bytes) -> Address: ...
    def get_chunk(self, address: Address) -> bytes: ...
    def create_pointer(self, target: Address) -> Pointer: ...
    def update_pointer(self, address: Address, target: Address) -> None: ...
```
{% endtab %}
{% endtabs %}

## Further Reading

* [Data Types Guide](../../core-concepts/data_types.md)
* [Local Network Setup](../../how-to-guides/local_network.md)
