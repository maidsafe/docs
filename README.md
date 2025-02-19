# Developer Documentation

Welcome to the Autonomi documentation… these guides will help you get started building with the Autonomi Network.

## What is Autonomi?

Autonomi is a decentralised data and communications platform designed to provide complete privacy, security, and freedom by distributing data across a peer-to-peer network, rather than relying on centralised servers. Through end-to-end encryption, self-authentication, and the allocation of storage and bandwidth from users’ own devices, it seeks to create an autonomous, self-sustaining system where data ownership remains firmly in the hands of individuals rather than corporations.

## Quick Links

* Quick Start Guides
  * [Installation Guide](getting-started/installation.md)
  * [Quick Start Guide](how-to-guides/quick-start-guide.md)
  * [Local Network Setup](how-to-guides/local-network.md)
  * [Build Apps with Python](how-to-guides/build-apps-with-python.md)
  * [Build Apps with Rust](how-to-guides/build-apps-with-rust.md)
* Core Concepts:
  * [Data Types](core-concepts/data-types.md) - Understanding the fundamental data structures
  * [Data Storage](core-concepts/data-storage.md) - How data is stored and retrieved
  * [Local Network Setup](how-to-guides/local-network.md) - Setting up a local development environment

### API References

* [Autonomi Client](api-reference/autonomi-client/) - Core client library for network operations
* [Ant Node](api-reference/ant-node.md) - Node implementation for network participation
* [BLS Threshold Crypto](api-reference/blsttc.md) - Threshold cryptography implementation
* [Self Encryption](api-reference/self-encryption.md) - Content-based encryption library
* Low-level [Rust Crate API Reference](https://docs.rs/autonomi/latest/autonomi/)

## Language Support

Autonomi provides client libraries for multiple languages:

{% tabs %}
{% tab title="Rust" %}
```rust
use autonomi::Client;

let client = Client::init()?;
```
{% endtab %}

{% tab title="Python" %}
```python
from autonomi-client import Client

client = Client()
await client.init()
```
{% endtab %}
{% endtabs %}

## Building from Source

{% tabs %}
{% tab title="Rust" %}
```rust
# Clone the repository
git clone <https://github.com/maidsafe/autonomi.git>
cd autonomi

# Build the project
cargo build --release

# Run tests
cargo test --all-features

# Install locally
cargo install --path .
```
{% endtab %}

{% tab title="Python" %}
```python
# Clone the repository
git clone https://github.com/maidsafe/autonomi.git
cd autonomi

# Create and activate virtual environment
uv venv
source .venv/bin/activate  # Unix
# or
.venv\Scripts\activate     # Windows

# Installs `maturin`
uv sync

# Build and install the package
maturin develop --uv
```
{% endtab %}
{% endtabs %}

## Contributing

We welcome contributions! Here's how you can help:

1. [Fork the repository](https://github.com/maidsafe/autonomi/tree/main)
2. Create a feature branch
3. Make your changes
4. Submit a pull request

## Getting Help

* [GitHub Issues](https://github.com/maidsafe/autonomi/issues)
