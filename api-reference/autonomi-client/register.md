# Register

Registers are a high level data type in the Autonomi Network. Which means they are a client side construct built on top of the lower level native data types. Registers are a 32 bytes mutable memory unit with all versions history accessible.

* **key-addressed:** Network address is derived from a BLS public key
* **Size:** 32 bytes of content per entry
* **Mutable but immutable history:** Register old entries are kept forever but the register can be updated with new entries endlessly.
* **Pay to update:** Register updates are paid for, but cheaper than creation.
* **Signed:** Owned and signed by a key for cryptographic verification

### Client Methods

* **register\_get**\
  Retrieves a register from the network by its address.
* **register\_create**\
  Creates and uploads a new register for a payment. Returns the total cost and the register's address.
* **register\_update**\
  Updates an existing register with new content for a payment. The old content is kept in the history.
* **register\_cost**\
  Estimates the storage cost for a register.
* **register\_history**\
  Retrieves the history of a register.

#### API helpers for registers:

* **register\_key\_from\_name**\
  Creates a new BLS key deriving it from a name and a base key. This is used to be able to create multiple registers with the same key (and different names). cf [bls key derivation](bls_keys.md)
* **register\_value\_from\_bytes** Makes sure the content fits in 32 bytes.

## Examples

{% tabs %}
{% tab title="Rust" %}
```rust
use autonomi::Client;
use autonomi::client::payment::PaymentOption;
use autonomi::SecretKey;
use eyre::Result;

async fn register_example() -> Result<()> {
    // initialize a local client and test wallet
    let client = Client::init_local().await?;
    let wallet = get_funded_wallet();
    let main_key = SecretKey::random();

    // create a register key from a name and our main key
    let register_key = Client::register_key_from_name(&main_key, "register1");

    // estimate the cost of the register
    let cost = client.register_cost(&register_key.public_key()).await?;

    // create the register
    let content = Client::register_value_from_bytes(b"Hello, World!")?;
    let (_cost, addr) = client
        .register_create(&register_key, content, PaymentOption::from(&wallet))
        .await?;

    // let the network replicate the register
    tokio::time::sleep(tokio::time::Duration::from_secs(5)).await;

    // get the register
    let value = client.register_get(&addr).await?;
    assert_eq!(value, content);

    // update the register
    let content2 = Client::register_value_from_bytes(b"你好世界")?;
    client
        .register_update(&register_key, content2, PaymentOption::from(&wallet))
        .await?;

    // let the network replicate the updates
    tokio::time::sleep(tokio::time::Duration::from_secs(5)).await;

    let all = client.register_history(&addr).collect().await?;
    assert_eq!(all.len(), 2);
    assert_eq!(all[0], content);
    assert_eq!(all[1], content2);

    Ok(())
}
```
{% endtab %}

{% tab title="Python" %}
```python
from autonomi_client import Client, Wallet, Network, PaymentOption, SecretKey
import asyncio

async def register_example():
    # Initialize client and wallet
    client = await Client.init_local()
    wallet = Wallet.new_from_private_key(
        Network(True), 
        "0xac0974bec39a17e36ba4a6b4d238ff944bacb478cbed5efcae784d7bf4f2ff80"
    )
    payment = PaymentOption.wallet(wallet)
    
    # Generate keys
    main_key = SecretKey.random()
    register_key = Client.register_key_from_name(main_key, "my-counter")
    
    # Create register content (max 32 bytes)
    initial_content = Client.register_value_from_bytes(b"counter: 0")
    
    # Check cost before creating
    cost = await client.register_cost(register_key.public_key())
    print(f"Register creation cost: {cost} AttoTokens")
    
    # Create the register
    creation_cost, address = await client.register_create(
        register_key, initial_content, payment)
    print(f"Register created at: {address.to_hex()}")
    
    # Wait for network replication
    await asyncio.sleep(5)
    
    # Read current value
    current_value = await client.register_get(address)
    print(f"Current value: {current_value}")
    
    # Update the register
    new_content = Client.register_value_from_bytes(b"counter: 1")
    update_cost = await client.register_update(register_key, new_content, payment)
    print(f"Update cost: {update_cost} AttoTokens")
    
    # Wait for replication
    await asyncio.sleep(5)
    
    # Get updated value
    updated_value = await client.register_get(address)
    print(f"Updated value: {updated_value}")
    
    # Get complete history
    history = client.register_history(address)
    all_versions = await history.collect()
    print(f"History has {len(all_versions)} versions:")
    for i, version in enumerate(all_versions):
        print(f"  Version {i}: {version}")

# Run the example
asyncio.run(register_example())
```
{% endtab %}

{% tab title="Node.js" %}
```js
import { Client, Wallet, Network, PaymentOption, SecretKey } from '@withautonomi/autonomi'

async function registerExample() {
  // Initialize client and wallet
  const client = await Client.initLocal()
  const wallet = Wallet.newFromPrivateKey(
    new Network(true), 
    "0xac0974bec39a17e36ba4a6b4d238ff944bacb478cbed5efcae784d7bf4f2ff80"
  )
  const payment = PaymentOption.fromWallet(wallet)
  
  // Generate keys
  const mainKey = SecretKey.random()
  const registerKey = Client.registerKeyFromName(mainKey, "my-counter")
  
  // Create register content (max 32 bytes)
  const initialContent = Client.registerValueFromBytes(Buffer.from("counter: 0"))
  
  // Check cost before creating
  const cost = await client.registerCost(registerKey.publicKey())
  console.log(`Register creation cost: ${cost} AttoTokens`)
  
  // Create the register
  const {cost: creationCost, addr} = await client.registerCreate(
    registerKey, initialContent, payment
  )
  console.log(`Register created at: ${addr.toHex()}`)
  
  // Wait for network replication
  await new Promise(resolve => setTimeout(resolve, 5000))
  
  // Read current value
  const currentValue = await client.registerGet(addr)
  console.log(`Current value: ${currentValue}`)
  
  // Update the register
  const newContent = Client.registerValueFromBytes(Buffer.from("counter: 1"))
  const updateCost = await client.registerUpdate(registerKey, newContent, payment)
  console.log(`Update cost: ${updateCost} AttoTokens`)
  
  // Wait for replication
  await new Promise(resolve => setTimeout(resolve, 5000))
  
  // Get updated value
  const updatedValue = await client.registerGet(addr)
  console.log(`Updated value: ${updatedValue}`)
  
  // Get complete history
  const history = client.registerHistory(addr)
  const allVersions = await history.collect()
  console.log(`History has ${allVersions.length} versions:`)
  allVersions.forEach((version, i) => {
    console.log(`  Version ${i}: ${version}`)
  })
}

// Run the example
registerExample().catch(console.error)
```
{% endtab %}
{% endtabs %}

## Language-Specific Usage

### Python API

The Python client provides these register methods:

#### Methods

* **`register_cost(public_key: PublicKey) -> str`**  
  Calculate the cost to create a register
* **`register_create(key: SecretKey, content: RegisterValue, payment: PaymentOption) -> Tuple[str, RegisterAddress]`**  
  Create a new register on the network
* **`register_get(address: RegisterAddress) -> RegisterValue`**  
  Retrieve current value of a register  
* **`register_update(key: SecretKey, content: RegisterValue, payment: PaymentOption) -> str`**  
  Update an existing register with new content
* **`register_history(address: RegisterAddress) -> RegisterHistory`**  
  Get the complete version history of a register

#### Utility Functions

* **`Client.register_key_from_name(main_key: SecretKey, name: str) -> SecretKey`**  
  Generate a deterministic register key from a main key and name
* **`Client.register_value_from_bytes(data: bytes) -> RegisterValue`**  
  Create RegisterValue from byte data (limited to 32 bytes)

#### RegisterAddress Class

* **`RegisterAddress(public_key: PublicKey)`** - Create from owner's public key
* **`from_hex(hex: str) -> RegisterAddress`** - Create from hex string  
* **`to_hex() -> str`** - Convert to hex string
* **`owner() -> PublicKey`** - Get the public key that owns this register

### Node.js API

The Node.js client provides these register methods:

#### Methods

* **`client.registerCost(publicKey)`** - Calculate creation cost
* **`client.registerCreate(key, content, payment)`** - Create new register
* **`client.registerGet(address)`** - Get current register value
* **`client.registerUpdate(key, content, payment)`** - Update register content
* **`client.registerHistory(address)`** - Get version history iterator

#### Utility Functions

* **`Client.registerKeyFromName(mainKey, name)`** - Generate deterministic key
* **`Client.registerValueFromBytes(data)`** - Create RegisterValue from Buffer

#### RegisterAddress Class  

* **`new RegisterAddress(publicKey)`** - Create from public key
* **`RegisterAddress.fromHex(hex)`** - Create from hex string
* **`addr.toHex()`** - Convert to hex string  
* **`addr.owner()`** - Get owner public key

## Use Cases

1. **Mutable Configuration**: Store application settings that need periodic updates
2. **Status Updates**: Maintain current status or state information
3. **Version Control**: Track document or data versions with full history  
4. **Counters**: Implement distributed counters or sequence numbers
5. **Metadata**: Store changeable metadata for files or applications

## Important Notes

1. **Size Limit**: Register values are limited to 32 bytes maximum
2. **Ownership**: Only the key holder can update a register
3. **Network Delays**: Allow time for network replication after operations (typically 5+ seconds)
4. **Deterministic Keys**: Use `register_key_from_name()` for consistent key generation  
5. **Payment Required**: Both creation and updates require payment
6. **History Preserved**: All versions are permanently stored and accessible
7. **Error Handling**: 
   - Creating an existing register throws `AlreadyExists` error
   - Updating a non-existent register throws `CannotUpdateNewRegister` error
