# Chain

In this crate, you will find the structures and functions that make up the blockchain, Bitcoin's core data structure.

## Conceptual Overview
Here we will dive deep into how the blockchain is created, organized, etc. as a preface for understanding the code in this crate.

Here we will cover the following concepts:
* blockchain
* block
* block header
* merkle tree
* transaction
* witnesses and SegWit
* indexed vs. non-indexed

### Blockchain
So what is a blockchain? A blockchain is a *chain* of *blocks*...

![mind blown gif](https://media.giphy.com/media/OK27wINdQS5YQ/giphy.gif)

### Block
Yep actually. The real question is, what is a [block](https://github.com/bitcoinbook/bitcoinbook/blob/develop/ch09.asciidoc#structure-of-a-block)?

A block is a data structure with two fields:
* **Block header:** a data structure containing the block's metadata
* **Transactions:** an array ([vector](https://doc.rust-lang.org/book/second-edition/ch08-01-vectors.html) in rust) of transactions

![Blockchain diagram](https://raw.githubusercontent.com/pluralsight/guides/master/images/8cd8b94f-d05f-41e8-a0f1-70853f390094.png)

### Block Header
So what is a [block header](https://github.com/bitcoinbook/bitcoinbook/blob/develop/ch09.asciidoc#block-header)?

A block header is a data structure with the following fields:
* **Version:** indicates which set of block validation rules to follow
* **Previous Header Hash:** a reference to the parent/previous block in the blockchain
* **Merkle Root Hash:** a hash (root hash) of the merkle root data structure containing this block's transactions
* **Time:** a timestamp (seconds from Unix Epoch)
* **Bits:** aka the difficulty target for this block
* **Nonce:** value used in proof-of-work

![Block header diagram](https://i.stack.imgur.com/BiaJK.png)

*How are blocks chained together?* They are chained together via the backwards reference (Previous Header Hash) present in the block header. Each block points backwards to its parent, all the way back to the [genesis block](https://github.com/bitcoinbook/bitcoinbook/blob/develop/ch09.asciidoc#the-genesis-block) (the first block in the Bitcoin blockchain that is hard coded into all clients).

### Merkle Root
*What is a Merkle Root?* A merkle root is the root of a merkle tree. As best stated in *Mastering Bitcoin*:

> A _merkle tree_, also known as a _binary hash tree_, is a data
> structure used for efficiently summarizing and verifying the integrity of large sets of data.

In a merkle tree, all the data, in this case transactions, are leaves in the tree. Each of these is hashed and concatenated with its sibling... all the way up the tree until you are left with a single *root* hash (the merkle root hash). 

![Merkle tree](https://upload.wikimedia.org/wikipedia/commons/9/95/Hash_Tree.svg)



### Transaction
According to [Mastering Bitcoin](https://github.com/bitcoinbook/bitcoinbook/) :

> Transactions are the most important part of the bitcoin system. Everything else in bitcoin is designed to ensure that transactions can  created, propagated on the network, validated, and finally added to the global ledger of transactions (the blockchain).

At its most basic level, a transaction is an encoded data structure that facilitates the transfer of value between two public key addresses on the Bitcoin blockchain.

The most fundamental building block of a transaction is a `transaction output` -- the bitcoin you own in your "wallet" is in fact a subset of `unspent transaction outputs` or `UTXO's` of the global `UTXO set`. `UTXOs` are indivisible, discrete units of value which can only be consumed in their entirety. Thus, if I want to send you 1 BTC and I only own one `UTXO` worth 2 BTC, I would construct a transaction that spends my `UTXO` and sends 1 BTC to you and 1 BTC back to me (just like receiving change).

**Transaction Output:** transaction outputs have two fields:
* *value*: the value of a transaction
* *scriptPubKey (aka locking script or witness script)*: conditions required to unlock (spend) a transaction value

**Transaction Input:** transaction inputs have four fields:
* *previous output*: the previous output transaction reference, as an OutPoint structure (see below)
* *scriptSig*: a script satisfying the conditions set on the UTXO
* *scriptWitness*:
* *sequence number*: transaction version as defined by the sender. Intended for "replacement" of transactions when information is updated before inclusion into a block.

**Outpoint**: 
* *hash*: references the transaction that contains the UTXO being spent
* *index*: identifies which UTXO from that transaction is referenced

**Transaction Version:** the version of the data formatting

**Transaction Locktime:** this specifies either a block number or a unix time at which this transaction is valid

**Transaction Fee:** A transaction's input value must equal the transaction's output value or else the transaction is invalid. The difference between these two values is the transaction fee, a fee paid to the miner who includes this transaction in his/her block.

### Witnesses and SegWit

Regular transaction id:
`  [nVersion][txins][txouts][nLockTime]`
Witness transaction id:
 `[nVersion][marker][flag][txins][txouts][witness][nLockTime]`

A `witness root hash` is calculated with all those `wtxid` as leaves, in a way similar to the `hashMerkleRoot` in the block header.


### Indexed vs. Non-indexed


Need a more interactive playground? Check out [this awesome website](https://anders.com/blockchain/).

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

### Block
A relatively straight forward implementation of the data structure described above. A `block` is a rust `struct`. It implements the following traits:
* ```From<&'static  str>```: this trait takes in a string and outputs a `block`. It is implemented via the `from` function which deserializes the received string into a `block` data structure. Read more about serialization [here](https://github.com/bitcoinbook/bitcoinbook/blob/develop/ch06.asciidoc#transaction-serializationoutputs) (in the context of transactions).
* ```RepresentH256```: this trait takes a `block` data structure and hashes it, returning the hash.

The `block` has a few methods of its own. The entierty of these are simple getter methods.