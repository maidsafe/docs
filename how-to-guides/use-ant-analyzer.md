# Tutorial Ant Analyzer
In this tutorial we will see how to use Ant Analyzer through some simple examples.

### Description
Ant Analyzer is a versatile tool to read and decode addresses on Autonomi Network following its protocol.
It can read binaries and addresses from local Private Archive & Datamap as well.

### Prerequisites
You will need the Autonomi toolchain installed: [Autonomi intallation](https://github.com/maidsafe/docs/blob/main/getting-started/installation.md).
You will need a local network to run: [local network setup](https://github.com/maidsafe/docs/blob/main/how-to-guides/local-network.md).

## Let's start!
We will use Ant Analyzer on a local network, the command will be:

```bash
cargo run --bin ant -- --local analyze -v <ADDR>
```
**Remark:**

 - '-v' option  (verbose ON)  is used to show the whole process of the Analyzer step by step.
 - To use the Analyzer on the real network, remove the option '-\-local'.

___

**Initial condition**: Having a folder with 3 files inside, in my case it will be:
```
test_folder_analyzer/
 |- analyze.rs
 |- config.rs
 |- mod.rs
```
The files above are from Autonomi repo [here](https://github.com/maidsafe/autonomi).

___

**Let's upload the folder:**
```bash
cargo run --bin ant -- --local file upload -p test_folder_analyzer
```
'-p' option is used to upload the folder publicly.

**Result should be:**
```bash
🔗 Connected to the Network
Uploading data to network...
Encrypting file: "test_folder_analyzer/config.rs"..
Encrypting file: "test_folder_analyzer/mod.rs"..
Encrypting file: "test_folder_analyzer/analyze.rs"..
Successfully encrypted file: "test_folder_analyzer/config.rs"
Successfully encrypted file: "test_folder_analyzer/mod.rs"
Successfully encrypted file: "test_folder_analyzer/analyze.rs"
Paying for 12 chunks..
Uploading file: test_folder_analyzer/config.rs (4 chunks)..
Uploading file: test_folder_analyzer/mod.rs (4 chunks)..
Uploading file: test_folder_analyzer/analyze.rs (4 chunks)..

[...]

Successfully uploaded: test_folder_analyzer
At address: c39ec8deb870fcc87625e501c369ba6c96da8088925dbb8ef7e6fdbc5ed44b02
Number of chunks uploaded: 16
Number of chunks already paid/uploaded: 0
Total cost: 48 AttoTokens
```

We will analyze the different addresses this way: **Public Archive -> Datamap of a file -> Chunk**.

___

**The last address from the previous command is the Public Archive address, let's analyze it!**

```bash
cargo run --bin ant -- --local analyze -v c39ec8deb870fcc87625e501c369ba6c96da8088925dbb8ef7e6fdbc5ed44b02
```

**Result should be:**
```bash
Analyzing address: c39ec8deb870fcc87625e501c369ba6c96da8088925dbb8ef7e6fdbc5ed44b02
🔗 Connected to the Network
Identified as a Chunk address...
Getting chunk at address: c39ec8deb870fcc87625e501c369ba6c96da8088925dbb8ef7e6fdbc5ed44b02 ...
Got chunk of 318 bytes...
Identified chunk content as a DataMap...
Identified a DataMap which directly contains data...
Fetching data from the Network...
Data fetched from the Network...
All addresses are xornames, so it's a public archive
Identified the data pointed to by the DataMap as a PublicArchive...
Analysis successful
PublicArchive stored at: c39ec8deb870fcc87625e501c369ba6c96da8088925dbb8ef7e6fdbc5ed44b02
PublicArchive with 3 files
PublicArchive {
    map: {
        "test_folder_analyzer/analyze.rs": (
            e00f3c4f81a4041266b63d6875fc33d450f36c9dd092c650394c824ec540eb8a,
            Metadata {
                created: 1742284233,
                modified: 1742284233,
                size: 18361,
                extra: None,
            },
        ),
        "test_folder_analyzer/config.rs": (
            25cdd6e9cf72985a4724e0abbc02d3af751ee27e16081ae028ffcaa50acc4848,
            Metadata {
                created: 1742284233,
                modified: 1742284233,
                size: 8295,
                extra: None,
            },
        ),
        "test_folder_analyzer/mod.rs": (
            281b5c8d3b0af2d24c076e36abad36939b1ff7b72ef4ea420954dd032857e547,
            Metadata {
                created: 1742284233,
                modified: 1742284233,
                size: 13593,
                extra: None,
            },
        ),
    },
}
```
**What can we read from this result?**
The Ant Analyzer revealed:
- The address contains: 1 Public Archive.
- The Public Archive contains: 3 Datamap addresses + metadata from 3 files.

___

**Let's analyze the Datamap address of "test_folder_analyzer/analyze.rs"**
```bash
cargo run --bin ant -- --local analyze -v e00f3c4f81a4041266b63d6875fc33d450f36c9dd092c650394c824ec540eb8a
```

**Result should be:**
```bash
Analyzing address: e00f3c4f81a4041266b63d6875fc33d450f36c9dd092c650394c824ec540eb8a
🔗 Connected to the Network
Identified as a Chunk address...
Getting chunk at address: e00f3c4f81a4041266b63d6875fc33d450f36c9dd092c650394c824ec540eb8a ...
Got chunk of 326 bytes...
Identified chunk content as a DataMap...
Identified a DataMap which directly contains data...
Fetching data from the Network...
Data fetched from the Network...
Analysis successful
DataMap stored at: e00f3c4f81a4041266b63d6875fc33d450f36c9dd092c650394c824ec540eb8a
DataMap containing 3 Chunks
Content is another DataMap: false
[
    e022fb63a0657fd0f4db0dad8adf31dd48e94ceca5db77489032247966f749f4,
    d3f0c963da171186e9628a2df81b13bb4f579739ad96ddf97e8a4e6d72474979,
    6d8bfd825f282c71d226baba60fa5705035d3a39d5a73eb481ba9cd8727c1aab,
]
Decrypted Data in hex: [18361 bytes of data]
```
**What can we read from this result?**
The Ant Analyzer revealed:
- The address contains: 1 Datamap.
- The Datamap contains: 3 Chunks addresses.
- The Chunks addresses contain: 3/3 parts of 1 file.
- The decrypted file has a size of 18 KB.

**Remark**
- Ant Analyzer will show the decrypted data in HEX if < 4 KB.
- 4 KB is an arbitrary value just to avoid saturating the terminal.
- If you need to see the data, download them with ```cargo run --bin ant -- --local file download <ADDR> <DEST>```
- ADDR is the address of the Datamap, DEST is the destination (file or folder).
- "Content is another DataMap: false" from previous result means the Datamap is not a Datamap of a Datamap, this can occur if a file is too big for a Datamap to map it entirely.

**Let's analyze a Chunk address of "test_folder_analyzer/analyze.rs"**
```bash
cargo run --bin ant -- --local analyze -v e022fb63a0657fd0f4db0dad8adf31dd48e94ceca5db77489032247966f749f4
```
**Result should be:**
```bash
Analyzing address: e022fb63a0657fd0f4db0dad8adf31dd48e94ceca5db77489032247966f749f4
🔗 Connected to the Network
Identified as a Chunk address...
Getting chunk at address: e022fb63a0657fd0f4db0dad8adf31dd48e94ceca5db77489032247966f749f4 ...
Got chunk of 1632 bytes...
Analysis successful
Chunk stored at: e022fb63a0657fd0f4db0dad8adf31dd48e94ceca5db77489032247966f749f4
Chunk content in hex: 7df3b0f2423b5c481426a3cac7a730a7aab81af24d089e3bc9a29b24726eba1522c4b84aeeeb1e[...]
```

**What can we read from this result?**
The Ant Analyzer revealed:
- The address contains: 1 Chunk.
- The Chunk has a size of 1632 B.
- The encrypted content is shown (< 4 KB).

___

**That's it! Now you know how to use Ant Analyzer!**


**To go further:**
You can test it out with private files, as the Datamap and Archive won't be uploaded on the Network you will need to analyze the local private addresses from Private Archive, or directly the Private Datamap encoded in HEX
You will find theses files in:
**macOS:**
```$HOME/Library/Application\ Support/autonomi/client/user_data/private_file_archives```
**Linux:**
```$HOME/.local/share/autonomi/client/user_data/private_file_archives```