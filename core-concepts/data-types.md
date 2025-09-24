# Data Types

This page provides detailed information about the core data types used in the Autonomi Client API.

## Native Data Types

The Autonomi Network supports four fundamental data types, known as native data types. These form the building blocks for all data storage within the network.

On top of these native types, higher-level abstractions can be constructed. The Autonomi API, for example, provides features such as arbitrary-length byte storage, files, vaults, and registers—each built upon these core data types.

Below is a detailed description of the native data types.

### Chunk

A Chunk is a 4MB block of raw bytes that serves as a fundamental unit of storage in the Autonomi Network. It is content-addressed, meaning its address is derived from its content hash. Chunks are immutable, ensuring that once stored, their data cannot be modified. They are also self-verifiable, as their integrity can be confirmed by computing the hash and comparing it to the address.

More on Chunks in the [Chunk API Reference](../api-reference/autonomi-client/chunks.md)

### Pointer

A Pointer is a mutable reference to any native data type on the Autonomi Network. It provides an indirection mechanism for referencing data stored elsewhere on the network.

More on Pointers in the [Pointer API Reference](../api-reference/autonomi-client/pointer.md)

### Scratchpad

A Scratchpad is a 4MB mutable storage space on the Autonomi Network. It follows a pay-once model, allowing unlimited updates for free. To maintain consistency, it includes a version counter, ensuring the latest version is always the one kept on the Network. Scratchpads are owned by a public key, with their data stored at the owner’s address. Each update is signed by the owner, making it cryptographically verifiable.

More on Scratchpads in the [Scratchpad API Reference](../api-reference/autonomi-client/scratchpad.md)

### GraphEntry

A GraphEntry is a fundamental unit in a generic directed graph within the Autonomi Network, built from multiple interconnected entries. It is immutable and can store 32 bytes of data per entry. Each GraphEntry can reference parent entries using their public keys and descendant entries along with 32 bytes of metadata per descendant, also identified by their public keys. The entry is owned by a public key, stored at the owner’s address, and signed by the owner, ensuring cryptographic verification.

More on GraphEntries in the [GraphEntry API Reference](../api-reference/autonomi-client/graphentry.md)

## High-Level Data Types

The API provides high-level data APIs and types that offer a more intuitive and accessible abstraction over the native data types. These higher-level constructs simplify usage while leveraging the native data types as their foundation. Below is a description of these data types.

### Public Archive

A Public Archive is a structure that maps file paths to data addresses on the network. While it can be used to store a single file with its metadata, it is generally used to organize multiple files in a hierarchical structure to simulate directories. Public Archives support nested paths and store metadata (creation time, modification time, size) for each file. Public Archives are stored on the network as Chunks.

**How it works:**

Think of a Public Archive as a directory listing that lives on the network. When you create one, each file gets uploaded as chunks first, then the archive creates a map linking file paths to where those chunks live on the network. The archive itself becomes a chunk too—it's like taking a photo of your file system structure and storing that photo on the network. Once uploaded, this snapshot is permanent and immutable, but it provides a convenient way to organize and reference collections of files as a single unit.

More on Public Archives in the [Public Archive API Reference](../api-reference/autonomi-client/public-archive.md)

### Private Archive

A Private Archive provides enhanced privacy by keeping data maps within the archive itself rather than referencing them on the network. Unlike Public Archives which reference files through their network addresses, Private Archives contain the complete data maps within the archive structure. Like Public Archives, they support nested paths to simulate directories and store metadata for each file. Private Archives are uploaded to the network as encrypted chunks.

**How it works:**

A Private Archive takes a different approach to privacy than its public counterpart. Instead of just storing references to where files live on the network, it actually embeds the complete data maps for each file within the archive itself. This means the archive becomes more like a self-contained package—you still upload your files as chunks first, but then the archive stores the detailed instructions for reassembling those chunks internally rather than just pointing to them. The entire archive gets encrypted before being stored as chunks, ensuring that even the file organization and metadata remain private. This gives you both the convenience of organized file collections and enhanced privacy protection.

More on Private Archives in the [Private Archive API Reference](../api-reference/autonomi-client/private-archive.md)

### Register

A Register is a 32-byte mutable memory register that allows storing and updating small pieces of data. Unlike immutable storage, Registers support updates, but each modification requires a payment. The entire update history is preserved, enabling users to traverse past versions when needed.

**How it works:**

Registers cleverly combine two native data types to create something that feels mutable in an immutable world. Each time you update a register, it creates a new GraphEntry containing your 32 bytes of data and links it to the previous version, building a chain of history. Meanwhile, a Pointer acts like a bookmark that always points to the latest entry in this chain. When you want the current value, you follow the pointer to the newest GraphEntry. When you want history, you can walk backward through the chain of linked entries. It's like having both the latest page of a notebook and the ability to flip through all the previous pages whenever you need them.

More on Registers in the [Register API Reference](../api-reference/autonomi-client/register.md)

### Vault

A Vault is a secure, encrypted storage system that helps users manage their data on the Autonomi Network. It allows storing owned Registers, Register keys, and references to file archives (both public and private). Users pay once for a fixed storage size and can update their Vault unlimited times for free. If more space is needed, Vault size can be upgraded at any time with an additional payment. With a Vault, users only need to retain a single key to access all their stored data on the network, simplifying data management and security.

**How it works:**

A Vault is like a sophisticated filing cabinet that grows as you need more space. It starts with a single GraphEntry that acts as your main filing drawer—this can point to up to 1,000 storage compartments called Scratchpads, each holding about 4MB of your encrypted data. One special slot in each GraphEntry is reserved as a "next drawer" pointer, so when you fill up those 1,000 compartments, the vault automatically creates a new GraphEntry and links it to the previous one, giving you another 1,000 compartments to fill.

The clever part is how vaults repurpose GraphEntry descendants—instead of linking to other GraphEntries (which is their typical use), vault descendants point directly to Scratchpads for data storage, making the vault structure both efficient and expandable.

Your vault's location is mathematically derived from your secret key, so you always know where to find it. All your data gets encrypted before being stored in the Scratchpads, ensuring privacy. The beauty is that once you've paid for your initial vault space, you can reorganize and update your data as much as you want—only expanding to new compartments requires additional payment. It's designed so that with just one secret key, you can access and manage all your data scattered across the network.

More on Vaults in the [Vault API Reference](../api-reference/autonomi-client/vault.md)
