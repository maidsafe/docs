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
{% endtabs %}

## Verifying Installation

Test your installation by running a simple client initialization:

{% tabs %}
{% tab title="Rust" %}
```rust
use autonomi::Client;

let client = Client::init_local().await?;
println!("Client initialized successfully");
```
{% endtab %}

{% tab title="Python" %}
```python
from autonomi import Client

client = await Client.init_local()
print('Client initialized successfully')
```
{% endtab %}
{% endtabs %}

## Next Steps

* API References:
  * [Autonomi Client](../api-reference/autonomi-client/)
  * [Ant Node](../api-reference/ant-node/)
  * [BLS Threshold Crypto](../api-reference/blsttc.md)
  * [Self Encryption](../api-reference/self-encryption.md)
* [Local Network Setup](../how-to-guides/local_network.md)
