## Pointers

Pointers are native data types in the Autonomi Network that provide an indirection mechanism for referencing to any data. 

* **key-addressed:** Derived from a BLS public key
* **Generic target:** Can reference to any native data type including pointers
* **Pay-once, free updates:** Unlimited updates for free  
* **Versioned & signed:** Includes a version counter and cryptographically verifiable signatures

### Client Methods

- **pointer_get** 
  Retrieves a pointer from the network by its address.
- **pointer_create**  
  Creates and uploads a new pointer for a payment.
  Returns the total cost and the pointer's address.
- **pointer_update**  
  Updates an existing pointer with a new target.
- **pointer_cost**  
  Estimates the storage cost for a pointer.
- **pointer_put**  
  Only use this when you know what you are doing. Else use pointer_create and pointer_update.
  Manually uploads a pointer to the network for a payment.
  Returns the total cost and the pointer's address.

### Example

```rust
use autonomi::Client;
use autonomi::client::payment::PaymentOption;
use autonomi::client::pointer::{Pointer, PointerTarget};
use autonomi::chunk::{Chunk};
use eyre::Result;

async fn pointer_example() -> Result<()> {
    // initialize a local client and test wallet
    let client = Client::init_local().await?;
    let wallet = get_funded_wallet();

    // create a Pointer targeting a random chunk
    let key = autonomi::SecretKey::random();
    let public_key = key.public_key();
    let chunk = Chunk::new(Bytes::from("Hello, world!"));
    let addr = chunk.address();
    let target = PointerTarget::ChunkAddress(addr);
    let pointer = Pointer::new(&key, 0, target.clone());

    // estimate the cost of the pointer
    let cost = client.pointer_cost(&public_key).await?;
    println!("pointer cost: {cost}");

    // put the pointer
    let payment_option = PaymentOption::from(&wallet);
    let (cost, addr) = client
        .pointer_create(&key, target.clone(), payment_option)
        .await?;
    println!("pointer create cost: {cost}");

    // wait for the pointer to be replicated
    tokio::time::sleep(tokio::time::Duration::from_secs(5)).await;

    // check that the pointer is stored
    let got = client.pointer_get(&addr).await?;
    assert_eq!(got, Pointer::new(&key, 0, target));
    println!("pointer got 1");

    // try update pointer and make it point to itself
    let target2 = PointerTarget::PointerAddress(addr);
    client.pointer_update(&key, target2.clone()).await?;

    // wait for the pointer to be replicated
    tokio::time::sleep(tokio::time::Duration::from_secs(5)).await;

    // check that the pointer is updated and its version is incremented
    let got = client.pointer_get(&addr).await?;
    assert_eq!(got, Pointer::new(&key, 1, target2));
    println!("pointer got 2");
    Ok(())
}
``` 