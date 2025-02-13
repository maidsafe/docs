# Data

The Data API enables storage of arbitrarily sized blobs on the Autonomi Network by leveraging the Chunk native data type. Under the hood, data is self-encrypted, splitting the payload into multiple encrypted chunks that are dispersed across the network. A DataMap, which contains the addresses of these chunks, is generated to facilitate data retrieval and decryption. Depending on the desired access control, the DataMap can be kept private for confidential data storage or published as a chunk, thereby making the data publicly available via the DataMap’s address. 

* **Immutable:** Permanent Immutable/Incorruptible storage
* **Size:** The data can be of any size
* **Access Control:** Data can be private or public

### Client Methods for Private Data (the default on the Network)

* **data\_get**\
  Retrieves bytes from the network using a data map.
* **data\_put**\
  Uploads bytes to the network and returns the price along with the data map to access the data. (This data map is to be kept secret)
* **data\_cost**\
  Estimates the storage cost for a piece of data.

### Client Methods for Public Data

* **data\_get\_public**\
  Retrieves bytes from the network using an address.
* **data\_put\_public**\
  Uploads bytes to the network and returns the price along with the address to access the data. (This address can be published so that others can access the data)
* **data\_cost**\
  Estimates the storage cost for a piece of data.

### Example

```rust
use autonomi::{Client, Bytes};
use autonomi::client::payment::PaymentOption;
use eyre::Result;

async fn data_example() -> Result<()> {
    // initialize a local client and test wallet
    let client = Client::init_local().await?;
    let wallet = get_funded_wallet();
    // initialize a local client and test wallet
    let client = Client::init_local().await?;
    let wallet = get_funded_wallet();

    let data = autonomi::Bytes::from("Hello, World!");
    let cost = client.data_cost(data.clone()).await?;
    println!("cost estimate of uploading data: {cost}");

    let (cost, addr) = client.data_put_public(data.clone(), wallet.into()).await?;
    println!("public data put for: {cost} at address: {addr}");

    let data_fetched = client.data_get_public(&addr).await?;
    assert_eq!(data, data_fetched, "data fetched should match data put");
    Ok(())
}
```
