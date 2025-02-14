# Advanced Archives and Files management

Although the basic API methods for uploading and downloading files are sufficient for simple use cases, there are more advanced methods for managing files and archives that are useful for certain scenarios such as modifying an Archive without re-uploading all the already uploaded files. The use of this API along with the Pointers API can create the illusion of a regular mutable file system. 

### Archives

Here's the definition of the Public and Private Archive structs:

```rust
pub struct PublicArchive {
    ///           Path of the file in the directory
    ///           |         Data address of the content of the file (points to a DataMap)
    ///           |         |         Metadata of the file
    ///           |         |         |
    ///           V         V         V
    map: BTreeMap<PathBuf, (DataAddr, Metadata)>,
}

pub struct PrivateArchive {
    ///           Path of the file in the directory
    ///           |         DataMap of the chunks of this file
    ///           |         |             Metadata of the file
    ///           |         |             |
    ///           V         V             V
    map: BTreeMap<PathBuf, (DataMapChunk, Metadata)>,
}
```

Notice that what differentiates the Public and Private Archives is that the Public Archive contains an address to the DataMap of the file contents, while the Private Archive contains the DataMap itself. 

Files and Archives, whether public or private, are always uploaded as encrypted chunks. The key difference lies in the handling of the decryption "key"—the DataMap (cf [self-encryption](../self-encryption.md)). With public data, the DataMap is uploaded as an unencrypted chunk, allowing anyone to retrieve the data using the chunk’s address. In contrast, private data does not include the DataMap in the upload, ensuring that only the encrypted chunks are available. More generally, this encapsulates the fundamental difference between public and private data on the Network: public data makes the DataMap openly accessible, while private data keeps it hidden.

Each file in the Archive has a Metadata struct which contains the following fields:

```rust
pub struct Metadata {
    /// File creation time on local file system. See [`std::fs::Metadata::created`] for details per OS.
    pub created: u64,
    /// Last file modification time taken from local file system. See [`std::fs::Metadata::modified`] for details per OS.
    pub modified: u64,
    /// File size in bytes
    pub size: u64,
    /// Optional extra metadata with undefined structure, e.g. JSON.
    pub extra: Option<String>,
}
```

The last field: `extra` can be used to store any kind of additional metadata for the file. 

### Advanced Methods for Private Files

* **dir\_content\_upload**\
  Manually upload all the files in a directory to the network, returns the cost of the upload and the Private Archive that contains all the files' DataMaps and Metadata. The Archive itself is not uploaded, it is the responsibility of the user to upload it separately. Use this if you want to modify the Archive Metadata before uploading it.
* **file\_content\_upload**\
  Upload the contents of a single file to the network, returns the cost of the upload and the DataMap. The DataMap is not uploaded to the network as doing so would make the file contents public.
* **archive\_put**\
  Upload a private archive to the network, returns the cost of the upload and the PrivateArchiveAccess (the Archive DataMap) which can be used to fetch this Archive. The DataMap is not uploaded to the network as doing so would make the archive contents public.
* **archive\_get**\
  Get a private archive from the network using the PrivateArchiveAccess (the Archive DataMap). 

### Advanced Methods for Public Files

* **dir\_content\_upload\_public**\
  Manually upload all the files in a directory to the network, returns the cost of the upload and the Public Archive that contains all the files' Addresses and Metadata. The Archive itself is not uploaded, it is the responsibility of the user to upload it separately. Use this if you want to modify the Archive Metadata before uploading it.
* **file\_content\_upload\_public**\
  Upload the contents of a single file to the network, returns the cost of the upload and the Address to the file (address of the uploaded DataMap for that file). As the DataMap is uploaded to the network, the file content is public. Anyone with the Address can download the file content.
* **archive\_put\_public**\
  Upload a public archive to the network, returns the cost of the upload and the Address to fetch that Archive
* **archive\_get\_public**\
  Get a public archive from the network using the Address of the Archive.

### Example

```rust
use autonomi::{Client, Bytes};
use autonomi::client::payment::PaymentOption;
use autonomi::client::files::Metadata;

#[tokio::main]
async fn main() {
    // initialize a local client and test wallet
    let client = Client::init_local().await.unwrap();
    let wallet = get_funded_wallet();
    let payment_option = PaymentOption::Wallet(wallet.into());

    // upload a directory
    let (cost, mut archive) = client
        .dir_content_upload("path/to/a/directory".into(), payment_option.clone())
        .await?;
    println!("cost to upload private directory: {cost:?}");
    println!("archive: {:#?}", archive);

    // upload a file separately
    let (cost, file_datamap) = client
        .file_content_upload("path/to/another/file".into(), payment_option.clone())
        .await?;
    println!("cost to upload additional file: {cost:?}");

    // add that file to the archive with some metadata (could be anything you like)
    let custom_metadata = Metadata {
        created: 424242,
        modified: 424242,
        size: 126,
        extra: Some("{ \"content\": \"why not some json?\" }".to_string()),
    };
    archive.add_file("the_file_i_forgot".into(), file_datamap, custom_metadata);

    // upload the archive which now includes an additional file
    let (cost, archive_datamap) = client.archive_put(&archive, payment_option.clone()).await?;
    println!("cost to upload archive: {cost:?}");

    // download the entire directory now including that additional file
    let dest = "path/to/downloads";
    client.dir_download(&archive_datamap, dest.into()).await?;
}
```