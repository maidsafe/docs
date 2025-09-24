# GraphEntry

GraphEntry is a fundamental data type in the Autonomi Network that represents an immutable, cryptographically signed graph node that can be linked to other graph entries to form directed acyclic graphs (DAGs).

## Key Features

* **Immutable:** Once uploaded, cannot be modified
* **Cryptographically signed:** Uses BLS signatures for authenticity
* **One per owner:** Each public key can only have one GraphEntry
* **Linkable:** Can reference other GraphEntries as parents or descendants
* **Content-addressed:** Stored at an address derived from the owner's public key
* **Size:** 32 bytes of content per entry

## Structure

* `owner`: The BLS public key of the entry owner
* `parents`: References to parent GraphEntries
* `content`: 32 bytes of arbitrary data
* `descendants`: References to descendant entries with associated data (32 bytes per descendant)
* `signature`: BLS signature ensuring integrity

## Common Use Cases

* Building DAG structures
* Creating audit trails
* Implementing linked data structures
* Foundation for higher-level types like Registers

### Client Methods

* **graph\_entry\_get**\
  Retrieves a GraphEntry from the network by its address.
* **graph\_entry\_put**\
  Uploads a GraphEntry to the network with payment handling.
* **graph\_entry\_cost**\
  Estimates the storage cost for a GraphEntry.

## Code Examples

{% tabs %}
{% tab title="Rust" %}
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
{% endtab %}

{% tab title="Python" %}
### Creating and Storing a GraphEntry

```python
from autonomi_client import Client, Network, Wallet, SecretKey, GraphEntry, GraphEntryAddress, PaymentOption
import asyncio

async def graph_entry_example():
    # Initialize a wallet with the testnet private key
    private_key = "0xac0974bec39a17e36ba4a6b4d238ff944bacb478cbed5efcae784d7bf4f2ff80"
    network = Network(True)
    wallet = Wallet.new_from_private_key(network, private_key)
    
    # Connect to the network
    client = await Client.init_local()
    print("Connected to network!")
    
    # Create a GraphEntry with some content
    key = SecretKey.random()
    content = b'\x42' * 32  # 32 bytes of 0x42
    graph_entry = GraphEntry(key, [], content, [])
    
    # Estimate the cost of the graph_entry
    cost = await client.graph_entry_cost(key.public_key())
    print(f"GraphEntry cost: {cost}")
    
    # Store the graph_entry
    payment_option = PaymentOption.wallet(wallet)
    (cost, addr) = await client.graph_entry_put(graph_entry, payment_option)
    print(f"GraphEntry stored for {cost} testnet ANT at: {addr.to_hex()}")
    
    # Wait for the graph_entry to be replicated
    await asyncio.sleep(5)
    
    # Retrieve the graph_entry
    retrieved_entry = await client.graph_entry_get(addr)
    print(f"Retrieved GraphEntry content: {retrieved_entry.content.hex()}")
    
    # Check if the entry exists (faster than fetching)
    exists = await client.graph_entry_check_existance(addr)
    print(f"GraphEntry exists: {exists}")

# Run the example
asyncio.run(graph_entry_example())
```

### Creating Linked GraphEntries

```python
from autonomi_client import Client, Network, Wallet, SecretKey, GraphEntry, GraphEntryAddress, PaymentOption
import asyncio

async def linked_graph_entries_example():
    # Initialize wallet and client
    private_key = "0xac0974bec39a17e36ba4a6b4d238ff944bacb478cbed5efcae784d7bf4f2ff80"
    network = Network(True)
    wallet = Wallet.new_from_private_key(network, private_key)
    client = await Client.init_local()
    
    # Create the first GraphEntry
    key1 = SecretKey.random()
    content1 = b'First entry' + b'\x00' * 21  # Pad to 32 bytes
    graph_entry1 = GraphEntry(key1, [], content1, [])
    
    # Store the first entry
    payment_option = PaymentOption.wallet(wallet)
    (cost1, addr1) = await client.graph_entry_put(graph_entry1, payment_option)
    print(f"First GraphEntry stored at: {addr1.to_hex()}")
    
    # Wait for replication
    await asyncio.sleep(5)
    
    # Create a second GraphEntry linking to the first as parent
    key2 = SecretKey.random()
    content2 = b'Second entry' + b'\x00' * 20  # Pad to 32 bytes
    graph_entry2 = GraphEntry(key2, [key1.public_key()], content2, [])
    
    # Store the second entry
    (cost2, addr2) = await client.graph_entry_put(graph_entry2, payment_option)
    print(f"Second GraphEntry stored at: {addr2.to_hex()}")
    
    # Create a third GraphEntry with descendants
    key3 = SecretKey.random()
    content3 = b'Third entry' + b'\x00' * 21  # Pad to 32 bytes
    descendant_data = b'Descendant metadata' + b'\x00' * 13  # Pad to 32 bytes
    graph_entry3 = GraphEntry(
        key3, 
        [key2.public_key()],  # Parent is entry2
        content3,
        [(key2.public_key(), descendant_data)]  # Descendant reference with metadata
    )
    
    # Store the third entry
    (cost3, addr3) = await client.graph_entry_put(graph_entry3, payment_option)
    print(f"Third GraphEntry stored at: {addr3.to_hex()}")
    
    # Wait for replication
    await asyncio.sleep(5)
    
    # Retrieve and verify the linked structure
    entry2 = await client.graph_entry_get(addr2)
    print(f"Entry2 has {len(entry2.parents)} parent(s)")
    
    entry3 = await client.graph_entry_get(addr3)
    print(f"Entry3 has {len(entry3.parents)} parent(s) and {len(entry3.descendants)} descendant(s)")

# Run the example
asyncio.run(linked_graph_entries_example())
```

### Handling GraphEntry Forks

```python
from autonomi_client import Client, Network, Wallet, GraphEntryAddress
import asyncio

async def handle_graph_entry_forks():
    # Initialize client
    client = await Client.init_local()
    
    # Attempt to retrieve a GraphEntry
    addr_hex = "0x..."  # Replace with actual address
    addr = GraphEntryAddress.from_hex(addr_hex)
    
    try:
        entry = await client.graph_entry_get(addr)
        print(f"Retrieved single GraphEntry: {entry.content.hex()}")
    except Exception as e:
        # In Python, forks are currently handled as exceptions
        # Check the error message for fork detection
        if "Fork" in str(e):
            print(f"Fork detected: {e}")
            # Handle fork scenario - application specific logic
            # You might need to choose between entries or merge them
        else:
            print(f"Error retrieving GraphEntry: {e}")

# Run the example
asyncio.run(handle_graph_entry_forks())
```
{% endtab %}

{% tab title="Node.js" %}
### Creating and Storing a GraphEntry

```javascript
import { Client, Network, Wallet, SecretKey, GraphEntry, GraphEntryAddress, PaymentOption } from '@withautonomi/autonomi'

async function graphEntryExample() {
    // Initialize a wallet with the testnet private key
    const privateKey = "0xac0974bec39a17e36ba4a6b4d238ff944bacb478cbed5efcae784d7bf4f2ff80"
    const wallet = Wallet.newFromPrivateKey(Network(true), privateKey)
    
    // Connect to the network
    const client = await Client.initLocal()
    console.log("Connected to network!")
    
    // Create a GraphEntry with some content
    const key = SecretKey.random()
    const content = new Uint8Array(32).fill(42) // 32 bytes of 42
    const graphEntry = new GraphEntry(key, [], content, [])
    
    // Estimate the cost of the graph_entry
    const cost = await client.graph_entry_cost(key.publicKey())
    console.log(`GraphEntry cost: ${cost}`)
    
    // Store the graph_entry
    const paymentOption = PaymentOption.fromWallet(wallet)
    const result = await client.graph_entry_put(graphEntry, paymentOption)
    console.log(`GraphEntry stored for ${result.cost} testnet ANT at: ${result.addr.to_hex()}`)
    
    // Wait for the graph_entry to be replicated
    await new Promise(resolve => setTimeout(resolve, 5000))
    
    // Retrieve the graph_entry
    const retrievedEntry = await client.graph_entry_get(result.addr)
    console.log(`Retrieved GraphEntry content: ${Buffer.from(retrievedEntry.content).toString('hex')}`)
    
    // Check if the entry exists (faster than fetching)
    const exists = await client.graph_entry_check_existance(result.addr)
    console.log(`GraphEntry exists: ${exists}`)
}

// Run the example
graphEntryExample().catch(console.error)
```

### Creating Linked GraphEntries

<pre class="language-javascript"><code class="lang-javascript">import { Client, Network, Wallet, SecretKey, GraphEntry, GraphEntryAddress, PaymentOption } from '@withautonomi/autonomi'
<strong>
</strong>async function linkedGraphEntriesExample() {
    // Initialize wallet and client
    const privateKey = "0xac0974bec39a17e36ba4a6b4d238ff944bacb478cbed5efcae784d7bf4f2ff80"
    const wallet = Wallet.newFromPrivateKey(Network(true), privateKey)
    const client = await Client.initLocal()
    
    // Create the first GraphEntry
    const key1 = SecretKey.random()
    const content1 = Buffer.concat([
        Buffer.from('First entry'),
        Buffer.alloc(21) // Pad to 32 bytes
    ])
    const graphEntry1 = new GraphEntry(key1, [], content1, [])
    
    // Store the first entry
    const paymentOption = PaymentOption.fromWallet(wallet)
    const result1 = await client.graph_entry_put(graphEntry1, paymentOption)
    console.log(`First GraphEntry stored at: ${result1.addr.to_hex()}`)
    
    // Wait for replication
    await new Promise(resolve => setTimeout(resolve, 5000))
    
    // Create a second GraphEntry linking to the first as parent
    const key2 = SecretKey.random()
    const content2 = Buffer.concat([
        Buffer.from('Second entry'),
        Buffer.alloc(20) // Pad to 32 bytes
    ])
    const graphEntry2 = new GraphEntry(key2, [key1.publicKey()], content2, [])
    
    // Store the second entry
    const result2 = await client.graph_entry_put(graphEntry2, paymentOption)
    console.log(`Second GraphEntry stored at: ${result2.addr.to_hex()}`)
    
    // Create a third GraphEntry with descendants
    const key3 = SecretKey.random()
    const content3 = Buffer.concat([
        Buffer.from('Third entry'),
        Buffer.alloc(21) // Pad to 32 bytes
    ])
    const descendantData = Buffer.concat([
        Buffer.from('Descendant metadata'),
        Buffer.alloc(13) // Pad to 32 bytes
    ])
    const graphEntry3 = new GraphEntry(
        key3,
        [key2.publicKey()], // Parent is entry2
        content3,
        [[key2.publicKey(), descendantData]] // Descendant reference with metadata
    )
    
    // Store the third entry
    const result3 = await client.graph_entry_put(graphEntry3, paymentOption)
    console.log(`Third GraphEntry stored at: ${result3.addr.to_hex()}`)
    
    // Wait for replication
    await new Promise(resolve => setTimeout(resolve, 5000))
    
    // Retrieve and verify the linked structure
    const entry2 = await client.graph_entry_get(result2.addr)
    console.log(`Entry2 has ${entry2.parents.length} parent(s)`)
    
    const entry3 = await client.graph_entry_get(result3.addr)
    console.log(`Entry3 has ${entry3.parents.length} parent(s) and ${entry3.descendants.length} descendant(s)`)
}

// Run the example
linkedGraphEntriesExample().catch(console.error)
</code></pre>

### Handling GraphEntry Forks

```javascript
import { Client, GraphEntryAddress } from '@withautonomi/autonomi'

async function handleGraphEntryForks() {
    // Initialize client
    const client = await Client.initLocal()
    
    // Attempt to retrieve a GraphEntry
    const addrHex = "0x..." // Replace with actual address
    const addr = GraphEntryAddress.from_hex(addrHex)
    
    try {
        const entry = await client.graph_entry_get(addr)
        console.log(`Retrieved single GraphEntry: ${Buffer.from(entry.content).toString('hex')}`)
    } catch (error) {
        // In NodeJS, forks are currently handled as exceptions
        // Check the error message for fork detection
        if (error.message.includes("Fork")) {
            console.log(`Fork detected: ${error.message}`)
            // Handle fork scenario - application specific logic
            // You might need to choose between entries or merge them
        } else {
            console.log(`Error retrieving GraphEntry: ${error.message}`)
        }
    }
}

// Run the example
handleGraphEntryForks().catch(console.error)
```

### Working with GraphEntry Addresses

```javascript
import { SecretKey, GraphEntryAddress } from '@withautonomi/autonomi'

// Create an address from a public key
const key = SecretKey.random()
const addr = new GraphEntryAddress(key.publicKey())

// Convert to/from hex representation
const hexAddr = addr.to_hex()
console.log(`GraphEntry address: ${hexAddr}`)

// Recreate address from hex
const addrFromHex = GraphEntryAddress.from_hex(hexAddr)
console.log(`Addresses match: ${addr.to_hex() === addrFromHex.to_hex()}`)
```
{% endtab %}
{% endtabs %}
