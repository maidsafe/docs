# Data

The Data API enables storage of arbitrarily sized blobs on the Autonomi Network by leveraging the Chunk native data type. Under the hood, data is self-encrypted, splitting the payload into multiple encrypted chunks that are dispersed across the network. A DataMap, which contains the addresses of these chunks, is generated to facilitate data retrieval and decryption. Depending on the desired access control, the DataMap can be kept private for confidential data storage or published as a chunk, thereby making the data publicly available via the DataMap’s address.

* **Immutable:** Permanent Immutable/Incorruptible storage
* **Size:** The data can be of any size
* **Access Control:** Data can be private or public

### Client Methods

* **data\_cost**\
  Estimates the storage cost for a piece of data.

### Client Methods for Private Data (the default on the Network)

#### Raw Data Operations

* **data\_get**\
  Retrieves bytes from the network using a data map.
* **data\_put**\
  Uploads bytes to the network and returns the price along with the data map to access the data. (This data map is to be kept secret)

#### File Operations

* **file\_content\_upload**\
  Uploads the contents of a single file from the local filesystem to the network and returns the cost and DataMap. The DataMap is not uploaded to the network as doing so would make the file contents public.

#### Directory Operations

* **dir\_upload**\
  Upload a directory to the network. Uploads all the files in the directory along with a Private Archive which contains the metadata for the directory. Returns the total cost of the upload and the Private Archive Access (an Archive DataMap) which can be used to download the directory. The directory is read from the local filesystem.
* **dir\_download**\
  Download a directory from the network using the Private Archive Access (the Archive DataMap). Writes the directory to the local filesystem.

### Client Methods for Public Data

#### Raw Data Operations

* **data\_get\_public**\
  Retrieves bytes from the network using an address.
* **data\_put\_public**\
  Uploads bytes to the network and returns the price along with the address to access the data. (This address can be published so that others can access the data)

#### File Operations

* **file\_content\_upload\_public**\
  Uploads the contents of a single file from the local filesystem to the network and returns the cost and Address. The DataMap is uploaded to the network making the file contents publicly accessible.

#### Directory Operations

* **dir\_upload\_public**\
  Upload a directory to the network. Uploads all the files in the directory along with a Public Archive which contains the metadata for the directory. Returns the total cost of the upload and the Public Archive Address (the address of the Archive's DataMap which is also uploaded to the network, making it publicly retrievable). The directory is read from the local filesystem.
* **dir\_download\_public**\
  Download a directory from the network using the Public Archive Address. Writes the directory to the local filesystem.

## Examples

### Data, File and Directory Operations

{% tabs %}
{% tab title="Rust" %}
```rust
use autonomi::{Client, Bytes};
use autonomi::client::payment::PaymentOption;
use eyre::Result;

async fn data_example() -> Result<()> {
    // initialize a local client and test wallet
    let client = Client::init_local().await?;
    let wallet = get_funded_wallet();

    let data = autonomi::Bytes::from("Hello, World!");
    let cost = client.data_cost(data.clone()).await?;
    println!("cost estimate of uploading data: {cost}");

    // Upload raw data (public)
    let (cost, addr) = client.data_put_public(data.clone(), wallet.into()).await?;
    println!("public data put for: {cost} at address: {addr}");

    // Upload raw data (private)
    let (cost, datamap) = client.data_put(data.clone(), PaymentOption::Wallet(wallet.clone().into())).await?;
    println!("private data put for: {cost}");

    // Upload a single file (public)
    let file_path = "path/to/single/file.txt";
    let (cost, file_addr) = client.file_content_upload_public(file_path.into(), &wallet).await?;
    println!("uploaded file to network for: {cost}");

    // Upload a single file (private)
    let (cost, file_datamap) = client.file_content_upload(file_path.into(), PaymentOption::Wallet(wallet.clone().into())).await?;
    println!("uploaded private file for: {cost}");

    // Upload a directory (public)
    let dir_path = "path/to/directory/to/upload";
    let (cost, dir_addr) = client.dir_upload_public(dir_path.into(), &wallet).await?;
    println!("uploaded directory to network for: {cost}");

    // Upload a directory (private)
    let (cost, archive_datamap) = client.dir_upload(dir_path.into(), PaymentOption::Wallet(wallet.into())).await?;
    println!("uploaded private directory for: {cost}");

    // Wait for the data to be replicated
    tokio::time::sleep(tokio::time::Duration::from_secs(5)).await;

    // Download public data
    let data_fetched = client.data_get_public(&addr).await?;
    assert_eq!(data, data_fetched, "data fetched should match data put");

    // Download private data
    let private_data = client.data_get(&datamap).await?;
    println!("Downloaded private data: {}", String::from_utf8_lossy(&private_data));

    // Download public file data
    let file_data = client.data_get_public(&file_addr).await?;
    println!("Downloaded file data: {} bytes", file_data.len());

    // Download private file data
    let private_file_data = client.data_get(&file_datamap).await?;
    println!("Downloaded private file data: {} bytes", private_file_data.len());

    // Download public directory
    let download_path = "path/where/download/happens/";
    client.dir_download_public(&dir_addr, download_path.into()).await?;
    println!("Downloaded public directory from network");

    // Download private directory
    client.dir_download(&archive_datamap, download_path.into()).await?;
    println!("Downloaded private directory from network");

    Ok(())
}
```
{% endtab %}

{% tab title="Python" %}
```python
import asyncio
from autonomi_client import Client, PaymentOption

async def main():
    # Initialize a local client and test wallet
    client = await Client.init_local()
    wallet = get_funded_wallet()
    
    data = b"Hello, World!"
    cost = await client.data_cost(data)
    print(f"Cost estimate of uploading data: {cost}")
    
    # Upload raw data (public)
    cost, addr = await client.data_put_public(data, PaymentOption.wallet(wallet))
    print(f"Public data put for: {cost} at address: {addr}")
    
    # Upload raw data (private)
    cost, datamap = await client.data_put(data, PaymentOption.wallet(wallet))
    print(f"Private data put for: {cost}")
    
    # Upload a single file (public)
    cost, file_addr = await client.file_content_upload_public("path/to/single/file.txt", PaymentOption.wallet(wallet))
    print(f"Uploaded file to network for: {cost}")
    
    # Upload a single file (private)
    cost, file_datamap = await client.file_content_upload("path/to/single/file.txt", PaymentOption.wallet(wallet))
    print(f"Uploaded private file for: {cost}")
    
    # Upload a directory (public)
    cost, dir_addr = await client.dir_upload_public("path/to/directory/to/upload", wallet)
    print(f"Uploaded directory to network for: {cost}")
    
    # Upload a directory (private)
    cost, archive_datamap = await client.dir_upload("path/to/directory/to/upload", PaymentOption.wallet(wallet))
    print(f"Uploaded private directory for: {cost}")
    
    # Wait for the data to be replicated
    await asyncio.sleep(5)
    
    # Download public data
    data_fetched = await client.data_get_public(addr)
    assert data == data_fetched, "Data fetched should match data put"
    
    # Download private data
    private_data = await client.data_get(datamap)
    print(f"Downloaded private data: {private_data.decode()}")
    
    # Download public file data
    file_data = await client.data_get_public(file_addr)
    print(f"Downloaded file data: {len(file_data)} bytes")
    
    # Download private file data
    private_file_data = await client.data_get(file_datamap)
    print(f"Downloaded private file data: {len(private_file_data)} bytes")
    
    # Download public directory
    await client.dir_download_public(dir_addr, "path/where/download/happens/")
    print("Downloaded public directory from network")
    
    # Download private directory
    await client.dir_download(archive_datamap, "path/where/download/happens/")
    print("Downloaded private directory from network")

if __name__ == "__main__":
    asyncio.run(main())
```
{% endtab %}

{% tab title="Node.js" %}
```javascript
import { Client, PaymentOption } from '@withautonomi/autonomi';

async function main() {
    // Initialize a local client and test wallet
    const client = await Client.initLocal();
    const wallet = getFundedWallet();
    
    const data = Buffer.from("Hello, World!");
    const cost = await client.dataCost(data);
    console.log(`Cost estimate of uploading data: ${cost}`);
    
    // Upload raw data (public)
    const { cost: publicCost, addr } = await client.dataPutPublic(data, PaymentOption.fromWallet(wallet));
    console.log(`Public data put for: ${publicCost} at address: ${addr}`);
    
    // Upload raw data (private)
    const { cost: privateCost, datamap } = await client.dataPut(data, PaymentOption.fromWallet(wallet));
    console.log(`Private data put for: ${privateCost}`);
    
    // Upload a single file (public)
    const { cost: fileCost, addr: fileAddr } = await client.fileContentUploadPublic("path/to/single/file.txt", PaymentOption.fromWallet(wallet));
    console.log(`Uploaded file to network for: ${fileCost}`);
    
    // Upload a single file (private)
    const { cost: privateFileCost, dataMap: fileDatamap } = await client.fileContentUpload("path/to/single/file.txt", PaymentOption.fromWallet(wallet));
    console.log(`Uploaded private file for: ${privateFileCost}`);
    
    // Upload a directory (public)
    const { cost: dirCost, addr: dirAddr } = await client.dirUploadPublic("path/to/directory/to/upload", PaymentOption.fromWallet(wallet));
    console.log(`Uploaded directory to network for: ${dirCost}`);
    
    // Upload a directory (private)
    const { cost: privateDirCost, dataMap: archiveDatamap } = await client.dirUpload("path/to/directory/to/upload", PaymentOption.fromWallet(wallet));
    console.log(`Uploaded private directory for: ${privateDirCost}`);
    
    // Wait for the data to be replicated
    await new Promise(resolve => setTimeout(resolve, 5000));
    
    // Download public data
    const dataFetched = await client.dataGetPublic(addr);
    console.assert(data.equals(dataFetched), "Data fetched should match data put");
    
    // Download private data
    const privateData = await client.dataGet(datamap);
    console.log(`Downloaded private data: ${privateData.toString()}`);
    
    // Download public file data
    const fileData = await client.dataGetPublic(fileAddr);
    console.log(`Downloaded file data: ${fileData.length} bytes`);
    
    // Download private file data
    const privateFileData = await client.dataGet(fileDatamap);
    console.log(`Downloaded private file data: ${privateFileData.length} bytes`);
    
    // Download public directory
    await client.dirDownloadPublic(dirAddr, "path/where/download/happens/");
    console.log("Downloaded public directory from network");
    
    // Download private directory
    await client.dirDownload(archiveDatamap, "path/where/download/happens/");
    console.log("Downloaded private directory from network");
}

main().catch(console.error);
```
{% endtab %}
{% endtabs %}
