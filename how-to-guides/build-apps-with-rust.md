# Build Apps with Rust

This guide will help you get started with Autonomi starting from scratch. This guide has 4 parts:

1. [Prerequisites](build_apps_with_rust.md#prerequisites)
2. [Create a local testnet](build_apps_with_rust.md#create-a-local-testnet)
3. [Connect to the testnet with Rust](build_apps_with_rust.md#connect-to-the-testnet-with-rust)
4. [Upload and retrieve data with Rust](build_apps_with_rust.md#upload-and-retrieve-data-with-rust)

> This has guide has been tested on MacOS, it should work on Linux or other unixes as well, but the commands might be slightly different for Windows (unless you are using [WSL](https://learn.microsoft.com/en-us/windows/wsl/install)).

## Prerequisites

First let's install the required tools to get started:

* The [**Rust toolchain installed**](https://www.rust-lang.org/tools/install), for running the testnet
* [**Anvil**](https://book.getfoundry.sh/getting-started/installation): to run a local Ethereum testnet

Once all the above is ready, let's proceed to create a local testnet. For this testnet we will use the following Ethereum wallet which is the default address for our testnet.

> The default private key and address (public key) for the testnet is:
>
> ```bash
> SECRET_KEY=0xac0974bec39a17e36ba4a6b4d238ff944bacb478cbed5efcae784d7bf4f2ff80
> ADDRESS=0xf39Fd6e51aad88F6F4ce6aB8827279cffFb92266
> ```
>
> It owns all the money on the testnet, you can use it to play around with the testnet! Which we will do in the next steps. **Don't send real money to this address!!**

## Create a local testnet

{% hint style="info" %}
An app is currently in development to make this a one-click process, but for now we need to run the testnet manually.
{% endhint %}

First let's clone the Autonomi repository:

```bash
git clone https://github.com/maidsafe/autonomi
cd autonomi
```

Then to run the testnet, run the following command:

```bash
cargo run --bin evm-testnet
```

Keep the terminal open and running.

In a separate terminal, in the same directory (autonomi), run the following command:

```bash
cargo run --bin antctl -- local run --build --rewards-address=0xf39Fd6e51aad88F6F4ce6aB8827279cffFb92266
```

You now have a local autonomi testnet running! Congrats! 🎉

## Connect to the testnet with Rust

Let's create a Rust project that interacts with the testnet. First let's setup a working environment and add [autonomi](https://crates.io/crates/autonomi) as a dependency.

```bash
# Create a new Rust project
cargo new autonomi-app
cd autonomi-app

# Add the autonomi crate and tokio (for the async runtime) as a dependency
cargo add autonomi
cargo add tokio
```

Your `Cargo.toml` should look something like this:

```toml
[package]
name = "autonomi-app"
version = "0.1.0"
edition = "2021"

[dependencies]
autonomi = "0.3.6"
tokio = "1.43.0"
```

Open up `src/main.rs` in your favorite editor and add the following code:

```rust
use autonomi::{Client, Network, Wallet};

#[tokio::main]
async fn main() {
    // Initialize a wallet with the testnet private key
    let private_key = "0xac0974bec39a17e36ba4a6b4d238ff944bacb478cbed5efcae784d7bf4f2ff80";
    let network = Network::new(true).expect("Failed to create network");
    let wallet = Wallet::new_from_private_key(network, private_key).expect("Failed to create wallet");
    println!("Wallet address: {}", wallet.address());
    println!("Wallet balance: {}", wallet.balance_of_tokens().await.expect("Failed to get balance"));

    // Connect to the network
    let _client = Client::init_local().await.expect("Failed to connect to network");
    println!("Connected to network!");
}
```

In your terminal (in the `autonomi-app` directory), run the following command to compile and run the program:

```bash
cargo run
```

You should see the following output:

```bash
Wallet address: 0xf39Fd6e51aad88F6F4ce6aB8827279cffFb92266
Wallet balance: 2500000000000000000000000
Connected to network!
```

Congrats! You've just connected to the testnet! 🎉

## Upload and retrieve data with Rust

Next up let's upload some data to the testnet and retrieve it. We will be using the autonomi data API for this. Expanding upon our previous work, change the `src/main.rs` file to the following:

```rust
use autonomi::{Client, Network, Wallet, Bytes};
use autonomi::client::payment::PaymentOption;
use tokio::time::sleep;
use std::time::Duration;

#[tokio::main]
async fn main() {
    // Initialize a wallet with the testnet private key
    let private_key = "0xac0974bec39a17e36ba4a6b4d238ff944bacb478cbed5efcae784d7bf4f2ff80";
    let network = Network::new(true).expect("Failed to create network");
    let wallet = Wallet::new_from_private_key(network, private_key).expect("Failed to create wallet");
    println!("Wallet address: {}", wallet.address());
    println!("Wallet balance: {}", wallet.balance_of_tokens().await.expect("Failed to get balance"));

    // Connect to the network
    let client = Client::init_local().await.expect("Failed to connect to network");
    println!("Connected to network!");

    // Choose to pay using our wallet
    let payment_option = PaymentOption::Wallet(wallet);

    // Upload data as public (meaning anyone can read it with the address)
    let data = Bytes::from("Hello, Freedom!");
    let (price, addr) = client.data_put_public(data, payment_option).await.expect("Failed to upload data");
    println!("Data uploaded for {price} testnet ANT to: {addr}");

    // Wait for the data to be stored by the network
    sleep(Duration::from_secs(1)).await;

    // Later, we can retrieve the data
    let retrieved_data = client.data_get_public(&addr).await.expect("Failed to retrieve data");
    println!("Retrieved data: {}", String::from_utf8(retrieved_data.to_vec()).expect("Failed to decode data"));
}
```

> For private data, use the `data_put` and `data_get` methods instead!

Congrats! If you got this far, you are ready to start building apps that can store data on Autonomi! 🎉

## Going further

The API offers many other tools to interact with the Network which you can find here: [Autonomi API Docs](https://docs.autonomi.com/developers/api-reference/).

## Cleanup and Troubleshooting

To stop and cleanup after a testnet, run the following commands to kill all the nodes, the evm testnet and delete all Autonomi related files&#x20;

{% hint style="danger" %}
**Warning**: if you are running local live nodes or apps/clients on Autonomi **DO NOT DELETE THE WHOLE** Autonomi data FOLDER as you risk losing all your data). \
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

> If you are on Windows, the autonomi data folder is `C:\Users\<username>\AppData\Roaming\autonomi`. Note that the 3 programs above might end with `.exe`

## For hackers

A **ONE LINER** I like to use to start a testnet (and the evm testnet too), and stop everything on `CTRL+C`:

```bash
# macOS
rm -rf $HOME/Library/Application\ Support/autonomi/; cargo run --bin evm-testnet& cargo run --bin antctl -- local run --build --rewards-address=0xf39Fd6e51aad88F6F4ce6aB8827279cffFb92266; (trap 'killall evm-testnet anvil antnode' SIGINT; cat)

# Linux
rm -rf $HOME/.local/share/autonomi/; cargo run --bin evm-testnet& cargo run --bin antctl -- local run --build --rewards-address=0xf39Fd6e51aad88F6F4ce6aB8827279cffFb92266; (trap 'killall evm-testnet anvil antnode' SIGINT; cat)
```

This has to be run in the autonomi directory (the one we cloned in [part 1](build_apps_with_rust.md#create-a-local-testnet)).
