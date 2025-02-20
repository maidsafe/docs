# Chunks

Chunks are native data types in the Autonomi Network:

* **Size:** 4MB of raw bytes
* **Content-addressed:** Address is the hash of its content
* **Immutable & self-verifiable:** Once stored, data cannot be modified, and its integrity can be verified by computing the hash and comparing it to the address.

### Client Methods

* **chunk\_get**\
  Retrieves a chunk from the network by its address.
* **chunk\_put**\
  Uploads a chunk to the network with payment handling.\
  Returns the total cost and the chunk's address.
* **chunk\_cost**\
  Estimates the storage cost for a chunk.

### Example

```rust
use autonomi::Client;
use autonomi::client::payment::PaymentOption;
use autonomi::client::chunk::{Chunk, Bytes};
use test_utils::evm::get_funded_wallet;
use eyre::Result;

async fn chunk_put_example() -> Result<()> {
    // initialize a local client and test wallet
    let client = Client::init_local().await?;
    let wallet = get_funded_wallet();

    // create a Chunk with some data
    let chunk = Chunk::new(Bytes::from("Hello, world!"));

    // Estimate cost
    let cost = client.chunk_cost(chunk.address()).await?;
    println!("Chunk cost: {cost}");

    // Upload chunk with payment
    let payment_option = PaymentOption::from(&wallet);
    let (put_cost, addr) = client.chunk_put(&chunk, payment_option).await?;
    assert_eq!(addr, *chunk.address());
    println!("Chunk put cost: {put_cost}");

    // Allow time for replication
    tokio::time::sleep(tokio::time::Duration::from_secs(5)).await;

    // Retrieve and verify the chunk
    let got = client.chunk_get(&addr).await?;
    assert_eq!(got, chunk.clone());
    println!("Chunk retrieved successfully");
    Ok(())
}
```
