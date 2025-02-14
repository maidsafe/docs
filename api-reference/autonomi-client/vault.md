# Vaults

Vaults are a scalable, encrypted storage space on the Autonomi Network designed for secure management of diverse data types. They serve as a dedicated repository for storing the DataMaps of private data uploads, as well as keys and other personal information—whether network-related or not. Essentially, a Vault acts as your personal account, maintaining references to all data you have stored on the Network.
Vaults are a high level data type in the Autonomi Network. Which means they are a client side construct built on top of the lower level native data types.

* **key-addressed:** Network address is derived from a BLS public key
* **Scalable:** Size grows as you add more data
* **Encrypted:** Data is always encrypted
* **Mutable:** You can add or remove data from the vault at any time for free
* **Pay to expand:** Vaults can be expanded for a fee

### Client Methods

* **fetch\_and\_decrypt\_vault**  
  Retrieves and returns a decrypted vault if one exists. Returns the decrypted bytes along with the vault content type.
* **write\_bytes\_to\_vault**  
  Puts data into the client's Vault, dynamically expanding the vault capacity by paying for additional space when needed. If the Vault does not exist, it will be created.
* **vault\_cost**  
  Estimates the cost of creating a new vault. The cost is computed based on the desired initial size of the vault.

### Example

```rust
use autonomi::Client;
use autonomi::client::payment::PaymentOption;
use autonomi::{SecretKey, Bytes};
use autonomi::vault::app_name_to_vault_content_type;

#[tokio::main]
async fn main() {
    // initialize a local client and test wallet
    let client = Client::init_local().await.unwrap();
    let wallet = get_funded_wallet();
    let payment_option = PaymentOption::Wallet(wallet.clone());

    // each key can have a vault which will be addressed at the public key (easy to find!)
    let main_key = SecretKey::random();

    // the content type can be any integer, it is used to identify the type of data in the vault
    let content_type = app_name_to_vault_content_type("Choose a unique name for your app here");
    let original_content = Bytes::from_static(b"Store any data you want in the vault");

    // if it doesn't already exist, the vault will be created
    let cost = client
        .write_bytes_to_vault(
            original_content.clone(),
            payment_option.clone(),
            &main_key,
            content_type,
        )
        .await
        .unwrap();
    println!("Vault creation cost: {cost}");

    // fetch the vault and decrypt it
    let (fetched_content, fetched_content_type) = client.fetch_and_decrypt_vault(&main_key).await.unwrap();

    // it should match the original content
    assert_eq!(fetched_content_type, content_type);
    assert_eq!(fetched_content, original_content);

    // the vault can be updated at will
    let new_content = Bytes::from_static(b"Update the vault with new data");
    let cost = client
        .write_bytes_to_vault(
            new_content.clone(),
            payment_option.clone(),
            &main_key,
            content_type,
        )
        .await
        .unwrap();
    println!("Vault update cost should be 0: {cost}");
}
```
