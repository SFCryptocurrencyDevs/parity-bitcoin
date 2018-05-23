# Chain

In this crate, you will find the structures and functions that make up the blockchain, Bitcoin's core data structure.

Here we will dive deep into how the blockchain is created, organized, etc. as a preface for understanding the code in this crate.

So what is a blockchain? A blockchain is a *chain* of *blocks*...

![mind blown gif](https://media.giphy.com/media/OK27wINdQS5YQ/giphy.gif)

Yep actually. The real question is, what is a [block](https://github.com/bitcoinbook/bitcoinbook/blob/develop/ch09.asciidoc#structure-of-a-block)?

A block is a data structure with two fields:
* **Block header:** a data structure containing the block's metadata
* **Transactions:** an array ([vector](https://doc.rust-lang.org/book/second-edition/ch08-01-vectors.html) in rust) of transactions

So what is a [block header](https://github.com/bitcoinbook/bitcoinbook/blob/develop/ch09.asciidoc#block-header)?

A block header is a data structure with the following fields:
* **Version:** indicates which set of block validation rules to follow
* **Previous Header Hash:** a reference to the parent/previous block in the blockchain
* **Merkle Root Hash:** a hash (root hash) of the merkle root data structure containing this block's transactions
* **Time:** a timestamp (seconds from Unix Epoch)
* **Bits:** aka the difficulty target for this block
* **Nonce:** value used in proof-of-work

*How are blocks chained together?* They are chained together via the backwards reference (Previous Header Hash) present in the block header. Each block points backwards to its parent, all the way back to the [genesis block](https://github.com/bitcoinbook/bitcoinbook/blob/develop/ch09.asciidoc#the-genesis-block) (the first block in the Bitcoin blockchain that is hard coded into all clients).

*What is a Merkle Root?* A merkle root is the root of a merkle tree. As best stated in *Mastering Bitcoin*:

> A _merkle tree_, also known as a _binary hash tree_, is a data
> structure used for efficiently summarizing and verifying the integrity of large sets of data.

In a merkle tree, all the data, in this case transactions, are leaves in the tree. Each of these is hashed and concatenated with its sibling... all the way up the tree until you are left with a single *root* hash (the merkle root hash). 

## Dependencies
#### 1. [rustc-serialize](https://crates.io/crates/rustc-serialize): Serialization and deserialization support provided by the compiler.

**One thing to note**: *This crate is deprecated in favor of [`serde`](https://serde.rs/). No new feature development will happen in this crate, although bug fixes proposed through PRs will still be merged. It is very highly recommended by the Rust Library Team that you use [`serde`](https://serde.rs/), not this crate.*

#### 2. [heapsize](https://crates.io/crates/heapsize): infrastructure for measuring the total runtime size of an object on the heap

#### 3. Crates from within this project:
	* bitcrypto (crypto)
	* primitives
	* serialization
	* serialization_derive

## Structure
**block.rs:** contains the block type. Contains some accessors to get various characteristics of a block.
**block_header.rs:** contains the block header type. Simple and straight forward, does not have many methods.
**constants.rs:** contains various constants, specifically those relating to sequence, locktime, and the # of Satoshis in one bitcoin. 
**indexed_block.rs:**
**indexed_header.rs:**
**indexed_transaction.rs:**
**lib.rs:**
**merkle_root.rs:**
**read_and_hash.rs:**
**transaction.rs:**

## Content
