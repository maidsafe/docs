# BLS Keys

The Autonomi Client API makes heavy use of BLS keys. They are used for signing and verifying authenticity of various data types, as well as for key-addressing (data types that derive their address from a public key).

It is assumed that each user has a keypair, and that the keypair is stored in a secure manner.

### What they are

BLS keys are part of the Boneh–Lynn–Shacham signature scheme, offering an efficient and secure method for digital signatures. A private key is used to sign data, while its corresponding public key is distributed to enable signature verification.

### What this means for the rest of us

BLS keys are like a digital signature kit: the secret key is your personal stamp that you keep hidden, while the public key is the version you share so others can verify your messages. In simple terms, when you sign a message with your secret key, anyone with your public key can confirm it’s really from you. The underlying math is complex, but all you need to know is that BLS keys make sure your digital signature is secure and unforgeable.

### Example of basic key usage

```rust
use autonomi::{SecretKey, PublicKey, Signature};

// create a random secret key
let secret_key = SecretKey::random();

// derive the public key from the secret key
let public_key: PublicKey = secret_key.public_key();

// sign a message
let message = "Hello, world!";
let signature: Signature = secret_key.sign(message.as_bytes());

// verify the signature
let verified = public_key.verify(message.as_bytes(), signature);
assert!(verified);
```

### Key Derivation

Many data types on the Autonomi Network are key-addressed, meaning they derive their address from a public key. This also means that each key can only be used for one data item. Since it would be tedious to keep track of all the keys needed for the various data types the Client API supports, each user has a main secret key, which is used to derive all other keys needed.

### Example of key derivation

This example demonstrates how to derive unique keys from a main key and how to perform hex conversion, serialization, and signature verification. The Autonomi Client API uses a main keypair from which all other keys are derived. This ensures that each data item can be addressed using a unique public key derived from the main key, without the need to manage numerous independent keypairs.

```rust
use autonomi::client::key_derivation::{MainSecretKey, MainPubkey, DerivedPubkey, DerivationIndex};
use thread_rng;
use rmp_serde;

// Generate a main secret key and derive its corresponding public key.
let main_sk = MainSecretKey::random();
let main_pubkey = main_sk.public_key();

// Derive a unique (child) public key using a random derivation index.
let derivation_index = DerivationIndex::random(&mut thread_rng());
let derived_pubkey = main_pubkey.derive_key(&derivation_index);

// Those derived keys can be turned into regular BLS keys for use with other data types APIs
let _regular_public_key: PublicKey = derived_sk.public_key().into();
let _regular_secret_key: SecretKey = derived_sk.into();
```
