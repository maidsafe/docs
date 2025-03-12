# Analyze

The `analyze` module provides functionality for analyzing data addresses on the Autonomi Network. 

This can also be used with the CLI to analyze data addresses:

```bash
ant analyze a7d2fdbb975efaea25b7ebe3d38be4a0b82c1d71e9b89ac4f37bc9f8677826e0
```

### Client Methods

* **analyze\_address**\
  Analyzes an address on the Autonomi Network.

### Example

```rust
use autonomi::Client;
use autonomi::client::payment::PaymentOption;
use autonomi::client::chunk::{Chunk, Bytes};
use autonomi::client::analyze::Analysis;
use test_utils::evm::get_funded_wallet;
use eyre::Result;

async fn analyze_address_example() -> Result<()> {
    // initialize a local client and test wallet
    let client = Client::init_local().await?;
    let wallet = get_funded_wallet();
    let payment_option = PaymentOption::from(&wallet);

    // create a Chunk with some data
    let chunk = Chunk::new(Bytes::from("Chunk content example"));
    let (_cost, addr) = client.chunk_put(&chunk, payment_option).await?;
    let chunk_addr = addr.to_hex();

    // analyze the chunk address
    let analysis = client.analyze_address(&chunk_addr, true).await?;

    // the analysis detects that the address is a Chunk
    assert_eq!(analysis, Analysis::Chunk(chunk));

    Ok(())
}
```