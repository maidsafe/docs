# Public Archive

A Public Archive is a structure that maps file paths to data addresses on the network. While it can be used to store a single file with its metadata, it is generally used to organize multiple files in a hierarchical structure to simulate directories. Public Archives support nested paths and store metadata (creation time, modification time, size) for each file.

The key feature of Public Archives is that they store references (addresses) to files already uploaded to the network. When you retrieve a Public Archive, you get the addresses of the files, which you can then use to download the actual file content. The archive itself is uploaded to the network as chunks, and can be retrieved using its address.

## Structure

```rust
pub struct PublicArchive {
    // Path of the file in the directory -> (Data address, Metadata)
    map: BTreeMap<PathBuf, (DataAddress, Metadata)>,
}
```

## Instance Methods

### `new() -> Self`

Create a new empty Public Archive.

```rust
let archive = PublicArchive::new();
```

### `add_file(&mut self, path: PathBuf, data_addr: DataAddress, meta: Metadata)`

Add a file to the archive with its network address and metadata.

```rust
archive.add_file(
    PathBuf::from("documents/report.pdf"),
    data_address,
    metadata
);
```

### `rename_file(&mut self, old_path: &Path, new_path: &Path) -> Result<(), RenameError>`

Rename a file within the archive. This only changes the path in the archive structure, not the actual data on the network.

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

### `addresses(&self) -> Vec<DataAddress>`

Get all data addresses stored in the archive.

```rust
let addresses = archive.addresses();
```

### `iter(&self) -> impl Iterator<Item = (&PathBuf, &DataAddress, &Metadata)>`

Iterate over all items in the archive.

```rust
for (path, addr, metadata) in archive.iter() {
    println!("{}: {:?}", path.display(), addr);
}
```

### `map(&self) -> &BTreeMap<PathBuf, (DataAddress, Metadata)>`

Get direct access to the underlying map structure.

```rust
let map = archive.map();
```

### `merge(&mut self, other: &PublicArchive)`

Merge another archive into this one. Files from the other archive will be added to this archive.

```rust
archive.merge(&other_archive);
```

### `to_bytes(&self) -> Result<Bytes, rmp_serde::encode::Error>`

Serialize the archive to bytes for storage.

```rust
let bytes = archive.to_bytes()?;
```

### `from_bytes(data: Bytes) -> Result<PublicArchive, rmp_serde::decode::Error>`

Deserialize an archive from bytes.

```rust
let archive = PublicArchive::from_bytes(bytes)?;
```

## Client Methods

### `archive_put_public(&self, archive: &PublicArchive, payment_option: PaymentOption) -> Result<(AttoTokens, ArchiveAddress), PutError>`

Upload a Public Archive to the network. Returns the cost and the address where the archive can be retrieved.

```rust
let (cost, archive_address) = client
    .archive_put_public(&archive, PaymentOption::Wallet(wallet))
    .await?;
```

### `archive_get_public(&self, addr: &ArchiveAddress) -> Result<PublicArchive, GetError>`

Retrieve a Public Archive from the network using its address.

```rust
let archive = client.archive_get_public(&archive_address).await?;
```

### `archive_cost(&self, archive: &PublicArchive) -> Result<AttoTokens, CostError>`

Calculate the cost to upload an archive without actually uploading it.

```rust
let cost = client.archive_cost(&archive).await?;
```

## Example Usage

{% tabs %}
{% tab title="Rust" %}
```rust
use autonomi::{Client, PublicArchive};
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
    let mut archive = PublicArchive::new();

    // Upload files and add them to the archive
    let file_path = PathBuf::from("document.pdf");
    let (_, file_addr) = client
        .file_content_upload_public(file_path.clone(), PaymentOption::from(&wallet))
        .await?;
    
    let metadata = Metadata {
        created: 1234567890,
        modified: 1234567890,
        size: 1024,
        extra: None,
    };
    
    archive.add_file(file_path, file_addr, metadata);

    // Upload the archive
    let (cost, archive_addr) = client
        .archive_put_public(&archive, PaymentOption::from(&wallet))
        .await?;

    println!("Archive uploaded for {} to address: {:?}", cost, archive_addr);

    // Wait for network replication
    tokio::time::sleep(tokio::time::Duration::from_secs(5)).await;

    // Later, retrieve the archive
    let retrieved_archive = client.archive_get_public(&archive_addr).await?;
    
    for (path, addr, meta) in retrieved_archive.iter() {
        println!("File: {} ({} bytes) at address: {:?}", 
                 path.display(), meta.size, addr);
    }

    Ok(())
}
```
{% endtab %}

{% tab title="Python" %}
```python
import asyncio
from autonomi import Client, PublicArchive, Metadata, PaymentOption, Wallet, EVMNetwork

async def public_archive_example():
    # Initialize client and payment
    client = await Client.init_local()
    
    # Create wallet from private key
    wallet = Wallet.new_from_private_key(
        EVMNetwork(is_custom=True), 
        "0xac0974bec39a17e36ba4a6b4d238ff944bacb478cbed5efcae784d7bf4f2ff80"
    )
    payment = PaymentOption.wallet(wallet)
    
    # Create a new archive
    archive = PublicArchive()
    
    # Upload a file and get its address
    file_path = "document.pdf"
    cost, file_addr = await client.file_content_upload_public(file_path, payment)
    print(f"File uploaded for: {cost} AttoTokens")
    
    # Create metadata for the file
    metadata = Metadata(
        created=1234567890,
        modified=1234567890, 
        size=1024,
        extra=None
    )
    
    # Add file to archive
    archive.add_file(file_path, file_addr, metadata)
    
    # Upload the archive
    archive_cost, archive_addr = await client.archive_put_public(archive, payment)
    print(f"Archive uploaded for: {archive_cost} AttoTokens")
    print(f"Archive address: {archive_addr}")
    
    # Wait for replication
    await asyncio.sleep(5)
    
    # Retrieve the archive
    retrieved_archive = await client.archive_get_public(archive_addr)
    
    # List all files in the archive
    files = retrieved_archive.files()
    addresses = retrieved_archive.addresses()
    for i, ((path, meta), addr) in enumerate(zip(files, addresses)):
        print(f"File: {path} ({meta.size} bytes) at address: {addr}")

# Run the example
asyncio.run(public_archive_example())
```
{% endtab %}

{% tab title="Node.js" %}
```js
import { Client, PublicArchive, Metadata, PaymentOption, Wallet, Network } from '@withautonomi/autonomi'

async function publicArchiveExample() {
  // Initialize client and payment
  const client = await Client.initLocal()
  
  // Create wallet from private key
  const wallet = Wallet.newFromPrivateKey(
    new Network(true), // local network
    "0xac0974bec39a17e36ba4a6b4d238ff944bacb478cbed5efcae784d7bf4f2ff80"
  )
  const payment = PaymentOption.fromWallet(wallet)
  
  // Create a new archive
  const archive = new PublicArchive()
  
  // Upload a file and get its address
  const filePath = "document.pdf"
  const {cost, addr: fileAddr} = await client.fileContentUploadPublic(filePath, payment)
  console.log(`File uploaded for: ${cost} AttoTokens`)
  
  // Create metadata for the file
  const metadata = new Metadata({
    created: 1234567890,
    modified: 1234567890,
    size: 1024,
    extra: null
  })
  
  // Add file to archive
  archive.addFile(filePath, fileAddr, metadata)
  
  // Upload the archive
  const {cost: archiveCost, addr: archiveAddr} = await client.archivePutPublic(archive, payment)
  console.log(`Archive uploaded for: ${archiveCost} AttoTokens`)
  console.log(`Archive address: ${archiveAddr}`)
  
  // Wait for replication
  await new Promise(resolve => setTimeout(resolve, 5000))
  
  // Retrieve the archive
  const retrievedArchive = await client.archiveGetPublic(archiveAddr)
  
  // List all files in the archive
  const files = retrievedArchive.files()
  const addresses = retrievedArchive.addresses()
  files.forEach((file, i) => {
    const [path, meta] = file
    const addr = addresses[i]
    console.log(`File: ${path} (${meta.size} bytes) at address: ${addr}`)
  })
}

// Run the example
publicArchiveExample().catch(console.error)
```
{% endtab %}
{% endtabs %}

## See Also

* Files API - High-level file upload/download operations
* Advanced Files Management - Advanced archive manipulation
* Private Archive API Reference - Private archive operations
