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
* **chunk\_batch\_upload**\
  Uploads multiple chunks efficiently with a single payment receipt.

## Examples

### Basic Chunk Operations

#### Rust

```rust
use autonomi::Client;
use autonomi::client::payment::PaymentOption;
use autonomi::client::chunk::{Chunk, Bytes};
use test_utils::evm::get_funded_wallet;
use eyre::Result;

#[tokio::main]
async fn main() -> Result<()> {
    // Initialize a local client and test wallet
    let client = Client::init_local().await?;
    let wallet = get_funded_wallet();

    // Create a chunk with some data
    let chunk = Chunk::new(Bytes::from("Hello, world!"));
    println!("Created chunk with size: {} bytes", chunk.size());

    // Estimate cost before uploading
    let cost = client.chunk_cost(chunk.address()).await?;
    println!("Estimated storage cost: {cost}");

    // Upload chunk with payment
    let payment_option = PaymentOption::from(&wallet);
    let (put_cost, addr) = client.chunk_put(&chunk, payment_option).await?;
    assert_eq!(addr, *chunk.address());
    println!("Chunk uploaded for: {put_cost} at address: {}", addr.to_hex());

    // Wait for replication across the network
    tokio::time::sleep(tokio::time::Duration::from_secs(5)).await;

    // Retrieve and verify the chunk
    let retrieved_chunk = client.chunk_get(&addr).await?;
    assert_eq!(retrieved_chunk, chunk);
    println!("Chunk retrieved successfully, data: {}", 
             String::from_utf8_lossy(retrieved_chunk.value()));
    
    Ok(())
}
```

#### Node.js

```javascript
import { Client, Wallet, Network, PaymentOption, ChunkAddress, XorName } from 'autonomi';

async function chunkExample() {
    // Initialize client and wallet
    const client = await Client.initLocal();
    const wallet = Wallet.newFromPrivateKey(
        new Network(true), 
        "0xac0974bec39a17e36ba4a6b4d238ff944bacb478cbed5efcae784d7bf4f2ff80"
    );

    // Create chunk data
    const data = Buffer.from("Hello, world!");
    console.log(`Creating chunk with size: ${data.length} bytes`);

    // Estimate cost before uploading
    const estimateAddr = new ChunkAddress(XorName.fromContent(data));
    const cost = await client.chunkCost(estimateAddr);
    console.log(`Estimated storage cost: ${cost}`);

    // Upload chunk with payment
    const paymentOption = PaymentOption.fromWallet(wallet);
    const result = await client.chunkPut(data, paymentOption);
    console.log(`Chunk uploaded for: ${result.cost} at address: ${result.addr.toHex()}`);

    // Wait for replication
    await new Promise(resolve => setTimeout(resolve, 5000));

    // Retrieve and verify the chunk
    const retrievedData = await client.chunkGet(result.addr);
    console.log(`Chunk retrieved successfully, data: ${retrievedData.toString()}`);
    
    // Verify data integrity
    if (Buffer.compare(data, retrievedData) === 0) {
        console.log("Data integrity verified ✓");
    }
}

chunkExample().catch(console.error);
```

#### Python

```python
import asyncio
from autonomi_client import Client, PaymentOption, Chunk, ChunkAddress

async def chunk_example():
    """Example of basic chunk operations in Python."""
    
    # Initialize client
    client = await Client.init_local()
    
    # Create payment option (assumes wallet is configured)
    payment_option = PaymentOption.from_wallet(wallet)
    
    # Create chunk from data
    data = b"Hello, world!"
    chunk = Chunk(data)
    print(f"Created chunk with size: {chunk.size()} bytes")
    print(f"Chunk address: {chunk.address().to_hex()}")
    
    # Estimate cost before uploading
    cost = await client.chunk_cost(chunk.address())
    print(f"Estimated storage cost: {cost}")
    
    # Upload chunk
    put_cost, address = await client.chunk_put(list(data), payment_option)
    print(f"Chunk uploaded for: {put_cost} at address: {address.to_hex()}")
    
    # Wait for replication
    await asyncio.sleep(5)
    
    # Retrieve chunk
    retrieved_data = await client.chunk_get(address)
    retrieved_bytes = bytes(retrieved_data)
    print(f"Chunk retrieved successfully, data: {retrieved_bytes.decode()}")
    
    # Verify data integrity
    assert retrieved_bytes == data, "Data mismatch!"
    print("Data integrity verified ✓")

# Run the example
if __name__ == "__main__":
    asyncio.run(chunk_example())
```

### Batch Upload Operations

For uploading multiple chunks efficiently:

#### Rust

```rust
use autonomi::{Client, client::chunk::Chunk};
use autonomi::client::payment::{PaymentOption, receipt_from_store_quotes};
use ant_protocol::storage::DataTypes;
use bytes::Bytes;
use eyre::Result;

async fn batch_chunk_upload() -> Result<()> {
    let client = Client::init_local().await?;
    let wallet = get_funded_wallet();

    // Create multiple chunks
    let data_chunks = vec![
        "First chunk data",
        "Second chunk data", 
        "Third chunk data",
    ];
    
    let chunks: Vec<Chunk> = data_chunks
        .into_iter()
        .map(|data| Chunk::new(Bytes::from(data)))
        .collect();

    // Collect chunk references for batch operations
    let chunk_refs: Vec<&Chunk> = chunks.iter().collect();

    // Get storage quotes for all chunks
    let quote = client.get_store_quotes(
        DataTypes::Chunk,
        chunks.iter().map(|chunk| (*chunk.address().xorname(), chunk.size()))
    ).await?;

    // Pay for all chunks at once
    wallet.pay_for_quotes(quote.payments()).await.map_err(|err| err.0)?;
    let receipt = receipt_from_store_quotes(quote);

    // Upload all chunks with the payment receipt
    client.chunk_batch_upload(chunk_refs, &receipt).await?;
    println!("Successfully uploaded {} chunks in batch", chunks.len());

    // Verify all chunks are retrievable
    for (i, chunk) in chunks.iter().enumerate() {
        let retrieved = client.chunk_get(chunk.address()).await?;
        assert_eq!(&retrieved, chunk);
        println!("Verified chunk {}: ✓", i + 1);
    }

    Ok(())
}
```

### Error Handling

#### Rust

```rust
use autonomi::{Client, client::chunk::Chunk};
use autonomi::client::{GetError, PutError};
use bytes::Bytes;
use eyre::Result;

async fn chunk_error_handling() -> Result<()> {
    let client = Client::init_local().await?;
    let wallet = get_funded_wallet();

    // Test size limits
    let oversized_data = vec![0u8; Chunk::MAX_SIZE + 1];
    let oversized_chunk = Chunk::new(Bytes::from(oversized_data));
    
    if oversized_chunk.is_too_big() {
        println!("Chunk exceeds maximum size of {} bytes", Chunk::MAX_SIZE);
        return Ok(());
    }

    // Handle put errors
    match client.chunk_put(&oversized_chunk, PaymentOption::from(&wallet)).await {
        Ok((cost, addr)) => {
            println!("Upload successful: cost={}, addr={}", cost, addr.to_hex());
        }
        Err(PutError::Serialization(msg)) => {
            println!("Serialization error: {}", msg);
        }
        Err(PutError::Network { address, network_error, payment }) => {
            println!("Network error at {}: {}", address, network_error);
        }
        Err(e) => {
            println!("Other put error: {}", e);
        }
    }

    // Handle get errors
    let invalid_addr = ChunkAddress::from_hex("0000000000000000000000000000000000000000000000000000000000000000")?;
    match client.chunk_get(&invalid_addr).await {
        Ok(chunk) => {
            println!("Retrieved chunk with {} bytes", chunk.size());
        }
        Err(GetError::RecordNotFound) => {
            println!("Chunk not found at address: {}", invalid_addr.to_hex());
        }
        Err(GetError::Network(network_err)) => {
            println!("Network error during retrieval: {}", network_err);
        }
        Err(e) => {
            println!("Other get error: {}", e);
        }
    }

    Ok(())
}
```

> Other language support for error handling is in development

### Advanced Configuration

#### Performance Tuning

```rust
use std::env;

// Configure batch sizes via environment variables
fn configure_performance() {
    // Set chunk upload concurrency (default: 1)
    env::set_var("CHUNK_UPLOAD_BATCH_SIZE", "4");
    
    // Set chunk download concurrency (default: 1)  
    env::set_var("CHUNK_DOWNLOAD_BATCH_SIZE", "8");
}
```

## Streaming Data

The `data_stream_public` method returns an iterator that yields chunks of data as they arrive from the network:

#### Rust

```rust
use autonomi::{Client, data::DataAddress};
use std::fs::File;
use std::io::Write;

#[tokio::main]
async fn main() -> Result<(), Box<dyn std::error::Error>> {
    let client = Client::init().await?;

    // Example: download a large video file
    let address = DataAddress::from_hex("bfbe41a1fffdcd0b40e8bc569e3f24f0dd7646a9d2883515ccca92c0623291ae")?;

    // Create output file
    println!("Creating empty file video.mp4");
    let mut file = File::create("video.mp4")?;

    // Stream chunks and write to file
    let data_stream = client.data_stream_public(&address).await?;

    let total_size = data_stream.data_size();
    println!("Data size: {} bytes", total_size);

    println!("Begin streaming...");
    let mut chunk_count = 0;
    for chunk_result in data_stream {
        let chunk = chunk_result?;
        chunk_count += 1;
        println!("Got chunk {}", chunk_count);
        file.write_all(&chunk)?;
    }

    println!("Download complete!");
    Ok(())
}
```

## Size Limits and Best Practices

- **Maximum chunk size:** 4MB raw data (`Chunk::MAX_RAW_SIZE`)
- **Content addressing:** Each chunk's address is the hash of its content
- **Immutability:** Chunks cannot be modified once stored
- **Batch operations:** Use `chunk_batch_upload()` for multiple chunks to reduce costs
- **Streaming:** Use `data_stream_public()` or `data_stream()` (private data) for efficient large data transfers
- **Caching:** Enable chunk caching for frequently accessed data
- **Error handling:** Always handle network errors and size limit violations
