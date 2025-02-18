# Payments Guide

This guide explains how payments work in Autonomi, particularly for put operations that store data on the network.

## Overview

When storing data on the Autonomi network, you need to pay for the storage space. Payments are made using an ERC-20 token through a smart contract system. There are two ways to handle payments:

1. Direct payment using an EVM wallet
2. Pre-paid operations using a receipt

## Payment Options

### Directly Using an EVM Wallet

The simplest way to pay for put operations is to use an EVM wallet:

{% tabs %}
{% tab title="Rust" %}
```rust
use autonomi::{Client, Wallet};

// Initialize client
let client = Client::init().await?;

let private_key = "0xac0974bec39a17e36ba4a6b4d238ff944bacb478cbed5efcae784d7bf4f2ff80";

// Create or load a wallet
let wallet = Wallet::new_from_private_key(client.evm_network().clone(), private_key)?;

// Put data with wallet payment
let data = b"Hello, World!".to_vec();
let address = client.data_put_public(data.into(), wallet.into()).await?;
```
{% endtab %}
{% endtabs %}

### Using a Receipt

Work in progress..

## Testing Payments

When testing your application, you can use the local development environment which provides a test EVM network with a pre-funded wallet. See the [Local Network Setup](local-network.md) Guide for details.
