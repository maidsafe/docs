# API Reference Overview

Autonomi provides several APIs for different aspects of the system:

## Client API

The [Autonomi Client API](autonomi-client/) is the core library for interacting with the Autonomi network. It provides:

* [Chunk Operations](autonomi-client/chunks.md)
* [Pointer Management](autonomi-client/pointer.md)
* [Graph Operations](autonomi-client/graphentry.md)
* [Scratchpad Operations](autonomi-client/scratchpad.md)
* [Register Operations](autonomi-client/register.md)
* [Vault Operations](autonomi-client/vault.md)
* [BLS Cryptographic Keys Handling](../core-concepts/bls-keys.md)

## Node API

The [Ant Node API](ant-node.md) allows you to run and manage nodes in the Autonomi network. Features include:

* Node setup and configuration
* Network participation
* Storage management
* Reward collection
* Event handling

## Cryptography APIs

### BLS Threshold Crypto

The [BLS Threshold Crypto API](blsttc.md) implements BLS (Boneh-Lynn-Shacham) threshold signatures, providing:

* Secret key generation and sharing
* Signature creation and verification
* Threshold signature schemes
* Key aggregation

### Self Encryption

The [Self Encryption API](self-encryption.md) implements content-based encryption, offering:

* Data-derived encryption
* Content deduplication
* Parallel processing
* Streaming interface

## Language Support

All APIs are available in multiple languages:

* Python (3.8+)
* Rust (stable)

Each API section includes language-specific installation instructions and code examples.
