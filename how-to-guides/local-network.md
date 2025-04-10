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

Keep the terminal open and running.

3. In a separate terminal, in the same directory (`autonomi`), run the following command:

```bash
cargo run --bin antctl -- local run --build --clean --rewards-address 0xf39Fd6e51aad88F6F4ce6aB8827279cffFb92266 evm-local
```

Done!

## Connecting to the testnet

You can now connect to the testnet with various APIs, for example: [Rust](./build-apps-with-rust.md), [Python](./build-apps-with-python.md) and [Node.js](./build-apps-with-nodejs.md). It's also possible to use the [Ant CLI](https://crates.io/crates/ant-cli), just remember to pass the `--local` flag.

## Cleanup and Troubleshooting

To stop and cleanup after a testnet, run the following commands to kill all the nodes, the evm testnet and delete all Autonomi related files.

{% hint style="danger" %}
**Warning:** If you are running local live nodes or apps/clients on Autonomi **DO NOT DELETE THE WHOLE** Autonomi data _FOLDER_ asyou risk losing all your data. \
\
It is recommended to run testnets on a separate user or machine.
{% endhint %}

```bash
# macOS
killall evm-testnet anvil antnode
rm -rf $HOME/Library/Application\ Support/autonomi/

# Linux
killall evm-testnet anvil antnode
rm -rf $HOME/.local/share/autonomi/
```

If you are on Windows, the Autonomi data folder is here:

`C:\Users\<username>\AppData\Roaming\autonomi`.

Note that the 3 programs above might end with `.exe`

## For hackers

A **ONE LINER** I like to use to start a testnet (and the evm testnet too), and stop everything on `CTRL+C`:

```bash
# macOS
rm -rf $HOME/Library/Application\ Support/autonomi/; cargo run --bin evm-testnet& cargo run --bin antctl -- local run --build --rewards-address=0xf39Fd6e51aad88F6F4ce6aB8827279cffFb92266; (trap 'killall evm-testnet anvil antnode' SIGINT; cat)

# Linux
rm -rf $HOME/.local/share/autonomi/; cargo run --bin evm-testnet& cargo run --bin antctl -- local run --build --rewards-address=0xf39Fd6e51aad88F6F4ce6aB8827279cffFb92266; (trap 'killall evm-testnet anvil antnode' SIGINT; cat)
```

{% hint style="info" %}
Note: this has to be run in the autonomi directory (the one we [cloned earlier](local-network.md#starting-a-network))
{% endhint %}
