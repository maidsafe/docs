# Installation Guide

## Prerequisites

* Rust (latest stable)
* Python 3.8 or higher

## API-specific Installation

Choose the APIs you need for your project:

### Autonomi Client

{% tabs %}
{% tab title="Rust" %}
```toml
# Add to Cargo.toml:
[dependencies]
autonomi = "0.3.1"
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
```toml
[dependencies]
ant-node = "0.3.2"
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
```toml
[dependencies]
blsttc = "8.0.2"
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
```toml
[dependencies]
self_encryption = "0.28.0"
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
