# Scratchpad

Scratchpad is a native data type in the Autonomi Network:

* **Key-addressed:** Network address is derived from a BLS public key
* **Size:** Up to 4MB mutable storage
* **Pay-once, free updates:** Unlimited updates for free after initial payment
* **Versioned & signed:** Includes a version counter and cryptographically verifiable signatures
* **Encrypted:** Content is automatically encrypted with the owner's key

## Client Methods

### scratchpad\_get\_from\_public\_key

Retrieves a scratchpad from the network using the owner's public key.

**Input:**

* `public_key: &PublicKey` - The owner's public key

**Output:**

* **Rust:** `Result<Scratchpad, ScratchpadError>`
* **Python:** `Scratchpad` or raises exception
* **Node.js:** `Promise<Scratchpad>`

### scratchpad\_get

Retrieves a scratchpad from the network by its address.

**Input:**

* `address: &ScratchpadAddress` - The scratchpad address

**Output:**

* **Rust:** `Result<Scratchpad, ScratchpadError>`
* **Python:** `Scratchpad` or raises exception
* **Node.js:** `Promise<Scratchpad>`

### scratchpad\_check\_existence

Checks if a scratchpad exists on the network. Much faster than `scratchpad_get`.

**Input:**

* `address: &ScratchpadAddress` - The scratchpad address

**Output:**

* **Rust:** `Result<bool, ScratchpadError>`
* **Python:** `bool` or raises exception
* **Node.js:** `Promise<boolean>`

### scratchpad\_verify

Verifies a scratchpad's signature and size constraints.

**Input:**

* `scratchpad: &Scratchpad` - The scratchpad to verify

**Output:**

* **Rust:** `Result<(), ScratchpadError>`
* **Python:** `None` or raises exception
* **Node.js:** `void` (throws on error)

### scratchpad\_put

Manually uploads a scratchpad to the network. Requires the scratchpad to be already created and signed.

**Input:**

* `scratchpad: Scratchpad` - The scratchpad to store
* `payment_option: PaymentOption` - Payment method

**Output:**

* **Rust:** `Result<(AttoTokens, ScratchpadAddress), ScratchpadError>`
* **Python:** `tuple[str, ScratchpadAddress]` (cost as string) or raises exception
* **Node.js:** `Promise<ScratchpadPut>` with `cost: string` and `addr: ScratchpadAddress`

### scratchpad\_create

Creates and uploads a new scratchpad to the network. Encrypts the content automatically.

**Input:**

* `owner: &SecretKey` - The owner's secret key
* `content_type: u64` - Application-specific content type identifier
* `initial_data: &Bytes` - The initial data to store
* `payment_option: PaymentOption` - Payment method

**Output:**

* **Rust:** `Result<(AttoTokens, ScratchpadAddress), ScratchpadError>`
* **Python:** `tuple[str, ScratchpadAddress]` (cost as string) or raises exception
* **Node.js:** `Promise<ScratchpadPut>` with `cost: string` and `addr: ScratchpadAddress`

### scratchpad\_update

Updates an existing scratchpad with new content. This operation is free.

**Input:**

* `owner: &SecretKey` - The owner's secret key
* `content_type: u64` - Content type identifier
* `data: &Bytes` - The new data to store

**Output:**

* **Rust:** `Result<(), ScratchpadError>`
* **Python:** `None` or raises exception
* **Node.js:** `Promise<void>`

### scratchpad\_update\_from

Updates an existing scratchpad from a specific scratchpad instance. Used internally by `scratchpad_update`.

**Input:**

* `current: &Scratchpad` - The current scratchpad
* `owner: &SecretKey` - The owner's secret key
* `content_type: u64` - Content type identifier
* `data: &Bytes` - The new data to store

**Output:**

* **Rust:** `Result<Scratchpad, ScratchpadError>`
* **Python:** `Scratchpad` or raises exception
* **Node.js:** Not directly exposed

### scratchpad\_cost

Estimates the storage cost for a new scratchpad.

**Input:**

* `owner: &PublicKey` - The owner's public key

**Output:**

* **Rust:** `Result<AttoTokens, CostError>`
* **Python:** `str` (cost as string) or raises exception
* **Node.js:** `Promise<string>`

## Language-Specific Type Differences

### Error Handling

* **Rust:** Uses `Result<T, ScratchpadError>` with detailed error variants
* **Python:** Raises exceptions with simplified error messages as strings
* **Node.js:** Uses Promise rejection with Error objects containing simplified messages

### Numeric Types

* **Rust:** Uses `u64` for content\_type, `AttoTokens` for costs
* **Python:** Uses `int` for content\_type, `str` for costs
* **Node.js:** Uses `bigint` for content\_type, `string` for costs

### Data Types

* **Rust:** Uses `Bytes` type for data
* **Python:** Uses `bytes` for data
* **Node.js:** Uses `Buffer` for data

## Examples

{% tabs %}
{% tab title="Rust" %}
```rust
use autonomi::{Client, SecretKey, AttoTokens, Bytes};
use autonomi::client::payment::PaymentOption;

async fn scratchpad_example() -> Result<(), Box<dyn std::error::Error>> {
    // Initialize client and wallet
    let client = Client::init_local().await?;
    let wallet = get_funded_wallet();
    let payment = PaymentOption::from(&wallet);

    // Create secret key for scratchpad
    let key = SecretKey::random();
    let public_key = key.public_key();
    
    // Check cost
    let cost = client.scratchpad_cost(&public_key).await?;
    println!("Scratchpad cost: {}", cost);

    // Create scratchpad
    let content_type = 42;
    let initial_data = Bytes::from("Hello, Autonomi!");
    let (cost, addr) = client
        .scratchpad_create(&key, content_type, &initial_data, payment.clone())
        .await?;
    println!("Created at {:?}, cost: {}", addr, cost);

    // Get scratchpad
    let scratchpad = client.scratchpad_get(&addr).await?;
    assert_eq!(scratchpad.counter(), 0);
    
    // Decrypt content
    let decrypted = scratchpad.decrypt_data(&key)?;
    assert_eq!(decrypted, initial_data);

    // Update scratchpad
    let new_data = Bytes::from("Updated content!");
    client.scratchpad_update(&key, content_type, &new_data).await?;

    // Get updated scratchpad
    let updated = client.scratchpad_get(&addr).await?;
    assert_eq!(updated.counter(), 1);
    let updated_content = updated.decrypt_data(&key)?;
    assert_eq!(updated_content, new_data);

    Ok(())
}
```
{% endtab %}

{% tab title="Python" %}
```python
import asyncio
from autonomi_client import Client, SecretKey, Wallet, PaymentOption, Network

async def scratchpad_example():
    # Initialize client and wallet
    client = await Client.init_local()
    network = Network(True)
    private_key = "0xac0974bec39a17e36ba4a6b4d238ff944bacb478cbed5efcae784d7bf4f2ff80"
    wallet = Wallet.new_from_private_key(network, private_key)
    payment = PaymentOption.wallet(wallet)

    # Create secret key for scratchpad
    key = SecretKey()
    public_key = key.public_key()

    try:
        # Check cost
        cost = await client.scratchpad_cost(public_key)
        print(f"Scratchpad cost: {cost}")

        # Create scratchpad
        content_type = 42
        initial_data = b"Hello, Autonomi!"
        cost, addr = await client.scratchpad_create(key, content_type, initial_data, payment)
        print(f"Created at {addr.hex}, cost: {cost}")

        # Get scratchpad
        scratchpad = await client.scratchpad_get(addr)
        assert scratchpad.counter() == 0
        
        # Decrypt content
        decrypted = scratchpad.decrypt_data(key)
        assert decrypted == initial_data

        # Update scratchpad (free)
        new_data = b"Updated content!"
        await client.scratchpad_update(key, content_type, new_data)

        # Get updated scratchpad
        updated = await client.scratchpad_get(addr)
        assert updated.counter() == 1
        updated_content = updated.decrypt_data(key)
        assert updated_content == new_data

    except Exception as e:
        print(f"Error: {e}")

asyncio.run(scratchpad_example()) 
```
{% endtab %}

{% tab title="Node.js" %}
```javascript
import { Client, SecretKey, Wallet, Network, PaymentOption } from '@withautonomi/autonomi';

async function scratchpadExample() {
    try {
        // Initialize client and wallet
        const client = await Client.initLocal();
        const network = new Network(true);
        const privateKey = "0xac0974bec39a17e36ba4a6b4d238ff944bacb478cbed5efcae784d7bf4f2ff80";
        const wallet = Wallet.newFromPrivateKey(network, privateKey);
        const payment = PaymentOption.fromWallet(wallet);

        // Create secret key for scratchpad
        const key = SecretKey.random();
        const publicKey = key.publicKey();

        // Check cost
        const cost = await client.scratchpadCost(publicKey);
        console.log(`Scratchpad cost: ${cost}`);

        // Create scratchpad
        const contentType = 42n;
        const initialData = Buffer.from("Hello, Autonomi!");
        const { cost: createCost, addr } = await client.scratchpadCreate(key, contentType, initialData, payment);
        console.log(`Created at ${addr.toHex()}, cost: ${createCost}`);

        // Get scratchpad
        const scratchpad = await client.scratchpadGet(addr);
        console.assert(scratchpad.counter() === 0n);
        
        // Decrypt content
        const decrypted = scratchpad.decryptData(key);
        console.assert(Buffer.compare(decrypted, initialData) === 0);

        // Update scratchpad (free)
        const newData = Buffer.from("Updated content!");
        await client.scratchpadUpdate(key, contentType, newData);

        // Get updated scratchpad
        const updated = await client.scratchpadGet(addr);
        console.assert(updated.counter() === 1n);
        const updatedContent = updated.decryptData(key);
        console.assert(Buffer.compare(updatedContent, newData) === 0);

        console.log("Scratchpad operations completed successfully!");

    } catch (error) {
        console.error('Error:', error.message);
    }
}

scratchpadExample();
```
{% endtab %}
{% endtabs %}

## Error Handling

{% tabs %}
{% tab title="Rust" %}
### Rust Errors

The `ScratchpadError` enum provides detailed error variants:

* `PutError` - Failed to store scratchpad
* `Pay` - Payment failure
* `GetError` - Failed to retrieve scratchpad
* `Corrupt` - Invalid scratchpad data
* `Serialization` - Serialization error
* `ScratchpadAlreadyExists` - Scratchpad already exists at address
* `CannotUpdateNewScratchpad` - Cannot update non-existent scratchpad
* `ScratchpadTooBig` - Exceeds maximum size (4MB)
* `BadSignature` - Invalid signature
* `Fork` - Multiple conflicting versions detected
{% endtab %}

{% tab title="Python" %}
### Python Error Handling

```python
try:
    scratchpad = await client.scratchpad_get(addr)
except Exception as e:
    if "RecordNotFound" in str(e):
        print("Scratchpad does not exist")
    elif "BadSignature" in str(e):
        print("Invalid scratchpad signature")
    else:
        print(f"Unexpected error: {e}")
```
{% endtab %}

{% tab title="Node.js" %}
### Node.js Error Handling

```javascript
try {
    const scratchpad = await client.scratchpadGet(addr);
} catch (error) {
    if (error.message.includes('RecordNotFound')) {
        console.log('Scratchpad does not exist');
    } else if (error.message.includes('BadSignature')) {
        console.log('Invalid scratchpad signature');
    } else {
        console.error('Unexpected error:', error.message);
    }
}
```
{% endtab %}
{% endtabs %}

{% hint style="warning" %}
Due to compatibility issues, Python and Node.js bindings simplify types (and error types to string messages within exceptions/rejections). For full experience and control it's recommended to use Rust.
{% endhint %}
