# Private Archive

A Private Archive provides enhanced privacy by keeping data maps within the archive itself rather than referencing them on the network. Unlike Public Archives which reference files through their network addresses, Private Archives contain the complete data maps within the archive structure. Like Public Archives, they support nested paths to simulate directories and store metadata for each file.

The key difference is that Private Archives embed the DataMapChunk directly in the archive, meaning the archive contains all the information needed to reconstruct the files without needing to fetch additional data maps from the network. This makes the archive self-contained and ensures that file organization and structure remain private.

## Structure

```rust
pub struct PrivateArchive {
    // Path of the file in the directory -> (DataMap chunk, Metadata)
    map: BTreeMap<PathBuf, (DataMapChunk, Metadata)>,
}
```

## Instance Methods

### `new() -> Self`

Create a new empty Private Archive.

```rust
let archive = PrivateArchive::new();
```

### `add_file(&mut self, path: PathBuf, data_map: DataMapChunk, meta: Metadata)`

Add a file to the archive with its data map and metadata.

```rust
archive.add_file(
    PathBuf::from("documents/report.pdf"),
    data_map_chunk,
    metadata
);
```

### `rename_file(&mut self, old_path: &Path, new_path: &Path) -> Result<(), RenameError>`

Rename a file within the archive. This only changes the path in the archive structure.

```rust
archive.rename_file(
    Path::new("old/path/file.txt"),
    Path::new("new/path/file.txt")
)?;
```

### `files(&self) -> Vec<(PathBuf, Metadata)>`

Get a list of all files in the archive with their metadata.

```rust
let file_list = archive.files();
for (path, metadata) in file_list {
    println!("{}: {} bytes", path.display(), metadata.size);
}
```

### `data_maps(&self) -> Vec<DataMapChunk>`

Get all data map chunks stored in the archive.

```rust
let data_maps = archive.data_maps();
```

### `iter(&self) -> impl Iterator<Item = (&PathBuf, &DataMapChunk, &Metadata)>`

Iterate over all items in the archive.

```rust
for (path, data_map, metadata) in archive.iter() {
    println!("{}: {} bytes", path.display(), metadata.size);
}
```

### `map(&self) -> &BTreeMap<PathBuf, (DataMapChunk, Metadata)>`

Get direct access to the underlying map structure.

```rust
let map = archive.map();
```

### `merge(&mut self, other: &PrivateArchive)`

Merge another archive into this one. Files from the other archive will be added to this archive.

```rust
archive.merge(&other_archive);
```

### `to_bytes(&self) -> Result<Bytes, rmp_serde::encode::Error>`

Serialize the archive to bytes for storage.

```rust
let bytes = archive.to_bytes()?;
```

### `from_bytes(data: Bytes) -> Result<PrivateArchive, rmp_serde::decode::Error>`

Deserialize an archive from bytes.

```rust
let archive = PrivateArchive::from_bytes(bytes)?;
```

## Client Methods

### `archive_put(&self, archive: &PrivateArchive, payment_option: PaymentOption) -> Result<(AttoTokens, PrivateArchiveDataMap), PutError>`

Upload a Private Archive to the network. Returns the cost and the data map needed to retrieve the archive. This data map should be kept private.

```rust
let (cost, archive_data_map) = client
    .archive_put(&archive, PaymentOption::Wallet(wallet))
    .await?;
```

### `archive_get(&self, addr: &PrivateArchiveDataMap) -> Result<PrivateArchive, GetError>`

Retrieve a Private Archive from the network using its data map.

```rust
let archive = client.archive_get(&archive_data_map).await?;
```

## Example Usage

{% tabs %}
{% tab title="Rust" %}
```rust
use autonomi::{Client, PrivateArchive};
use autonomi::client::files::Metadata;
use autonomi::client::payment::PaymentOption;
use test_utils::evm::get_funded_wallet;
use std::path::PathBuf;
use eyre::Result;

#[tokio::main]
async fn main() -> Result<()> {
    let client = Client::init_local().await?;
    let wallet = get_funded_wallet();

    // Create a new archive
    let mut archive = PrivateArchive::new();

    // Upload files and add them to the archive
    let file_path = PathBuf::from("sensitive_document.pdf");
    let (_, data_map) = client
        .file_content_upload(file_path.clone(), PaymentOption::from(&wallet))
        .await?;
    
    let metadata = Metadata {
        created: 1234567890,
        modified: 1234567890,
        size: 1024,
        extra: Some(r#"{"encrypted": true, "algorithm": "AES-256"}"#.to_string()),
    };
    
    archive.add_file(file_path, data_map, metadata);

    // Upload the archive
    let (cost, archive_data_map) = client
        .archive_put(&archive, PaymentOption::from(&wallet))
        .await?;

    println!("Private archive uploaded for {}", cost);
    // IMPORTANT: Store archive_data_map securely - it's needed to retrieve the archive

    // Wait for network replication
    tokio::time::sleep(tokio::time::Duration::from_secs(5)).await;

    // Later, retrieve the archive using the data map
    let retrieved_archive = client.archive_get(&archive_data_map).await?;
    
    for (path, _, meta) in retrieved_archive.iter() {
        println!("File: {} ({} bytes)", path.display(), meta.size);
    }

    Ok(())
}
```
{% endtab %}

{% tab title="Python" %}
```python
import asyncio
import json
from autonomi import Client, PrivateArchive, Metadata, PaymentOption, Wallet, EVMNetwork

async def private_archive_example():
    # Initialize client and payment
    client = await Client.init_local()
    
    # Create wallet from private key
    wallet = Wallet.new_from_private_key(
        EVMNetwork(is_custom=True), 
        "0xac0974bec39a17e36ba4a6b4d238ff944bacb478cbed5efcae784d7bf4f2ff80"
    )
    payment = PaymentOption.wallet(wallet)
    
    # Create a new archive
    archive = PrivateArchive()
    
    # Upload a file and get its data map (private)
    file_path = "sensitive_document.pdf"
    cost, data_map = await client.file_content_upload(file_path, payment)
    print(f"File uploaded for: {cost} AttoTokens")
    
    # Create metadata with extra information
    extra_info = json.dumps({"encrypted": True, "algorithm": "AES-256"})
    metadata = Metadata(
        created=1234567890,
        modified=1234567890,
        size=1024,
        extra=extra_info
    )
    
    # Add file to private archive
    archive.add_file(file_path, data_map, metadata)
    
    # Upload the archive (returns private data map)
    archive_cost, archive_data_map = await client.archive_put(archive, payment)
    print(f"Private archive uploaded for: {archive_cost} AttoTokens")
    print("IMPORTANT: Store archive_data_map securely!")
    
    # Wait for replication
    await asyncio.sleep(5)
    
    # Retrieve the archive using the private data map
    retrieved_archive = await client.archive_get(archive_data_map)
    
    # List all files in the archive
    files = retrieved_archive.files()
    data_maps = retrieved_archive.data_maps()
    for i, ((path, meta), data_map_chunk) in enumerate(zip(files, data_maps)):
        print(f"File: {path} ({meta.size} bytes)")
        if meta.extra:
            extra_data = json.loads(meta.extra)
            print(f"  Extra info: {extra_data}")

# Run the example
asyncio.run(private_archive_example())
```
{% endtab %}

{% tab title="Node.js" %}
```js
import { Client, PrivateArchive, Metadata, PaymentOption, Wallet, Network } from '@withautonomi/autonomi'

async function privateArchiveExample() {
  // Initialize client and payment
  const client = await Client.initLocal()
  
  // Create wallet from private key
  const wallet = Wallet.newFromPrivateKey(
    new Network(true), // local network
    "0xac0974bec39a17e36ba4a6b4d238ff944bacb478cbed5efcae784d7bf4f2ff80"
  )
  const payment = PaymentOption.fromWallet(wallet)
  
  // Create a new archive
  const archive = new PrivateArchive()
  
  // Upload a file and get its data map (private)
  const filePath = "sensitive_document.pdf"
  const {cost, dataMap} = await client.fileContentUpload(filePath, payment)
  console.log(`File uploaded for: ${cost} AttoTokens`)
  
  // Create metadata with extra information
  const extraInfo = JSON.stringify({encrypted: true, algorithm: "AES-256"})
  const metadata = new Metadata({
    created: 1234567890,
    modified: 1234567890,
    size: 1024,
    extra: extraInfo
  })
  
  // Add file to private archive
  archive.addFile(filePath, dataMap, metadata)
  
  // Upload the archive (returns private data map)
  const {cost: archiveCost, dataMap: archiveDataMap} = await client.archivePut(archive, payment)
  console.log(`Private archive uploaded for: ${archiveCost} AttoTokens`)
  console.log("IMPORTANT: Store archiveDataMap securely!")
  
  // Wait for replication
  await new Promise(resolve => setTimeout(resolve, 5000))
  
  // Retrieve the archive using the private data map
  const retrievedArchive = await client.archiveGet(archiveDataMap)
  
  // List all files in the archive
  const files = retrievedArchive.files()
  const dataMaps = retrievedArchive.dataMaps()
  files.forEach((file, i) => {
    const [path, meta] = file
    const dataMapChunk = dataMaps[i]
    console.log(`File: ${path} (${meta.size} bytes)`)
    if (meta.extra) {
      const extraData = JSON.parse(meta.extra)
      console.log(`  Extra info:`, extraData)
    }
  })
}

// Run the example
privateArchiveExample().catch(console.error)
```
{% endtab %}
{% endtabs %}

## Privacy Considerations

* The `PrivateArchiveDataMap` returned from `archive_put` must be kept secret - anyone with this data map can retrieve and decrypt the archive
* Unlike Public Archives, the data maps are embedded within the archive, making the file structure itself private
* Consider storing the archive data map in a Vault for secure long-term storage

## See Also

* Files API - High-level file upload/download operations
* Advanced Files Management - Advanced archive manipulation
* Public Archive API Reference - Public archive operations
* Vault API Reference - Secure storage for private data maps
