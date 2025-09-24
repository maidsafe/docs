# Build Apps with Python

This guide will help you get started with building apps with Autonomi starting from scratch. This guide has 4 parts:

1. [Prerequisites](build-apps-with-python.md#prerequisites)
2. [Create a local testnet](build-apps-with-python.md#create-a-local-testnet)
3. [Connect to the testnet with Python](build-apps-with-python.md#connect-to-the-testnet-with-python)
4. [Upload and retrieve data with Python](build-apps-with-python.md#upload-and-retrieve-data-with-python)

{% hint style="info" %}
This has guide has been tested on MacOS, it should work on Linux or other unixes as well, but the commands might be slightly different for Windows (unless you are using [WSL](https://learn.microsoft.com/en-us/windows/wsl/install)).
{% endhint %}

## Prerequisites

First let's install the required tools to get started:

* **Python 3.9+**
* [**The uv Python package manager**](https://docs.astral.sh/uv/getting-started/installation/): to manage the python environment and install the dependencies, although you can use any other package manager you want.
* **The** [**Rust toolchain installed**](https://www.rust-lang.org/tools/install), for running the testnet
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
An app is currently in development to make this process a one click thing, but for now we need to run the testnet manually.
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

You now have a local Autonomi testnet running! Congrats! 🎉

## Connect to the testnet with Python

Let's create a python project that interacts with the testnet. First let's setup a working environment and install the [autonomi-client](https://pypi.org/project/autonomi-client/) package.

```bash
# Create a new virtual environment in the current directory
uv venv

# Install the autonomi-client package
uv pip install autonomi-client
```

Then create a new Python file in which we will write our app:

```bash
touch main.py
```

Open up the file in your favorite editor and add the following code:

```python
from autonomi_client import Client, EVMNetwork, Wallet
import asyncio

async def main():
    # Initialize a wallet with the testnet private key
    private_key = "0xac0974bec39a17e36ba4a6b4d238ff944bacb478cbed5efcae784d7bf4f2ff80"
    network = EVMNetwork(True)
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

Next up let's upload some data to the testnet and retrieve it. We will be using the Autonomi data API for this. Expanding upon our previous work, change the `main.py` file to the following:

```python
from autonomi_client import Client, EVMNetwork, Wallet, PaymentOption
import asyncio

async def main():
    # Initialize a wallet with the testnet private key
    private_key = "0xac0974bec39a17e36ba4a6b4d238ff944bacb478cbed5efcae784d7bf4f2ff80"
    network = EVMNetwork(True)
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

## Working with Registers

Registers are mutable data structures that allow you to store updateable content with versioned history. Here's how to use them:

```python
from autonomi_client import Client, EVMNetwork, Wallet, PaymentOption, SecretKey
import asyncio

async def register_example():
    # Initialize client and wallet (same as before)
    private_key = "0xac0974bec39a17e36ba4a6b4d238ff944bacb478cbed5efcae784d7bf4f2ff80"
    network = EVMNetwork(True)
    wallet = Wallet.new_from_private_key(network, private_key)
    client = await Client.init_local()
    payment_option = PaymentOption.wallet(wallet)
    
    # Generate keys for register
    main_key = SecretKey.random()
    register_key = Client.register_key_from_name(main_key, "my-app-config")
    
    # Create register content (max 32 bytes)
    initial_config = Client.register_value_from_bytes(b"version: 1.0")
    
    # Check cost and create register
    cost = await client.register_cost(register_key.public_key())
    print(f"Register creation cost: {cost} AttoTokens")
    
    creation_cost, address = await client.register_create(
        register_key, initial_config, payment_option)
    print(f"Register created at: {address.to_hex()}")
    
    # Wait for network replication
    await asyncio.sleep(5)
    
    # Read current value
    current_value = await client.register_get(address)
    print(f"Current config: {current_value}")
    
    # Update the register
    updated_config = Client.register_value_from_bytes(b"version: 1.1")
    update_cost = await client.register_update(register_key, updated_config, payment_option)
    print(f"Update cost: {update_cost} AttoTokens")
    
    # Wait for replication
    await asyncio.sleep(5)
    
    # Get complete history
    history = client.register_history(address)
    all_versions = await history.collect()
    print(f"Config history ({len(all_versions)} versions):")
    for i, version in enumerate(all_versions):
        print(f"  {i}: {version}")

# Run the example
asyncio.run(register_example())
```

### Key Register Features:

* **Mutable**: Can be updated with new content while preserving history
* **Versioned**: All previous versions are permanently accessible
* **Owned**: Only the key holder can update the register
* **32-byte limit**: Each register value is limited to 32 bytes
* **Paid updates**: Both creation and updates require payment

### Common Register Use Cases:

1. **Application Configuration**: Store app settings that need updates
2. **Status Tracking**: Maintain current state with full history
3. **Version Control**: Track document versions
4. **Counters**: Implement distributed counters
5. **Metadata**: Store changeable file or app metadata

## Going further

The API offers many other tools to interact with the Network which you can find here: [Autonomi API Docs](https://docs.autonomi.com/developers/api-reference/python-api-reference).

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

If you are on Windows, the Autonomi data folder is:&#x20;

`C:\Users\<username>\AppData\Roaming\autonomi`.&#x20;

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
Note: this has to be run in the autonomi directory (the one we cloned in [part 1](build-apps-with-python.md#create-a-local-testnet))
{% endhint %}
