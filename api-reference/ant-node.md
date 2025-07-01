# Node API

The **antnode** crate provides an API for programmatically running a node.

## Installation

{% tabs %}
{% tab title="Rust" %}
```bash
cargo add ant-node
```
{% endtab %}

{% tab title="Python" %}
```bash
# Install using uv (recommended)
curl -LsSf <https://astral.sh/uv/install.sh> | sh
uv pip install maturin
uv pip install antnode

# Or using pip
pip install antnode.
```
{% endtab %}

{% tab title="Node.js" %}
```console
npm install @withautonomi/ant-node
```
{% endtab %}
{% endtabs %}

## Start a Single Node

use `NodeSpawner` to programmatically spawn a single node and connect it to the live network.

{% tabs %}
{% tab title="Rust" %}
```rust
use ant_node::spawn::node_spawner::NodeSpawner;
use autonomi::Multiaddr;
use std::str::FromStr;

#[tokio::main]
async fn main() {
    // Using the genesis node as a bootstrap node for example
    let bootstrap_peers = vec![
        Multiaddr::from_str("/ip4/209.97.181.193/udp/57402/quic-v1/p2p/12D3KooWHygG9a7inESky2KpvHQmbX5o2UC8D29B5njdshAcv1p6")
            .expect("Invalid multiaddr")
    ];

    let running_node = NodeSpawner::new()
        .with_initial_peers(bootstrap_peers)
        .spawn()
        .await
        .expect("Failed to spawn node");

    let listen_addrs = running_node
        .get_listen_addrs_with_peer_id()
        .await
        .expect("Failed to get listen addrs with peer id");

    println!("Node started with listen addrs: {listen_addrs:?}");
}
```
{% endtab %}

{% tab title="Node.js" %}
```ts
import { NodeSpawner } from '@withautonomi/ant-node'

const bootstrapPeers = ['/ip4/209.97.181.193/udp/57402/quic-v1/p2p/12D3KooWHygG9a7inESky2KpvHQmbX5o2UC8D29B5njdshAcv1p6'];
const spawner = new NodeSpawner({
    // Using the genesis node as a bootstrap node for example
    initialPeers: bootstrapPeers,
});
const runningNode = await spawner.spawn();
const listenAddrs = await runningNode.getListenAddrsWithPeerId();

console.log(`Node started with listen addrs: ${listenAddrs}`);
```
{% endtab %}
{% endtabs %}

## Start a Network

Use `NetworkSpawner` to programmatically start a node network (very useful for automated testing).

{% tabs %}
{% tab title="Rust" %}
```rust
use ant_node::spawn::network_spawner::NetworkSpawner;
use std::time::Duration;
use tokio::time::sleep;

#[tokio::main]
async fn main() {
    let network_size = 20;

    let running_network = NetworkSpawner::new()
        .with_evm_network(Default::default())
        .with_local(true)
        .with_size(network_size)
        .spawn()
        .await
        .expect("Failed to spawn network");

    // Wait for nodes to dial each other
    sleep(Duration::from_secs(10)).await;

    for node in running_network.running_nodes() {
        println!("Node listening on: {:?}", node.get_listen_addrs().await);
    }
}
```
{% endtab %}

{% tab title="Node.js" %}
```ts
import { NetworkSpawner, Network } from '@withautonomi/ant-node';
import { setTimeout } from 'node:timers/promises';

const networkSize = 3;
const spawner = new NetworkSpawner({
    local: true,
    size: networkSize,
}, Network.fromString('evm-arbitrum-one'));
const runningNetwork = await spawner.spawn();
const runningNodes = await runningNetwork.runningNodes();

// Wait for nodes to dial each other
await setTimeout(10 * 1000);

for (const node of runningNodes) {
    console.log(`Node listening on: ${await node.getListenAddrs()}`);
}

await runningNetwork.shutdown();
```
{% endtab %}
{% endtabs %}
