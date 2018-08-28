# Ch.9 The Blockchain
_Mastering Bitcoin_ of O'Reilly

## Introduction

- An ordered back-linked list of blocks of TXs

- Can be stored as a flat file, or in a simple DB

- The Bitcoin Core client stores the BC metadata using Google's LevelDB

- Visualization
	- Often visualized as a vertical stack
	- "Height" refer to the distance from the first block
	- "Top" or "tip" refer to the most recently added block

- Block identified by a SHA-256 hash value

- Parent block is referred through the "previous block hash" field in the header

- A block can temporarily have multiple children, called a "fork"
	- There will be only one child become part of the BC eventually

- Previous block hash affects current hash
	- Thus, when a parent is modified, all of its descendant need to change
	- That's why deep history of a long BC is immutable
	- The key feature of bitcoin's security

## Structure of a Block

- A container data structure

- Aggregates TXs in the BC (public ledger)

- Components

Size         | Field        | Description
-------------|--------------|--------------
4 bytes      | Block Size   | Size of the block in bytes
80 bytes     | Block Header | Block header
1 to 9 bytes | TX Counter   | # of Following TXs
Variable     | TXs          | TXs recorded in this block

### Block Header

Size     | Field (in code) | Description
---------|-----------------|--------------
4 bytes  | nVersion        | Version no. to track software / protocol upgrades
32 bytes | hashPrevBlock   | Previous Block Hash: Reference to the previous block
32 bytes | hashMerkleRoot  | Hash of the root of the Merkle tree of TXs
4 bytes  | nTime           | Approx. timestamp of creation
4 bytes  | nBits           | PoW difficulty target
4 bytes  | nNonce          | Counter for PoW

- The nonce, difficulty target, and timestamp are used in the mining process.

## Block Identifiers: Block Header Hash and Block Height

- Primary identifier
	- Cryptographic hash
		- Hash function: Double SHA-256
		- Result: _Block hash_ (But actually _block header hash_)
	- Not included inside the data structure, neither during transmission, nor storing on BC
		- But computed by each receiving node
		- Might be stored in separate DB as metadata for faster retrieval

- Secondary identifier
	- Position in the BC
		- _Block height_
		- The genesis block (mentioned in the next section) is at height 0	
		- Each block is "one higher" than the previous one
		- Not a unique identifier: fork condition
	- Also not included in the data structure
		- Each node dynamically identifies a block's height upon reception
		- Might also be stored as metadata


## The Genesis Block

- The first block in the BC

- Created in 2009

- Common ancestor of all blocks

- Block hash: `000000000019d6689c085ae165831e934ff763ae46a2a6c172b3f1b60a8ce26f`

- Statically encoded within the bitcoin client
	- Every node always starts with a BC of at least one block
	- Every node has the starting point for the BC

- Hidden message
	- Embedded by _Satoshi Nakamoto_, bitcoin's creator
	- Coinbase TX input contains the text:
		`The Times 03/Jan/2009 Chancellor on brink of second bailout for banks.`
	- Intended to offer proof of the earliest date this block was created

## Linking Blocks in the BC

- Upon reception, a node will examine the block header and look for the "previous block hash"

- Blocks linked in a chain by reference to the previous block header hash

## Merkle Trees

- Known as _binary hash tree_

- Blocks contains a _Merkle tree_ of summary of TXs

- Used to summarize all TXs in a block
	- Producing an overall digital fingerprint
	- Providing a very efficient verification way

- Construction
	- Principle: bottom-up
	- Each leaf node contains a hash value of a TX
	- Each internal node contains a hash value of concatenation of hash value of its children
	- If there are odd # of TXs, the last is duplicated and put aside "itself"

- No matter how many TXs in the block, a Merkle root "summarizes" them with 32 bytes of data

### Existence Proof of a TX

- To find a specific TX in a block need approx. log~2~(N) 32-byte hashes
- _Authentication path_ or _Merkle path_ connecting the TX to the root
	- Consists of all the other "complementing" hashes
	- Or, all the other hashes needed to construct the hash of Merkle root
	- Example as below

![](https://raw.githubusercontent.com/bitcoinbook/bitcoinbook/develop/images/mbc2_0905.png)

- Noticeably, the log-ed time scale saves much time for the proof

- By retrieving a Merkle path, a node can identify a TX in a block

- Merlke paths help SPV nodes verifying TXs without downloas full blocks

### Merkle Trees and Simpleified Payment Verification (SPV)

## Bitcoin's Test BC

- There are 3 other BC of Bitcoin for testing purposes
	- _testnet_
	- _segnet_
	- _regtest_

### Test - Bitcoin's Testing Playground

- __Testnet__ is the name of the BC, network, and currency for that testing purposes

-  A fully featured live P2P network, with wallet, testnet coins, mining

- Full features of mainnet

- Differences from mainnet
	- Testnet coin are worthless
	- Mining difficulty is much lower

- Bitcoin software development should be tested on testnet before launch
	- To protect developer from monetary losses and unintended behaviors

- Some people use advanced equipment to mine testnet
	- Increases difficulty
	- Testnet is impossible to mine with CPU over time
	- Resulting testnet to restart sometimes with a new genesis block
	- Currently (2017), it's _testnet3_, the third iteration

- Testnet also support segwit (Segregated Witness)

### Segnet - The Segregated Witness Testnet

- A special-purpose testnet for segwit

- Running a special version (branch) of Bitcoin ore

- No longer necessary due to the support of segwit on testnet

### Regetst - The Local BC

- Stands for "Regression Testing"

- Allow creation of a local BC for testing

- Intended to run as closed systems for local testing

### Using Test BCs for Development

- Establishment of a development pipeline

- Test on regtest when development

- Try on public network when ready
	- A dynamic environment with more diversity

- Switch to mainnet to deploy as a product
