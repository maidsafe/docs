# Local Network Setup Guide

This guide explains how to set up and run a local Autonomi network for development and testing purposes.

## Prerequisites

* Rust toolchain (with `cargo` installed)
* Git (for cloning the repository)

That's it! Everything else needed will be built from source.

## Starting a Network

1. Clone the repository:

```bash
git clone https://github.com/maidsafe/autonomi
cd autonomi
```

2. Start a local EVM testnet:

```bash
cargo run --bin evm-testnet 
```

3. Start a local Autonomi network:

```bash
cargo run --bin antctl -- local run --build --clean --rewards-address 0xf39Fd6e51aad88F6F4ce6aB8827279cffFb92266 evm-local
```

Done!
