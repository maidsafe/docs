# Files

The Files API enables the storage of files and directories as immutable archives on the Autonomi Network. Archives leverage the same robust capabilities as the Data API by encrypting and splitting files into chunks that are dispersed across the network, with a DataMap maintaining the addresses for retrieval and decryption. Archives can be designated as public—uploaded to the network and retrievable via an address—or private, where the DataMap is kept confidential. For efficient management of numerous private archives, the Vault API offers a secure way to organize and safeguard these archives.

* **Immutable:** Permanent Immutable/Incorruptible storage
* **Metadata:** Archives keep track of metadata like size, creation date, and other properties
* **Access Control:** File archives can be private or public

### Client Methods for Private Files (the default on the Network)

* **dir\_upload**\
  Upload a directory to the network. Uploads all the files in the directory along with a Private Archive which contains the metadata for the directory.
  Returns the total cost of the upload and the Private Archive Access (an Archive DataMap) which can be used to download the directory. 
  The directory is read from the local filesystem.
* **dir\_download**\
  Download a directory from the network using the Private Archive Access (the Archive DataMap). Writes the directory to the local filesystem.

### Public Files

* **dir\_upload\_public**\
  Upload a directory to the network. Uploads all the files in the directory along with a Public Archive which contains the metadata for the directory.
  Returns the total cost of the upload and the Public Archive Address (the address of the Archive's DataMap which is also uploaded to the network, making it publicly retrievable). The directory is read from the local filesystem.
* **dir\_download\_public**\
  Download a directory from the network using the Public Archive Address. Writes the directory to the local filesystem.

### Example

```rust
use autonomi::{Client, Bytes};
use autonomi::client::payment::PaymentOption;

#[tokio::main]
async fn main() {
    // initialize a local client and test wallet
    let client = Client::init_local().await.unwrap();
    let wallet = get_funded_wallet();

    // Upload a directory to the network
    let path = "path/to/directory/to/upload";
    let (cost, addr) = client
        .dir_upload_public(path.into(), &wallet)
        .await.unwrap();
    println!("Uploaded directory to network for: {}", cost);

    // Wait for the data to be replicated
    tokio::time::sleep(tokio::time::Duration::from_secs(5)).await;

    // Download the directory from the network
    let download_path = "path/where/download/happens/";
    client
        .dir_download_public(&addr, download_path.into())
        .await.unwrap();
    println!("Downloaded directory from network for: {}", cost);
}
```

For more advanced file management see the [Advanced Files Management](./files-advanced.md) page.