# Data Storage

The Autonomi Network is a decentralized data storage network that allows users to store and retrieve data securely and in an end to end encrypted way. 

There are mutliple ways to store data on the Autonomi Network, each with their own use cases and benefits. They are described in the [Network Data Types section](./data-types.md).

The section below describes the most common way to store data on the Autonomi Network. It is a way to store raw bytes data securely and privately on the Autonomi Network.

> Usage examples can be found [in the Data API Reference](../api-reference/autonomi-client/data.md)

## Regular Data Storage

This guide explains how Autonomi handles data storage, including self-encryption and the decentralized content addressed storage of data.

## Self-Encryption

Self-encryption is a fundamental mechanism that secures data storage by dividing data into chunks and encrypting each one individually.

### How It Works

1. The data is divided into fixed-size chunks.
2. Each chunk is independently encrypted.
3. A DataMap is generated to reference and track the encrypted chunks.

## Decentralized Content-Addressed Storage

After self-encryption, the encrypted chunks are stored on the Autonomi Network in a decentralized and unlinkable manner.

### How It Works

1. Encrypted chunks are produced via self-encryption.
2. Each chunk is hashed, and the resulting hash serves as its storage address.
3. Chunks are distributed across network segments responsible for specific address ranges.
4. The DataMap is essential for data retrieval, as it contains the references to all stored chunks.

## Public and Private Data

Public and private data on the Autonomi Network are distinguished by how the DataMap—the essential component for reassembling and decrypting encrypted chunks—is managed.

### How It Works

1. The DataMap is vital for reconstructing and decrypting the distributed encrypted chunks.
2. For public data, the DataMap is uploaded to the network in an unencrypted form, allowing open access to the content.
3. For private data, the DataMap is either kept locally or uploaded in an encrypted form, ensuring that only authorized users can access the underlying data.