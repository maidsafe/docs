# Pointer

Pointers are native data types in the Autonomi Network that provide an indirection mechanism for referencing any data on the network.

## Overview

* **Key-addressed:** Network address is derived from a BLS public key
* **Generic target:** Can reference any native data type including pointers
* **Pay-once, free updates:** Unlimited updates for free after initial creation
* **Versioned & signed:** Includes a version counter and cryptographically verifiable signatures
* **Owner-controlled:** Only the owner can update the pointer

## Data Structure

```rust
pub struct Pointer {
    owner: PublicKey,      // BLS public key that owns this pointer
    counter: u64,          // Version number that increases with each update
    target: PointerTarget, // The address being pointed to
    signature: Signature,  // Cryptographic signature ensuring authenticity
}

pub enum PointerTarget {
    ChunkAddress(ChunkAddress),
    GraphEntryAddress(GraphEntryAddress),
    PointerAddress(PointerAddress),
    ScratchpadAddress(ScratchpadAddress),
}
```

## Client Methods

### `pointer_get(address: &PointerAddress) -> Result<Pointer, PointerError>`
Retrieves a pointer from the network by its address.

### `pointer_create(owner: &SecretKey, target: PointerTarget, payment_option: PaymentOption) -> Result<(AttoTokens, PointerAddress), PointerError>`
Creates and uploads a new pointer for a payment. Returns the total cost and the pointer's address.

**Note:** Each public key can only have one pointer. If a pointer already exists at the address, this will fail.

### `pointer_update(owner: &SecretKey, target: PointerTarget) -> Result<(), PointerError>`
Updates an existing pointer with a new target. This operation is free as the pointer was already paid for at creation.

**Note:** The pointer must exist before it can be updated. Only the latest version (highest counter) is kept on the network.

### `pointer_cost(key: &PublicKey) -> Result<AttoTokens, CostError>`
Estimates the storage cost for creating a pointer.

### `pointer_put(pointer: Pointer, payment_option: PaymentOption) -> Result<(AttoTokens, PointerAddress), PointerError>`
Manually uploads a pointer to the network for a payment. Returns the total cost and the pointer's address.

**Warning:** Only use this when you know what you are doing. For most use cases, prefer `pointer_create` and `pointer_update`.

### `pointer_check_existence(address: &PointerAddress) -> Result<bool, PointerError>`
Checks if a pointer exists on the network. This is much faster than `pointer_get` but may fail if called immediately after creating a pointer.

## Error Types

```rust
pub enum PointerError {
    PutError(PutError),                    // Failed to put pointer
    GetError(GetError),                    // Failed to get pointer
    Serialization,                         // Serialization error
    Corrupt(String),                       // Pointer record corrupt
    BadSignature,                          // Pointer signature is invalid
    Pay(PayError),                         // Payment failure
    Wallet(EvmWalletError),                // Failed to retrieve wallet payment
    InvalidQuote,                          // Received invalid quote from node
    PointerAlreadyExists(PointerAddress),  // Pointer already exists at address
    CannotUpdateNewPointer,                // Cannot update non-existent pointer
}
```

## Examples

### Basic Pointer Creation and Update

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

    // Update the pointer to point to itself (creating a self-reference)
    let new_target = PointerTarget::PointerAddress(pointer_addr);
    client.pointer_update(&owner_key, new_target.clone()).await?;

    // Wait for replication
    tokio::time::sleep(tokio::time::Duration::from_secs(5)).await;

    // Verify the update
    let updated_pointer = client.pointer_get(&pointer_addr).await?;
    assert_eq!(updated_pointer.counter(), 1); // Version should be incremented
    assert_eq!(updated_pointer.target(), &new_target);

    Ok(())
}
```

**Python Example:**
```python
from autonomi_client import Client, Network, Wallet, PaymentOption, SecretKey, PointerTarget, ChunkAddress, Pointer
import asyncio

async def basic_pointer_example():
    """Basic example of creating and using a pointer."""
    print("=== Basic Pointer Example ===")
    
    # Initialize a wallet with a private key
    private_key = "0xac0974bec39a17e36ba4a6b4d238ff944bacb478cbed5efcae784d7bf4f2ff80"
    network = Network(True)
    wallet = Wallet.new_from_private_key(network, private_key)
    client = await Client.init_local()

    # Upload some data that we want to point to
    target_data = b"Hello, I'm the target data!"
    [cost, target_addr] = await client.data_put_public(target_data, PaymentOption.wallet(wallet))
    print(f"Target data uploaded to: {target_addr}")

    # Create a pointer target from the address
    target = PointerTarget.new_chunk(ChunkAddress(target_addr))
    
    # Create owner key pair
    key = SecretKey()

    # Estimate the cost of the pointer
    cost = await client.pointer_cost(key.public_key())
    print(f"Pointer cost: {cost}")

    # Create the pointer
    pointer = Pointer(key, 0, target)
    payment_option = PaymentOption.wallet(wallet)
    
    # Create and store the pointer
    pointer_addr = await client.pointer_put(pointer, payment_option)
    print(f"Pointer stored successfully at: {pointer_addr}")

    # Wait for the pointer to be stored by the network
    await asyncio.sleep(2)

    # Retrieve the pointer
    retrieved_pointer = await client.pointer_get(pointer_addr)
    print(f"Retrieved pointer: {retrieved_pointer}")

    # Get the data the pointer points to
    retrieved_data = await client.data_get_public(retrieved_pointer.target.hex)
    print(f"Retrieved target data: {retrieved_data.decode()}")
```

**Node.js Example:**
```javascript
import init, * as atnm from '../pkg/autonomi.js';

async function basicPointerExample() {
    console.log("=== Basic Pointer Example ===");
    
    // Initialize client and wallet
    await init();
    atnm.logInit("ant-networking=warn,autonomi=trace");
    const client = await atnm.Client.connect([window.peer_addr]);
    const wallet = atnm.getFundedWallet();
    
    console.log(`Wallet address: ${wallet.address()}`);
    console.log(`Wallet balance: ${await wallet.balance()}`);

    // Upload some data that we want to point to
    const targetData = new TextEncoder().encode("Hello, I'm the target data!");
    const targetAddr = await client.putData(targetData, wallet);
    console.log(`Target data uploaded to: ${targetAddr}`);

    // Create a secret key for the pointer owner
    const key = atnm.genSecretKey();
    const publicKey = key.publicKey();
    
    // Estimate the cost of the pointer
    const cost = await client.getPointerCost(publicKey);
    console.log(`Pointer cost: ${cost}`);

    // Create a pointer target from the address
    const target = atnm.PointerTarget.newChunk(targetAddr);
    
    // Create the pointer
    const pointer = new atnm.Pointer(key, 0, target);
    
    // Create and store the pointer
    const pointerAddr = await client.putPointer(pointer, wallet);
    console.log(`Pointer stored successfully at: ${pointerAddr}`);

    // Wait for the pointer to be stored by the network
    await new Promise(resolve => setTimeout(resolve, 2000));

    // Retrieve the pointer
    const retrievedPointer = await client.getPointer(pointerAddr);
    console.log(`Retrieved pointer: ${retrievedPointer}`);

    // Get the data the pointer points to
    const retrievedData = await client.getData(retrievedPointer.target.hex);
    const decodedData = new TextDecoder().decode(retrievedData);
    console.log(`Retrieved target data: ${decodedData}`);
}
```

### Pointer Update Example

**Rust Example:**
```rust
async fn pointer_update_example() -> Result<()> {
    let client = Client::init_local().await?;
    let wallet = get_funded_wallet();
    let payment_option = PaymentOption::from(&wallet);

    // Create owner key
    let key = SecretKey::random();
    
    // Upload first target data
    let data1 = b"First target data";
    let (_, addr1) = client.data_put_public(data1, payment_option.clone()).await?;
    let target1 = PointerTarget::ChunkAddress(ChunkAddress::from_content(data1));
    
    // Create pointer pointing to first data
    let (_, pointer_addr) = client
        .pointer_create(&key, target1, payment_option.clone())
        .await?;
    println!("Created pointer at: {pointer_addr:?}");
    
    // Wait for replication
    tokio::time::sleep(tokio::time::Duration::from_secs(5)).await;
    
    // Upload second target data
    let data2 = b"Second target data";
    let target2 = PointerTarget::ChunkAddress(ChunkAddress::from_content(data2));
    
    // Update pointer to point to second data
    client.pointer_update(&key, target2).await?;
    println!("Pointer updated successfully");
    
    // Wait for replication
    tokio::time::sleep(tokio::time::Duration::from_secs(5)).await;
    
    // Retrieve and verify the updated pointer
    let final_pointer = client.pointer_get(&pointer_addr).await?;
    println!("Updated pointer counter: {}", final_pointer.counter());

    Ok(())
}
```

**Python Example:**
```python
async def pointer_update_example():
    """Example of updating a pointer to point to different data."""
    print("\n=== Pointer Update Example ===")
    
    # Initialize client and wallet
    private_key = "0xac0974bec39a17e36ba4a6b4d238ff944bacb478cbed5efcae784d7bf4f2ff80"
    network = Network(True)
    wallet = Wallet.new_from_private_key(network, private_key)
    client = await Client.init_local()

    # Create owner key
    key = SecretKey()
    
    # Upload first target data
    data1 = b"First target data"
    [cost, addr1] = await client.data_put_public(data1, PaymentOption.wallet(wallet))
    target1 = PointerTarget.new_chunk(ChunkAddress(addr1))
    
    # Create pointer pointing to first data
    pointer = Pointer(key, 0, target1)
    payment_option = PaymentOption.wallet(wallet)
    pointer_addr = await client.pointer_put(pointer, payment_option)
    print(f"Created pointer at: {pointer_addr}")
    
    # Wait for replication
    await asyncio.sleep(2)
    
    # Upload second target data
    data2 = b"Second target data"
    [cost, addr2] = await client.data_put_public(data2, PaymentOption.wallet(wallet))
    target2 = PointerTarget.new_chunk(ChunkAddress(addr2))
    
    # Update pointer to point to second data
    updated_pointer = Pointer(key, 1, target2)
    await client.pointer_put(updated_pointer, payment_option)
    print("Pointer updated successfully")
    
    # Wait for replication
    await asyncio.sleep(2)
    
    # Retrieve and verify the updated pointer
    final_pointer = await client.pointer_get(pointer_addr)
    print(f"Updated pointer counter: {final_pointer.counter}")
    
    # Get the data the pointer now points to
    final_data = await client.data_get_public(final_pointer.target.hex)
    print(f"Final target data: {final_data.decode()}")
```

**Node.js Example:**
```javascript
async function pointerUpdateExample() {
    console.log("\n=== Pointer Update Example ===");
    
    // Initialize client and wallet
    await init();
    atnm.logInit("ant-networking=warn,autonomi=trace");
    const client = await atnm.Client.connect([window.peer_addr]);
    const wallet = atnm.getFundedWallet();

    // Create owner key
    const key = atnm.genSecretKey();
    
    // Upload first target data
    const data1 = new TextEncoder().encode("First target data");
    const addr1 = await client.putData(data1, wallet);
    const target1 = atnm.PointerTarget.newChunk(addr1);
    
    // Create pointer pointing to first data
    const pointer = new atnm.Pointer(key, 0, target1);
    const pointerAddr = await client.putPointer(pointer, wallet);
    console.log(`Created pointer at: ${pointerAddr}`);
    
    // Wait for replication
    await new Promise(resolve => setTimeout(resolve, 2000));
    
    // Upload second target data
    const data2 = new TextEncoder().encode("Second target data");
    const addr2 = await client.putData(data2, wallet);
    const target2 = atnm.PointerTarget.newChunk(addr2);
    
    // Update pointer to point to second data
    const updatedPointer = new atnm.Pointer(key, 1, target2);
    await client.putPointer(updatedPointer, wallet);
    console.log("Pointer updated successfully");
    
    // Wait for replication
    await new Promise(resolve => setTimeout(resolve, 2000));
    
    // Retrieve and verify the updated pointer
    const finalPointer = await client.getPointer(pointerAddr);
    console.log(`Updated pointer counter: ${finalPointer.counter}`);
    
    // Get the data the pointer now points to
    const finalData = await client.getData(finalPointer.target.hex);
    const decodedFinalData = new TextDecoder().decode(finalData);
    console.log(`Final target data: ${decodedFinalData}`);
}
```

### Pointer Chain Example

**Rust Example:**
```rust
async fn pointer_chain_example() -> Result<()> {
    let client = Client::init_local().await?;
    let wallet = get_funded_wallet();
    let payment_option = PaymentOption::from(&wallet);

    // Create multiple keys for different pointers
    let key1 = SecretKey::random();
    let key2 = SecretKey::random();
    let key3 = SecretKey::random();

    // Create a chunk as the final target
    let chunk_addr = ChunkAddress::new(XorName::random(&mut rand::thread_rng()));
    
    // Create pointer3 pointing to the chunk
    let (_, addr3) = client
        .pointer_create(&key3, PointerTarget::ChunkAddress(chunk_addr), payment_option.clone())
        .await?;

    // Create pointer2 pointing to pointer3
    let (_, addr2) = client
        .pointer_create(&key2, PointerTarget::PointerAddress(addr3), payment_option.clone())
        .await?;

    // Create pointer1 pointing to pointer2
    let (_, addr1) = client
        .pointer_create(&key1, PointerTarget::PointerAddress(addr2), payment_option)
        .await?;

    // Wait for replication
    tokio::time::sleep(tokio::time::Duration::from_secs(5)).await;

    // Follow the chain: pointer1 -> pointer2 -> pointer3 -> chunk
    let ptr1 = client.pointer_get(&addr1).await?;
    let ptr2 = client.pointer_get(&ptr1.target().as_pointer_address().unwrap()).await?;
    let ptr3 = client.pointer_get(&ptr2.target().as_pointer_address().unwrap()).await?;
    
    // Verify we reach the original chunk
    assert!(matches!(ptr3.target(), PointerTarget::ChunkAddress(_)));

    Ok(())
}
```

**Python Example:**
```python
async def pointer_chain_example():
    """Example of creating a chain of pointers."""
    print("\n=== Pointer Chain Example ===")
    
    # Initialize client and wallet
    private_key = "0xac0974bec39a17e36ba4a6b4d238ff944bacb478cbed5efcae784d7bf4f2ff80"
    network = Network(True)
    wallet = Wallet.new_from_private_key(network, private_key)
    client = await Client.init_local()

    # Create multiple keys for different pointers
    key1 = SecretKey()
    key2 = SecretKey()
    key3 = SecretKey()
    
    # Upload final target data
    final_data = b"Final target in the chain"
    [cost, final_addr] = await client.data_put_public(final_data, PaymentOption.wallet(wallet))
    final_target = PointerTarget.new_chunk(ChunkAddress(final_addr))
    
    payment_option = PaymentOption.wallet(wallet)
    
    # Create pointer3 pointing to the final data
    pointer3 = Pointer(key3, 0, final_target)
    addr3 = await client.pointer_put(pointer3, payment_option)
    print(f"Created pointer3 at: {addr3}")
    
    # Create pointer2 pointing to pointer3
    target2 = PointerTarget.new_pointer(addr3)
    pointer2 = Pointer(key2, 0, target2)
    addr2 = await client.pointer_put(pointer2, payment_option)
    print(f"Created pointer2 at: {addr2}")
    
    # Create pointer1 pointing to pointer2
    target1 = PointerTarget.new_pointer(addr2)
    pointer1 = Pointer(key1, 0, target1)
    addr1 = await client.pointer_put(pointer1, payment_option)
    print(f"Created pointer1 at: {addr1}")
    
    # Wait for replication
    await asyncio.sleep(3)
    
    # Follow the chain: pointer1 -> pointer2 -> pointer3 -> final_data
    print("\nFollowing the pointer chain:")
    ptr1 = await client.pointer_get(addr1)
    print(f"Pointer1 points to: {ptr1.target.hex}")
    
    ptr2 = await client.pointer_get(ptr1.target.hex)
    print(f"Pointer2 points to: {ptr2.target.hex}")
    
    ptr3 = await client.pointer_get(ptr2.target.hex)
    print(f"Pointer3 points to: {ptr3.target.hex}")
    
    # Get the final data
    chain_result = await client.data_get_public(ptr3.target.hex)
    print(f"Chain result: {chain_result.decode()}")
```

**Node.js Example:**
```javascript
async function pointerChainExample() {
    console.log("\n=== Pointer Chain Example ===");
    
    // Initialize client and wallet
    await init();
    atnm.logInit("ant-networking=warn,autonomi=trace");
    const client = await atnm.Client.connect([window.peer_addr]);
    const wallet = atnm.getFundedWallet();

    // Create multiple keys for different pointers
    const key1 = atnm.genSecretKey();
    const key2 = atnm.genSecretKey();
    const key3 = atnm.genSecretKey();
    
    // Upload final target data
    const finalData = new TextEncoder().encode("Final target in the chain");
    const finalAddr = await client.putData(finalData, wallet);
    const finalTarget = atnm.PointerTarget.newChunk(finalAddr);
    
    // Create pointer3 pointing to the final data
    const pointer3 = new atnm.Pointer(key3, 0, finalTarget);
    const addr3 = await client.putPointer(pointer3, wallet);
    console.log(`Created pointer3 at: ${addr3}`);
    
    // Create pointer2 pointing to pointer3
    const target2 = atnm.PointerTarget.newPointer(addr3);
    const pointer2 = new atnm.Pointer(key2, 0, target2);
    const addr2 = await client.putPointer(pointer2, wallet);
    console.log(`Created pointer2 at: ${addr2}`);
    
    // Create pointer1 pointing to pointer2
    const target1 = atnm.PointerTarget.newPointer(addr2);
    const pointer1 = new atnm.Pointer(key1, 0, target1);
    const addr1 = await client.putPointer(pointer1, wallet);
    console.log(`Created pointer1 at: ${addr1}`);
    
    // Wait for replication
    await new Promise(resolve => setTimeout(resolve, 3000));
    
    // Follow the chain: pointer1 -> pointer2 -> pointer3 -> final_data
    console.log("\nFollowing the pointer chain:");
    const ptr1 = await client.getPointer(addr1);
    console.log(`Pointer1 points to: ${ptr1.target.hex}`);
    
    const ptr2 = await client.getPointer(ptr1.target.hex);
    console.log(`Pointer2 points to: ${ptr2.target.hex}`);
    
    const ptr3 = await client.getPointer(ptr2.target.hex);
    console.log(`Pointer3 points to: ${ptr3.target.hex}`);
    
    // Get the final data
    const chainResult = await client.getData(ptr3.target.hex);
    const decodedChainResult = new TextDecoder().decode(chainResult);
    console.log(`Chain result: ${decodedChainResult}`);
}
```

### Error Handling Example

**Rust Example:**
```rust
async fn error_handling_example() -> Result<()> {
    let client = Client::init_local().await?;
    let wallet = get_funded_wallet();
    let payment_option = PaymentOption::from(&wallet);

    let key = SecretKey::random();
    let target = PointerTarget::ChunkAddress(ChunkAddress::new(XorName::random(&mut rand::thread_rng())));

    // Create a pointer
    let (_, addr) = client
        .pointer_create(&key, target.clone(), payment_option.clone())
        .await?;

    // Try to create another pointer with the same key (should fail)
    match client.pointer_create(&key, target, payment_option).await {
        Ok(_) => panic!("Should have failed - pointer already exists"),
        Err(PointerError::PointerAlreadyExists(existing_addr)) => {
            println!("Pointer already exists at: {existing_addr:?}");
        }
        Err(e) => panic!("Unexpected error: {e}"),
    }

    // Try to update a non-existent pointer
    let non_existent_key = SecretKey::random();
    let new_target = PointerTarget::ChunkAddress(ChunkAddress::new(XorName::random(&mut rand::thread_rng())));
    
    match client.pointer_update(&non_existent_key, new_target).await {
        Ok(_) => panic!("Should have failed - pointer doesn't exist"),
        Err(PointerError::CannotUpdateNewPointer) => {
            println!("Cannot update non-existent pointer");
        }
        Err(e) => panic!("Unexpected error: {e}"),
    }

    Ok(())
}
```

**Python Example:**
```python
async def error_handling_example():
    """Example of handling pointer-related errors."""
    print("\n=== Error Handling Example ===")
    
    # Initialize client and wallet
    private_key = "0xac0974bec39a17e36ba4a6b4d238ff944bacb478cbed5efcae784d7bf4f2ff80"
    network = Network(True)
    wallet = Wallet.new_from_private_key(network, private_key)
    client = await Client.init_local()

    key = SecretKey()
    
    # Try to get a non-existent pointer
    try:
        non_existent_addr = "0000000000000000000000000000000000000000000000000000000000000000"
        await client.pointer_get(non_existent_addr)
    except Exception as e:
        print(f"Expected error when getting non-existent pointer: {e}")
    
    # Create a pointer
    data = b"Test data"
    [cost, addr] = await client.data_put_public(data, PaymentOption.wallet(wallet))
    target = PointerTarget.new_chunk(ChunkAddress(addr))
    pointer = Pointer(key, 0, target)
    payment_option = PaymentOption.wallet(wallet)
    pointer_addr = await client.pointer_put(pointer, payment_option)
    
    # Try to create another pointer with the same key (should fail)
    try:
        another_pointer = Pointer(key, 0, target)
        await client.pointer_put(another_pointer, payment_option)
    except Exception as e:
        print(f"Expected error when creating duplicate pointer: {e}")
```

**Node.js Example:**
```javascript
async function errorHandlingExample() {
    console.log("\n=== Error Handling Example ===");
    
    // Initialize client and wallet
    await init();
    atnm.logInit("ant-networking=warn,autonomi=trace");
    const client = await atnm.Client.connect([window.peer_addr]);
    const wallet = atnm.getFundedWallet();

    const key = atnm.genSecretKey();
    
    // Try to get a non-existent pointer
    try {
        const nonExistentAddr = "0000000000000000000000000000000000000000000000000000000000000000";
        await client.getPointer(nonExistentAddr);
    } catch (e) {
        console.log(`Expected error when getting non-existent pointer: ${e}`);
    }
    
    // Create a pointer
    const data = new TextEncoder().encode("Test data");
    const addr = await client.putData(data, wallet);
    const target = atnm.PointerTarget.newChunk(addr);
    const pointer = new atnm.Pointer(key, 0, target);
    const pointerAddr = await client.putPointer(pointer, wallet);
    
    // Try to create another pointer with the same key (should fail)
    try {
        const anotherPointer = new atnm.Pointer(key, 0, target);
        await client.putPointer(anotherPointer, wallet);
    } catch (e) {
        console.log(`Expected error when creating duplicate pointer: ${e}`);
    }
}
```

## Best Practices

1. **Use `pointer_create` and `pointer_update`** instead of `pointer_put` for most use cases
2. **Handle errors appropriately** - especially `PointerAlreadyExists` and `CannotUpdateNewPointer`
3. **Wait for replication** after creating or updating pointers before trying to retrieve them
4. **Use `pointer_check_existence`** for fast existence checks when you don't need the full pointer data
5. **Consider pointer chains** for complex data structures, but be aware of the traversal cost
6. **Verify signatures** when working with pointers from untrusted sources using `pointer.verify_signature()`

## Performance Considerations

- **Creation cost:** Initial pointer creation requires payment
- **Update cost:** Updates are free after creation
- **Retrieval cost:** Reading pointers is free
- **Replication delay:** Allow 3-5 seconds for network replication after writes
- **Existence checks:** Use `pointer_check_existence` for faster existence verification
