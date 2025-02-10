## Scratchpad

Scratchpad is a native data type in the Autonomi Network:  

* **key-addressed:** Network address is derived from a BLS public key
* **Size:** 4MB mutable storage
* **Pay-once, free updates:** Unlimited updates for free  
* **Versioned & signed:** Includes a version counter and cryptographically verifiable signatures

### Client Methods

- **scratchpad_get**  
  Retrieves a scratchpad from the network by its address.
- **scratchpad_create**  
  Creates and uploads a new scratchpad for a payment. 
  Encrypts the content automatically.
  Returns the total cost and the scratchpad's address.
- **scratchpad_update**  
  Updates an existing scratchpad with new content. 
  Encrypts the content automatically.
- **scratchpad_cost**  
  Estimates the storage cost for a scratchpad.
- **scratchpad_put**  
  Only use this when you know what you are doing. Else use scratchpad_create and scratchpad_update.
  Manually uploads or updates a scratchpad on the network for a payment. Assumes it is valid and the content already encrypted. 
  Returns the total cost and the scratchpad's address.

### Example

```rust
use autonomi::Client;
use autonomi::client::payment::PaymentOption;
use autonomi::client::scratchpad::{Scratchpad, Bytes};
use eyre::Result;

async fn scratchpad_example() -> Result<()> {
    // initialize a local client and test wallet
    let client = Client::init_local().await?;
    let wallet = get_funded_wallet();

    // create a Scratchpad with some data
    let key = autonomi::SecretKey::random();
    let public_key = key.public_key();
    let content = Bytes::from("what's the meaning of life the universe and everything?");
    let content_type = 42;

    // estimate the cost of the scratchpad
    let cost = client.scratchpad_cost(&public_key).await?;
    println!("scratchpad cost: {cost}");

    // create the scratchpad
    let payment_option = PaymentOption::from(&wallet);
    let (cost, addr) = client
        .scratchpad_create(&key, content_type, &content, payment_option)
        .await?;
    println!("scratchpad create cost: {cost}");

    // wait for the scratchpad to be replicated
    tokio::time::sleep(tokio::time::Duration::from_secs(5)).await;

    // check that the scratchpad is stored
    let got = client.scratchpad_get(&addr).await?;
    assert_eq!(*got.owner(), public_key);
    assert_eq!(got.data_encoding(), content_type);
    assert_eq!(got.decrypt_data(&key), Ok(content.clone()));
    assert_eq!(got.counter(), 0);
    assert!(got.verify_signature());
    println!("scratchpad got 1");

    // check that the content is decrypted correctly
    let got_content = got.decrypt_data(&key)?;
    assert_eq!(got_content, content);

    // try update scratchpad
    let content2 = Bytes::from("42");
    client
        .scratchpad_update(&key, content_type, &content2)
        .await?;

    // wait for the scratchpad to be replicated
    tokio::time::sleep(tokio::time::Duration::from_secs(5)).await;

    // check that the scratchpad is updated
    let got = client.scratchpad_get(&addr).await?;
    assert_eq!(*got.owner(), public_key);
    assert_eq!(got.data_encoding(), content_type);
    assert_eq!(got.decrypt_data(&key), Ok(content2.clone()));
    assert_eq!(got.counter(), 1);
    assert!(got.verify_signature());
    println!("scratchpad got 2");

    // check that the content is decrypted correctly
    let got_content2 = got.decrypt_data(&key)?;
    assert_eq!(got_content2, content2);
    Ok(())
}
```