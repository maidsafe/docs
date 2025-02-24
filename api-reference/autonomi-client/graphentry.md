# GraphEntry

GraphEntry is a native data type in the Autonomi Network:

* **key-addressed:** Network address is derived from a BLS public key
* **Size:** 32 bytes of content per entry
* **Immutable:** Once created, it cannot be changed
* **Directed Graph:** Can reference parent entries and descendant entries (with 32 bytes of metadata per descendant)
* **Signed:** Owned and signed by a key for cryptographic verification

### Client Methods

* **graph\_entry\_get**\
  Retrieves a GraphEntry from the network by its address.
* **graph\_entry\_put**\
  Uploads a GraphEntry to the network with payment handling.
* **graph\_entry\_cost**\
  Estimates the storage cost for a GraphEntry.

### Example

```rust
use autonomi::Client;
use autonomi::client::payment::PaymentOption;
use autonomi::client::graph::GraphEntry;
use eyre::Result;

async fn graph_entry_example() -> Result<()> {
    // initialize a local client and test wallet
    let client = Client::init_local().await?;
    let wallet = get_funded_wallet();

    // create a GraphEntry with some content
    let key = autonomi::SecretKey::random();
    let content = [42u8; 32]; // 32 bytes of 42s
    let graph_entry = GraphEntry::new(&key, vec![], content, vec![]);

    // estimate the cost of the graph_entry
    let cost = client.graph_entry_cost(&key.public_key()).await?;
    println!("graph_entry cost: {cost}");

    // put the graph_entry
    let payment_option = PaymentOption::from(&wallet);
    let (cost, addr) = client
        .graph_entry_put(graph_entry.clone(), payment_option)
        .await?;
    println!("graph_entry put cost: {cost}");

    // wait for the graph_entry to be replicated
    tokio::time::sleep(tokio::time::Duration::from_secs(5)).await;

    // check that the graph_entry is stored
    let entry = client.graph_entry_get(&graph_entry.address()).await?;
    assert_eq!(entry, graph_entry.clone());

    // try to put a graph entry linking to the first graph entry
    let key2 = autonomi::SecretKey::random();
    let content2 = [41u8; 32];
    let graph_entry2 = GraphEntry::new(&key2, vec![graph_entry.owner], content2, vec![]);

    // put the graph_entry
    let payment_option = PaymentOption::from(&wallet);
    let (cost, addr) = client
        .graph_entry_put(graph_entry2.clone(), payment_option)
        .await?;
    println!("graph_entry2 put cost: {cost}");

    // wait for the graph_entry to be replicated
    tokio::time::sleep(tokio::time::Duration::from_secs(5)).await;

    // check that the graph_entry is stored
    let entry2 = client.graph_entry_get(&graph_entry2.address()).await?;
    assert_eq!(entry2, graph_entry2.clone());

    Ok(())
}
```

### Forks

It's possible for multiple graph entries to exist for the same key. This can happen when two nodes store (unique) entries simultaneously. These entries are replicated and nodes will store them all (up to a certain amount).

The graph entry API will surface these forks as errors. In case of Rust, the `graph_entry_get` will return `Result::Err(GraphError::Fork(Vec<GraphEntry>))`, which holds all entries found for the same key.

```rust
use autonomi::client::data_types::graph::GraphError;
let graph_entry_addr = todo!();
let entry = match self.graph_entry_get(graph_entry_addr).await {
    Ok(entry) => entry,
    Err(GraphError::Fork(entries)) => {
        // Fork detected.
        todo!();
    }
    Err(err) => todo!(),
};
```
