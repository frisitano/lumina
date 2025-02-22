# Lumina node

A crate to configure, run and interact with Celestia's data availability nodes.

```rust,no_run
use libp2p::{identity, multiaddr::Protocol, Multiaddr};
use lumina_node::network::{
    canonical_network_bootnodes, network_genesis, network_id, Network,
};
use lumina_node::node::{Node, NodeConfig};
use lumina_node::store::SledStore;

#[tokio::main]
async fn main() {
    let p2p_local_keypair = identity::Keypair::generate_ed25519();
    let network = Network::Mainnet;
    let network_id = network_id(network).to_owned();
    let genesis_hash = network_genesis(network);
    let p2p_bootnodes = canonical_network_bootnodes(network).collect();

    let store = SledStore::new(network_id.clone())
        .await
        .expect("Failed to create a store");

    let node = Node::new(NodeConfig {
        network_id,
        genesis_hash,
        p2p_local_keypair,
        p2p_bootnodes,
        p2p_listen_on: vec!["/ip4/0.0.0.0/tcp/0".parse().unwrap()],
        store,
    })
    .await
    .expect("Failed to start node");

    node.wait_connected().await.expect("Failed to connect");

    let header = node
        .request_header_by_height(15)
        .await
        .expect("Height not found");
}
```
