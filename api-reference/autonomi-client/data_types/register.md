## Register

Registers are a high level data type in the Autonomi Network. Which means they are a client side construct built on top of the lower level native data types. Registers are a 32 bytes mutable memory unit with all versions history accessible. 

* **key-addressed:** Network address is derived from a BLS public key
* **Size:** 32 bytes of content per entry
* **Mutable but immutable history:** Register old entries are kept forever but the register can be updated with new entries endlessly.
* **Pay to update:** Register updates are paid for, but cheaper than creation.
* **Signed:** Owned and signed by a key for cryptographic verification

### Client Methods

- **register_get**  
  Retrieves a register from the network by its address.
- **register_create**  
  Creates and uploads a new register for a payment. 
  Returns the total cost and the register's address.
- **register_update**  
  Updates an existing register with new content for a payment.
  The old content is kept in the history.
- **register_cost**  
  Estimates the storage cost for a register.
- **register_history**  
  Retrieves the history of a register.

API helpers for registers:

- **register_key_from_name**  
  Creates a new BLS key deriving it from a name and a base key. 
  This is used to be able to create multiple registers with the same key (and different names). cf [bls key derivation](../bls_keys.md)
- **register_value_from_bytes**
  Makes sure the content fits in 32 bytes.

### Example

```rust
use autonomi::Client;
use autonomi::client::payment::PaymentOption;
use autonomi::SecretKey;
use eyre::Result;

async fn register_example() -> Result<()> {
    // initialize a local client and test wallet
    let client = Client::init_local().await?;
    let wallet = get_funded_wallet();
    let main_key = SecretKey::random();

    // create a register key from a name and our main key
    let register_key = Client::register_key_from_name(&main_key, "register1");

    // estimate the cost of the register
    let cost = client.register_cost(&register_key.public_key()).await?;

    // create the register
    let content = Client::register_value_from_bytes(b"Hello, World!")?;
    let (_cost, addr) = client
        .register_create(&register_key, content, PaymentOption::from(&wallet))
        .await?;

    // let the network replicate the register
    tokio::time::sleep(tokio::time::Duration::from_secs(5)).await;

    // get the register
    let value = client.register_get(&addr).await?;
    assert_eq!(value, content);

    // update the register
    let content2 = Client::register_value_from_bytes(b"你好世界")?;
    client
        .register_update(&register_key, content2, PaymentOption::from(&wallet))
        .await?;

    // let the network replicate the updates
    tokio::time::sleep(tokio::time::Duration::from_secs(5)).await;

    let all = client.register_history(&addr).collect().await?;
    assert_eq!(all.len(), 2);
    assert_eq!(all[0], content);
    assert_eq!(all[1], content2);

    Ok(())
}
```