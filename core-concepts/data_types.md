# Data Types

This page provides detailed information about the core data types used in the Autonomi Client API.

## Native Data Types

The Autonomi Network supports four fundamental data types, known as native data types. These form the building blocks for all data storage within the network.

On top of these native types, higher-level abstractions can be constructed. The Autonomi API, for example, provides features such as arbitrary-length byte storage, files, vaults, and registers—each built upon these core data types.

Below is a detailed description of the native data types.

### Chunk

A Chunk is a 1MB block of raw bytes that serves as a fundamental unit of storage in the Autonomi Network. It is content-addressed, meaning its address is derived from its content hash. Chunks are immutable, ensuring that once stored, their data cannot be modified. They are also self-verifiable, as their integrity can be confirmed by computing the hash and comparing it to the address.

More on Chunks in the [Chunk API Reference](../api-reference/autonomi-client/chunks.md)

### Pointer

A Pointer is a mutable reference to any native data type on the Autonomi Network. It stores an address that identifies the target data. Unlike immutable data, a Pointer follows a pay-once model but can be updated for free. To ensure consistency, it includes a version counter, preventing outdated writes. Pointers are owned by a public key, with their data stored at the owner’s address. Each update is signed by the owner, allowing verification through cryptographic signature checks.

More on Pointers in the [Pointer API Reference](../api-reference/autonomi-client/pointer.md)

### Scratchpad

A Scratchpad is a 4MB mutable storage space on the Autonomi Network. It follows a pay-once model, allowing unlimited updates for free. To maintain consistency, it includes a version counter, ensuring the latest version is always the one kept on the Network. Scratchpads are owned by a public key, with their data stored at the owner’s address. Each update is signed by the owner, making it cryptographically verifiable.

More on Scratchpads in the [Scratchpad API Reference](../api-reference/autonomi-client/scratchpad.md)

### GraphEntry

A GraphEntry is a fundamental unit in a generic directed graph within the Autonomi Network, built from multiple interconnected entries. It is immutable and can store 32 bytes of data per entry. Each GraphEntry can reference parent entries using their public keys and descendant entries along with 32 bytes of metadata per descendant, also identified by their public keys. The entry is owned by a public key, stored at the owner’s address, and signed by the owner, ensuring cryptographic verification.

More on GraphEntries in the [GraphEntry API Reference](../api-reference/autonomi-client/graphentry.md)

## High-Level Data Types

The API provides high-level data APIs and types that offer a more intuitive and accessible abstraction over the native data types. These higher-level constructs simplify usage while leveraging the native data types as their foundation. Below is a description of these data types.

### Regular Data Storage

Regular data storage allows storing arbitrary-length data securely on the Autonomi Network. It uses self-encryption, a strong encryption mechanism that ensures data is fragmented into encrypted Chunks and distributed across the network. This storage supports both public data (accessible to anyone) and private data (restricted to the owner). It provides immutable, permanent storage, where data remains unchanged once uploaded. Users pay once for the upload, and the data is preserved forever.

### Files

Files are built on top of regular data storage, allowing users to store entire files or directories as immutable archives. They support deeply nested paths and associated metadata, enabling structured file organization. Like regular data storage, files are immutable and permanent, requiring a one-time payment for upload, after which they remain unchanged forever.

### Register

A Register is a 32-byte mutable memory register that allows storing and updating small pieces of data. Unlike immutable storage, Registers support updates, but each modification requires a payment. The entire update history is preserved, enabling users to traverse past versions when needed.

### Vault

A Vault is a secure, encrypted storage system that helps users manage their data on the Autonomi Network. It allows storing owned Registers, Register keys, and references to file archives (both public and private). Users pay once for a fixed storage size and can update their Vault unlimited times for free. If more space is needed, Vault size can be upgraded at any time with an additional payment. With a Vault, users only need to retain a single key to access all their stored data on the network, simplifying data management and security.
