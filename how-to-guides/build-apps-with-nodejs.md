# Build Apps with Node.js

This guide will help you get started with building apps with Autonomi starting from scratch. This guide has 4 parts:

1. [Prerequisites](build-apps-with-nodejs.md#prerequisites)
2. [Create a local testnet](build-apps-with-nodejs.md#create-a-local-testnet)
3. [Connect to the testnet with Node.js](build-apps-with-nodejs.md#connect-to-the-testnet-with-nodejs)
4. [Upload and retrieve data with Node.js](build-apps-with-nodejs.md#upload-and-retrieve-data-with-nodejs)

{% hint style="info" %}
This has guide has been tested on MacOS and Linux, but the commands might be slightly different for Windows (unless you are using [WSL](https://learn.microsoft.com/en-us/windows/wsl/install)).
{% endhint %}

## Prerequisites

First let's install the required tools to get started:

* [**Node.js**](https://nodejs.org/en/download) with its NPM command-line tool to install dependencies and run your code

Once Node.js is installed, let's proceed to create a local testnet.

## Create a local testnet

See [Local Network Setup](./local-network.md).

## Connect to the testnet with Node.js

Let's create a Node.js project that interacts with the testnet. First let's setup a working environment and install the [@withautonomi/autonomi](https://www.npmjs.com/package/@withautonomi/autonomi) package.

```bash
npm install @withautonomi/autonomi
```

Then create a new Node.js file in which we will write our app:

```bash
touch main.mjs
```

Open up the file in your favorite editor and add the following code:

```js
import { Client, Network, Wallet } from '@withautonomi/autonomi'

// Initialize a wallet with the testnet private key
const privateKey = "0xac0974bec39a17e36ba4a6b4d238ff944bacb478cbed5efcae784d7bf4f2ff80"
const wallet = Wallet.newFromPrivateKey(Network(true), privateKey)
console.log("Wallet address: ", wallet.address())
console.log("Wallet balance: ", await wallet.balance())
// Connect to the network
const client = await Client.initLocal()
console.log("Connected to network!")
```

In your terminal, run the following command to execute the script:

```bash
node main.mjs
```

You should see the following output:

```bash
Wallet address: 0xf39Fd6e51aad88F6F4ce6aB8827279cffFb92266
Wallet balance: 2500000000000000000000000
Connected to network!
```

Congrats! You've just connected to the testnet! 🎉

## Upload and retrieve data with Node.js

Next up let's upload some data to the testnet and retrieve it. We will be using the Autonomi data API for this. Expanding upon our previous work, change the `main.mjs` file to the following:

```js
import { Client, Network, Wallet, PaymentOption } from '@withautonomi/autonomi'

// Initialize a wallet with the testnet private key
const privateKey = "0xac0974bec39a17e36ba4a6b4d238ff944bacb478cbed5efcae784d7bf4f2ff80"
const wallet = Wallet.newFromPrivateKey(Network(true), privateKey)
console.log("Wallet address: ", wallet.address())
console.log("Wallet balance: ", await wallet.balance())
// Connect to the network
const client = await Client.initLocal()
console.log("Connected to network!")

// Choose to pay using our wallet
const paymentOption = PaymentOption.fromWallet(wallet)

// Upload data as public (meaning anyone can read it with the address)
const data = Buffer.from("Hello, World.")
const { cost, addr } = await client.dataPutPublic(data, paymentOption)
console.log("Data uploaded for ", cost, " testnet ANT to: ", addr)

// Wait for the data to be stored by the network
await new Promise(h => setTimeout(h, 1000))

// Later, we can retrieve the data
const retrievedData = await client.dataGetPublic(addr)
console.log("Retrieved data: ", retrievedData.toString())

```

> For private data, use the `dataPut` and `dataGet` methods instead!

Congrats! If you got this far, you are ready to start building apps that can store data on Autonomi! 🎉

## Working with Registers

Registers are mutable data structures that allow you to store updateable content with versioned history. Here's how to use them:

```js
import { Client, Network, Wallet, PaymentOption, SecretKey } from '@withautonomi/autonomi'

// Initialize client and wallet (same as before)
const privateKey = "0xac0974bec39a17e36ba4a6b4d238ff944bacb478cbed5efcae784d7bf4f2ff80"
const wallet = Wallet.newFromPrivateKey(new Network(true), privateKey)
const client = await Client.initLocal()
const paymentOption = PaymentOption.fromWallet(wallet)

// Generate keys for register
const mainKey = SecretKey.random()
const registerKey = Client.registerKeyFromName(mainKey, "my-app-config")

// Create register content (max 32 bytes)
const initialConfig = Client.registerValueFromBytes(Buffer.from("version: 1.0"))

// Check cost and create register
const cost = await client.registerCost(registerKey.publicKey())
console.log(`Register creation cost: ${cost} AttoTokens`)

const {cost: creationCost, addr} = await client.registerCreate(
  registerKey, initialConfig, paymentOption
)
console.log(`Register created at: ${addr.toHex()}`)

// Wait for network replication
await new Promise(resolve => setTimeout(resolve, 5000))

// Read current value
const currentValue = await client.registerGet(addr)
console.log(`Current config: ${currentValue}`)

// Update the register
const updatedConfig = Client.registerValueFromBytes(Buffer.from("version: 1.1"))
const updateCost = await client.registerUpdate(registerKey, updatedConfig, paymentOption)
console.log(`Update cost: ${updateCost} AttoTokens`)

// Wait for replication
await new Promise(resolve => setTimeout(resolve, 5000))

// Get complete history
const history = client.registerHistory(addr)
const allVersions = await history.collect()
console.log(`Config history (${allVersions.length} versions):`)
allVersions.forEach((version, i) => {
  console.log(`  ${i}: ${version}`)
})
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

### Register API Methods:

* `client.registerCost(publicKey)` - Calculate creation cost
* `client.registerCreate(key, content, payment)` - Create new register
* `client.registerGet(address)` - Get current register value
* `client.registerUpdate(key, content, payment)` - Update register content  
* `client.registerHistory(address)` - Get version history iterator
* `Client.registerKeyFromName(mainKey, name)` - Generate deterministic key
* `Client.registerValueFromBytes(buffer)` - Create RegisterValue from Buffer
