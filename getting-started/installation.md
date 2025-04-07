# Installation Guide

## Prerequisites

Download the latest toolchain for you programming language:

{% tabs %}
{% tab title="Rust" %}
* Rust 1.83.0 or higher. (See [instructions](https://www.rust-lang.org/tools/install).)
{% endtab %}

{% tab title="Python" %}
* Python 3.8 or higher. (See [downloads](https://www.python.org/downloads/).)
{% endtab %}
{% endtabs %}

## API-specific Installation

Choose the APIs you need for your project:

### Autonomi Client

{% tabs %}
{% tab title="Rust" %}
```bash
cargo add autonomi
```
{% endtab %}

{% tab title="Python" %}
```bash
pip install autonomi-client
```
{% endtab %}

{% tab title="Node.js" %}
```bash
npm install `@withautonomi/autonomi`
```
{% endtab %}
{% endtabs %}

### Ant Node

{% tabs %}
{% tab title="Rust" %}
```bash
cargo add ant-node
```
{% endtab %}

{% tab title="Python" %}
```bash
pip install antnode
```
{% endtab %}

{% tab title="Node.js" %}
Ant Node is not available for Node.js currently.
{% endtab %}
{% endtabs %}

### BLS Threshold Crypto

{% tabs %}
{% tab title="Rust" %}
```bash
cargo add blsttc
```
{% endtab %}

{% tab title="Python" %}
```bash
pip install blsttc
```
{% endtab %}

{% tab title="Node.js" %}
`blsttc` is not available for Node.js currently.
{% endtab %}
{% endtabs %}

### Self Encryption

{% tabs %}
{% tab title="Rust" %}
```bash
cargo add self_encryption
```
{% endtab %}

{% tab title="Python" %}
```bash
pip install self-encryption
```
{% endtab %}

{% tab title="Node.js" %}
Self Encryption is not available for Node.js currently.
{% endtab %}
{% endtabs %}

## Verifying Installation

Test your installation by running a simple client initialization:

{% tabs %}
{% tab title="Rust" %}
```rust
use autonomi::Client;

let client = Client::init().await.expect("Could not initialize the client");
println!("Client initialized successfully");
```
{% endtab %}

{% tab title="Python" %}
```python
from autonomi_client import Client

client = await Client.init()
print('Client initialized successfully')
```
{% endtab %}

{% tab title="Node.js" %}
```js
import { Client } from '@withautonomi/autonomi'

const client = await Client.init()
console.log('Client initialized successfully')
```
{% endtab %}
{% endtabs %}

## Next Steps

* Quick Start Guides:
  * [Quick Start Guide](../how-to-guides/quick-start-guide.md)
  * [Python app Tutorial](../how-to-guides/build-apps-with-python.md)
  * [Rust app Tutorial](../how-to-guides/build-apps-with-rust.md)
  * [Local Network Setup](../how-to-guides/local-network.md)
* API References:
  * [Autonomi Client](../api-reference/autonomi-client/)
  * [Ant Node](../api-reference/ant-node.md)
  * [BLS Threshold Crypto](../api-reference/blsttc.md)
  * [Self Encryption](../api-reference/self-encryption.md)
