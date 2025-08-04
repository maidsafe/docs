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

### Pointer Chain Example

```rust
async fn pointer_chain_example() -> Result<()> {
    let client = Client::init_local().await?;
    let wallet = get_funded_wallet();
    let payment_option = PaymentOption::from(&wallet);

    // Create multiple pointers that form a chain
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

    // Follow the chain: pointer1 -> pointer2 -> pointer3 -> chunk
    let ptr1 = client.pointer_get(&addr1).await?;
    let ptr2 = client.pointer_get(&ptr1.target().as_pointer_address().unwrap()).await?;
    let ptr3 = client.pointer_get(&ptr2.target().as_pointer_address().unwrap()).await?;
    
    // Verify we reach the original chunk
    assert!(matches!(ptr3.target(), PointerTarget::ChunkAddress(_)));

    Ok(())
}
```

### Error Handling Example

```rust
async fn error_handling_example() -> Result<()> {
    let client = Client::init_local().await?;
    let wallet = get_funded_wallet();
    let payment_option = PaymentOption::from(&wallet);

    let key = SecretKey::random();
    let target = PointerTarget::ChunkAddress(ChunkAddress::new(XorName::random(&mut rand::thread_rng())));

    // Create a pointer
    let (_, addr) = client
        .pointer_create(&key, target.clone(), payment_option)
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

### Checking Pointer Existence

```rust
async fn existence_check_example() -> Result<()> {
    let client = Client::init_local().await?;
    let wallet = get_funded_wallet();
    let payment_option = PaymentOption::from(&wallet);

    let key = SecretKey::random();
    let target = PointerTarget::ChunkAddress(ChunkAddress::new(XorName::random(&mut rand::thread_rng())));

    // Check if pointer exists before creation
    let addr = PointerAddress::new(key.public_key());
    let exists = client.pointer_check_existence(&addr).await?;
    assert!(!exists);

    // Create the pointer
    client.pointer_create(&key, target, payment_option).await?;

    // Wait a bit for replication
    tokio::time::sleep(tokio::time::Duration::from_secs(3)).await;

    // Check if pointer exists after creation
    let exists = client.pointer_check_existence(&addr).await?;
    assert!(exists);

    Ok(())
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
