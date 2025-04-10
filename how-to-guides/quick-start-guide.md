---
description: >-
  This section will help you get started on your Autonomi adventure as quickly
  as possible. It will walk you through setting up your development environment
  and writing a simple Autonomi app.
---

# Quick Start Guide

## My first App

Let's get right to it and build your first Autonomi app!

### Add Autonomi as a Dependency

First import our Autonomi dependency using the language you love:

{% tabs %}
{% tab title="Rust" %}
```rust
cargo add autonomi
```
{% endtab %}

{% tab title="Python" %}
pip install autonomi-client
{% endtab %}

{% tab title="Node.js" %}
```bash
npm install @withautonomi/autonomi
```
{% endtab %}
{% endtabs %}

### Setup a Client

To connect to the Autonomi network, we'll need a \`Client\`:

{% tabs %}
{% tab title="Rust" %}
```rust
use autonomi::Client;

#[tokio::main]
async fn main() {
    let client = Client::init()
        .await
        .expect("Could not initialize the client");
}
```
{% endtab %}

{% tab title="Python" %}
```python
from autonomi_client import Client
import asyncio

async def main():
    client = await Client.init()

asyncio.run(main())
```
{% endtab %}

{% tab title="Node.js" %}
```js
import { Client } from '@withautonomi/autonomi'

const client = await Client.init()
```
{% endtab %}
{% endtabs %}

### Download a Dog Picture

What better way is there to show off the capabilities of the network? Let's download a dog picture from this public data address:

```
a7d2fdbb975efaea25b7ebe3d38be4a0b82c1d71e9b89ac4f37bc9f8677826e0
```

{% tabs %}
{% tab title="Rust" %}
```rust
use autonomi::client::address::str_to_addr;
use autonomi::Client;

#[tokio::main]
async fn main() {
    let client = Client::init()
        .await
        .expect("Could not initialize the client");

    // Data address of the dog picture
    let data_address =
        DataAddress::from_hex("a7d2fdbb975efaea25b7ebe3d38be4a0b82c1d71e9b89ac4f37bc9f8677826e0")
            .expect("Invalid data address");

    // Get the bytes of the dog picture
    let bytes = client
        .data_get_public(&data_address)
        .await
        .expect("Could not fetch data from the network");

    // Write the bytes of the dog picture to a file
    std::fs::write("lucky.jpg", bytes).expect("Failed to write the file");
}
```
{% endtab %}

{% tab title="Python" %}
```python
from autonomi_client import Client
import asyncio

async def main():
    client = await Client.init()
    
    # Data address of the dog picture
    data_address = "a7d2fdbb975efaea25b7ebe3d38be4a0b82c1d71e9b89ac4f37bc9f8677826e0"

    # Get the bytes of the dog picture
    dog_picture = await client.data_get_public(data_address)
    
    # Write the bytes of the dog picture to a file
    with open("lucky.jpg", "wb") as f:
        f.write(dog_picture)

asyncio.run(main())
```
{% endtab %}

{% tab title="Node.js" %}
```js
import { Client, DataAddress } from '@withautonomi/autonomi'
import fs from 'node:fs/promises'

const client = await Client.init()
// Data address of the dog picture
const dataAddress = DataAddress.fromHex("a7d2fdbb975efaea25b7ebe3d38be4a0b82c1d71e9b89ac4f37bc9f8677826e0")
// Get the bytes of the dog picture
const dogPicture = await client.dataGetPublic(dataAddress)

// Write the bytes of the dog picture to a file
await fs.writeFile('lucky.jpg', dogPicture);
```
{% endtab %}
{% endtabs %}

After running this code, you'll see a `lucky.jpg` file downloaded to your work directory!
