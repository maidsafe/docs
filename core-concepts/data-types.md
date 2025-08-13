# Data Types

This page provides detailed information about the core data types used in the Autonomi Client API.

## Native Data Types

The Autonomi Network supports four fundamental data types, known as native data types. These form the building blocks for all data storage within the network.

On top of these native types, higher-level abstractions can be constructed. The Autonomi API, for example, provides features such as arbitrary-length byte storage, files, vaults, and registers—each built upon these core data types.

Below is a detailed description of the native data types.

### Chunk

A Chunk is a 4MB block of raw bytes that serves as a fundamental unit of storage in the Autonomi Network. It is content-addressed, meaning its address is derived from its content hash. Chunks are immutable, ensuring that once stored, their data cannot be modified. They are also self-verifiable, as their integrity can be confirmed by computing the hash and comparing it to the address.

More on Chunks in the [Chunk API Reference](../api-reference/autonomi-client/chunks.md)

### Pointer

A Pointer is a mutable reference to any native data type on the Autonomi Network. It provides an indirection mechanism for referencing data stored elsewhere on the network.

**Key Characteristics:**
- **Key-addressed:** The network address is derived from a BLS public key, meaning each public key can only have one pointer
- **Generic target:** Can reference any native data type including Chunks, GraphEntries, Scratchpads, and other Pointers
- **Pay-once, free updates:** Initial creation requires payment, but subsequent updates are free
- **Versioned & signed:** Includes a version counter (u64) and cryptographically verifiable signatures
- **Owner-controlled:** Only the owner of the public key can update the pointer
- **CRDT-like behavior:** Only the latest version (highest counter) is kept on the network

**Structure:**
- `owner`: The BLS public key that owns this pointer
- `counter`: Version number (u64) that increases with each update
- `target`: The address being pointed to (PointerTarget enum)
- `signature`: Cryptographic signature ensuring authenticity

**Target Types:**
- `ChunkAddress`: Points to a chunk of data
- `GraphEntryAddress`: Points to a graph entry
- `PointerAddress`: Points to another pointer (enabling pointer chains)
- `ScratchpadAddress`: Points to a scratchpad

Pointers are particularly useful for creating mutable references to immutable data, building linked data structures, and implementing dynamic content management systems.

#### Basic Usage Examples

**Rust Example:**
```rust
use autonomi::{Client, SecretKey, PublicKey};
use autonomi::client::payment::PaymentOption;
use autonomi::client::pointer::{Pointer, PointerTarget};
use autonomi::chunk::ChunkAddress;
use autonomi::AttoTokens;
use eyre::Result;

async fn basic_pointer_example() -> Result<()> {
    // Initialize client and wallet
    let client = Client::init_local().await?;
    let wallet = get_funded_wallet();
    let payment_option = PaymentOption::from(&wallet);

    // Create a secret key for the pointer owner
    let owner_key = SecretKey::random();
    let public_key = owner_key.public_key();

    // Create a target (e.g., a chunk address)
    let chunk_addr = ChunkAddress::new(XorName::random(&mut rand::thread_rng()));
    let target = PointerTarget::ChunkAddress(chunk_addr);

    // Estimate the cost
    let estimated_cost = client.pointer_cost(&public_key).await?;
    println!("Estimated pointer cost: {estimated_cost}");

    // Create the pointer
    let (actual_cost, pointer_addr) = client
        .pointer_create(&owner_key, target.clone(), payment_option)
        .await?;
    println!("Pointer created at {pointer_addr:?} with cost: {actual_cost}");

    // Wait for replication
    tokio::time::sleep(tokio::time::Duration::from_secs(5)).await;

    // Retrieve the pointer
    let retrieved_pointer = client.pointer_get(&pointer_addr).await?;
    println!("Retrieved pointer: {retrieved_pointer:?}");

    // Verify the pointer signature
    if retrieved_pointer.verify_signature() {
        println!("Pointer signature is valid!");
    }

    Ok(())
}
```

**Python Example:**
```python
from autonomi_client import Client, Network, Wallet, PaymentOption, SecretKey, PointerTarget, ChunkAddress, Pointer
import asyncio

async def basic_pointer_example():
    # Initialize client and wallet
    private_key = "0xac0974bec39a17e36ba4a6b4d238ff944bacb478cbed5efcae784d7bf4f2ff80"
    network = Network(True)
    wallet = Wallet.new_from_private_key(network, private_key)
    client = await Client.init_local()

    # Upload target data
    target_data = b"Hello, I'm the target data!"
    [cost, target_addr] = await client.data_put_public(target_data, PaymentOption.wallet(wallet))
    
    # Create pointer
    key = SecretKey()
    target = PointerTarget.new_chunk(ChunkAddress(target_addr))
    pointer = Pointer(key, 0, target)
    pointer_addr = await client.pointer_put(pointer, PaymentOption.wallet(wallet))
    
    # Retrieve and use pointer
    retrieved_pointer = await client.pointer_get(pointer_addr)
    retrieved_data = await client.data_get_public(retrieved_pointer.target.hex)
    print(f"Retrieved: {retrieved_data.decode()}")
```

**Node.js Example:**
```javascript
import init, * as atnm from '../pkg/autonomi.js';

async function basicPointerExample() {
    // Initialize client and wallet
    await init();
    const client = await atnm.Client.connect([window.peer_addr]);
    const wallet = atnm.getFundedWallet();
    
    // Upload target data
    const targetData = new TextEncoder().encode("Hello, I'm the target data!");
    const targetAddr = await client.putData(targetData, wallet);
    
    // Create pointer
    const key = atnm.genSecretKey();
    const target = atnm.PointerTarget.newChunk(targetAddr);
    const pointer = new atnm.Pointer(key, 0, target);
    const pointerAddr = await client.putPointer(pointer, wallet);
    
    // Retrieve and use pointer
    const retrievedPointer = await client.getPointer(pointerAddr);
    const retrievedData = await client.getData(retrievedPointer.target.hex);
    const decodedData = new TextDecoder().decode(retrievedData);
    console.log(`Retrieved: ${decodedData}`);
}
```

More on Pointers in the [Pointer API Reference](../api-reference/autonomi-client/pointer.md)

### Scratchpad

A Scratchpad is a 4MB mutable storage space on the Autonomi Network. It follows a pay-once model, allowing unlimited updates for free. To maintain consistency, it includes a version counter, ensuring the latest version is always the one kept on the Network. Scratchpads are owned by a public key, with their data stored at the owner’s address. Each update is signed by the owner, making it cryptographically verifiable.

More on Scratchpads in the [Scratchpad API Reference](../api-reference/autonomi-client/scratchpad.md)

### GraphEntry

A GraphEntry is a fundamental unit in a generic directed graph within the Autonomi Network, built from multiple interconnected entries. It is immutable and can store 32 bytes of data per entry. Each GraphEntry can reference parent entries using their public keys and descendant entries along with 32 bytes of metadata per descendant, also identified by their public keys. The entry is owned by a public key, stored at the owner’s address, and signed by the owner, ensuring cryptographic verification.

More on GraphEntries in the [GraphEntry API Reference](../api-reference/autonomi-client/graphentry.md)

## High-Level Data Types

The API provides high-level data APIs and types that offer a more intuitive and accessible abstraction over the native data types. These higher-level constructs simplify usage while leveraging the native data types as their foundation. Below is a description of these data types.

### Regular Data Storage

Regular data storage allows storing arbitrary-length data securely on the Autonomi Network. It uses self-encryption, a strong encryption mechanism that ensures data is fragmented into encrypted Chunks and distributed across the network. This storage supports both public data (accessible to anyone) and private data (restricted to the owner). It provides immutable, permanent storage, where data remains unchanged once uploaded. Users pay once for the upload, and the data is preserved forever.

More on Regular Data Storage in the [Data API Reference](../api-reference/autonomi-client/data.md)

### Public Archive

A Public Archive is a structure that maps file paths to data addresses on the network. While it can be used to store a single file with its metadata, it is generally used to organize multiple files in a hierarchical structure to simulate directories. Public Archives support nested paths and store metadata (creation time, modification time, size) for each file. Public Archives are stored on the network as Chunks.

### Private Archive

A Private Archive provides enhanced privacy by storing data locally rather than on the network. Unlike Public Archives which reference files through their network addresses, Private Archives contain the complete data maps within the archive itself. Like Public Archives, they support nested paths to simulate directories and store metadata for each file, but all this information stays local to the owner.

### Register

A Register is a 32-byte mutable memory register that allows storing and updating small pieces of data. Unlike immutable storage, Registers support updates, but each modification requires a payment. The entire update history is preserved, enabling users to traverse past versions when needed.

More on Registers in the [Register API Reference](../api-reference/autonomi-client/register.md)

### Vault

A Vault is a secure, encrypted storage system that helps users manage their data on the Autonomi Network. It allows storing owned Registers, Register keys, and references to file archives (both public and private). Users pay once for a fixed storage size and can update their Vault unlimited times for free. If more space is needed, Vault size can be upgraded at any time with an additional payment. With a Vault, users only need to retain a single key to access all their stored data on the network, simplifying data management and security.
