---
id: what-are-merkle-feeds
title: What are Merkle Feeds
sidebar_position: 1
---

---

Built in collaboration with [DOPP](https://www.dopp.finance/). Merkle feeds are the most efficient way to get complex financial instruments updated on-demand on Starknet.

The Pragma Consumer SDK is a powerful tool that allows developers to interact with Pragma's Merkle Feed system.
This SDK enables you to fetch option prices and their associated Merkle proofs, which you can then use with the Pragma Oracle contract to access on-chain data.

## A Merkle Feed?

A Merkle Feed is an efficient way to publish and verify large amounts of data on-chain. In Pragma's case, we publish a Merkle root on-chain that represents a tree of option prices for a specific blockchain height. Users can then use our SDK to retrieve prices off-chain and verify their correctness on-chain, creating a secure and efficient "pull oracle" model.

## How It Works

  <div>
  <img width="100%" height="100%" src="/img/flowchart/Flowchart-1.webp" />
  </div>

1. The merkle trees are built with data from [Deribit](https://www.deribit.com/) the leading platform for options trading.
2. Merkle root is published on-chain _every_ block.
3. Merkle proofs with associated data can be retrieved through REST and WebSocket endpoints or through a rust crate we provide.
4. Data is updated on-demand on-chain upon successful verification of the merkle proof.

## Contract Address

We use the same contract as the one used for Realized Volatilty or TWAP.
You can find it at these addresses:

- Sepolia: [0x0379afb83d2f8e38ab08252750233665a812a24278aacdde52475618edbf879c](https://sepolia.voyager.online/contract/0x0379afb83d2f8e38ab08252750233665a812a24278aacdde52475618edbf879c#readContract)
- Mainnet: 🔜
