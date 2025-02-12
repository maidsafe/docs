# Start building apps in Python with Autonomi

This guide will help you get started with building apps with Autonomi starting from scratch. This guide has 4 parts:

1. [Prerequisites](#prerequisites)
2. [Create a local testnet](#create-a-local-testnet)
3. [Connect to the testnet with Python](#connect-to-the-testnet-with-python)
4. [Upload and retrieve data with Python](#upload-and-retrieve-data-with-python)

> This has guide has been tested on MacOS, it should work on Linux or other unixes as well, but the commands might be slightly different for Windows (unless you are using [WSL](https://learn.microsoft.com/en-us/windows/wsl/install)).

## Prerequisites

First let's install the required tools to get started:

- **Python 3.9+**
- **[The uv Python package manager](https://docs.astral.sh/uv/getting-started/installation/)**: to manage the python environment and install the dependencies, although you can use any other package manager you want.
- **The [Rust toolchain installed](https://www.rust-lang.org/tools/install)**, for running the testnet
- **[Anvil](https://book.getfoundry.sh/getting-started/installation)**: to run a local Ethereum testnet

Once all the above is ready, let's proceed to create a local testnet. For this testnet we will use the following Ethereum wallet which is the default address for our testnet. 

> The default private key and address (public key) for the testnet is:
> ```bash
> SECRET_KEY=0xac0974bec39a17e36ba4a6b4d238ff944bacb478cbed5efcae784d7bf4f2ff80
> ADDRESS=0xf39Fd6e51aad88F6F4ce6aB8827279cffFb92266
> ```
> It owns all the money on the testnet, you can use it to play around with the testnet! Which we will do in the next steps.
> **Don't send real money to this address!!**

## Create a local testnet

> An app is currently in development to make this process a one click thing, but for now we need to run the testnet manually. 

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

## Connect to the testnet with Python

Let's create a python project that interacts with the testnet. First let's setup a working environment and install the [autonomi-client](https://pypi.org/project/autonomi-client/) package.

```bash
# Create a new virtual environment in the current directory
uv venv

# Install the autonomi-client package
uv pip install autonomi-client
```

Then create a new python file in which we will write our app:

```bash
touch main.py
```

Open up the file in your favorite editor and add the following code:

```python
from autonomi_client import Client, Network, Wallet
import asyncio

async def main():
    # Initialize a wallet with the testnet private key
    private_key = "0xac0974bec39a17e36ba4a6b4d238ff944bacb478cbed5efcae784d7bf4f2ff80"
    network = Network(True)
    wallet = Wallet.new_from_private_key(network, private_key)
    print(f"Wallet address: {wallet.address()}")
    print(f"Wallet balance: {await wallet.balance()}")

    # Connect to the network
    client = await Client.init_local()
    print(f"Connected to network!")

# Run the main in an async runtime
asyncio.run(main())
```

In your terminal, run the following command to execute the script:

```bash
python main.py
```

You should see the following output:

```bash
Wallet address: 0xf39Fd6e51aad88F6F4ce6aB8827279cffFb92266
Wallet balance: 2500000000000000000000000
Connected to network!
```

Congrats! You've just connected to the testnet! 🎉

## Upload and retrieve data with Python

Next up let's upload some data to the testnet and retrieve it. We will be using the autonomi data API for this. Expanding upon our previous work, change the `main.py` file to the following:

```python
from autonomi_client import Client, Network, Wallet, PaymentOption
import asyncio

async def main():
    # Initialize a wallet with the testnet private key
    private_key = "0xac0974bec39a17e36ba4a6b4d238ff944bacb478cbed5efcae784d7bf4f2ff80"
    network = Network(True)
    wallet = Wallet.new_from_private_key(network, private_key)
    print(f"Wallet address: {wallet.address()}")
    print(f"Wallet balance: {await wallet.balance()}")

    # Connect to the network
    client = await Client.init_local()
    print("Connected to network!")

    # Choose to pay using our wallet
    payment_option = PaymentOption.wallet(wallet)

    # Upload data as public (meaning anyone can read it with the address)
    data = b"Hello, Freedom!"
    (price, addr) = await client.data_put_public(data, payment_option)
    print(f"Data uploaded for {price} testnet ANT to: {addr}")

    # Wait for the data to be stored by the network
    await asyncio.sleep(1)

    # Later, we can retrieve the data
    retrieved_data = await client.data_get_public(addr)
    print(f"Retrieved data: {retrieved_data.decode()}")

# Run the main in an async runtime
asyncio.run(main())
```

> For private data, use the `data_put` and `data_get` methods instead!

Congrats! If you got this far, you are ready to start building apps that can store data on Autonomi! 🎉

## Going further

The API offers many other tools to interact with the Network which you can find here: [Autonomi API Docs](https://docs.autonomi.com/developers/api-reference/python-api-reference).

## Cleanup and Troubleshooting

To stop and cleanup after a testnet, run the following commands to kill all the nodes, the evm testnet and delete all autonomi related files (**WARNING**: if you are running local live nodes or apps/clients on autonomi **DO NOT DELETE THE WHOLE** autonomi data FOLDER you risk losing all your data). It is recommended to run testnets on a separate user or machine. 

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
rm -rf $HOME/Library/Application\ Support/autonomi/; cargo run --bin evm-testnet& cargo run --bin antctl -- local run --build --rewards-address=0xf39Fd6e51aad88F6F4ce6aB8827279cffFb92266; cat || killall evm-testnet anvil antnode

# Linux
rm -rf $HOME/.local/share/autonomi/; cargo run --bin evm-testnet& cargo run --bin antctl -- local run --build --rewards-address=0xf39Fd6e51aad88F6F4ce6aB8827279cffFb92266; cat || killall evm-testnet anvil antnode
```

> has to be run in the autonomi directory (the one we cloned in [part 1](#create-a-local-testnet))