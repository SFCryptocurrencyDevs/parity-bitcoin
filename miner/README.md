# Miner

In this crate, you will find the structures and functions that deal with generation of blocks by the bitcoinbook mining software.

## Conceptual Overview
We'll cover the components that make up the creation and generation of a bitcoin block.

Here we will cover the following concepts:
* Blocks (brief recap)
* Miners
* Fees
* MemPools

### Blocks
A block is a data structure with two fields:
* **Block header:** A data structure containing the block's metadata
* **Transactions:** An array ([vector](https://doc.rust-lang.org/book/second-edition/ch08-01-vectors.html) in rust) of transactions. More sauce here: [What is a Bitcoin Block](https://github.com/SFCryptocurrencyDevs/parity-bitcoin/tree/master/chain#block)


### Miners
So what is a miner? A miner is a piece of software that runs the software needed to create and validate a bitcoin block, its block header, coinbase, and transactions.

![mining gif](http://cloudmininggravity.info/allimg/362739.gif)

The miner software also contains the hashing algorithms that determine the level of difficulty in the Proof of Work.

### Fees
Fees are the payments that are included in a transaction to compensate blockchain miners for creating blocks and validating transactions in those blocks.

### MemPools
The memory pool (mempool) contains the accounting or list about all transactions That are verified, but not yet confirmed into a block. Bitcoin clients use the MemPool message function to respond to requests of the list of unspent outputs. See [mempool message BIP 037](https://github.com/bitcoin/bips/blob/master/bip-0035.mediawiki)
for more details about how Mempools provide network information services in bitcoin.

***

## Dependencies
#### 1. [byteorder](https://crates.io/crates/byteorder)
*Convenience methods for encoding and decoding numbers in either big-endian or little-endian order.*
#### 2. [heapsize](https://crates.io/crates/heapsize)
*Infrastructure for measuring the total runtime size of an object on the heap*
#### 2. [bitcrypto](../crypto)
 *Building off of the primitives crate, this crate provides a toolkit of cryptographic hash functions.*
#### 3. [chain](../chain)
 *The structures and functions that make up the blockchain, Bitcoin's core data structure.*
#### 4. [storage](..master/storage)
#### 5. [db](../db)
#### 6. [network](../network)
#### 7. [primitives](../primitives)
*The lowest level Bitcoin primitives*
#### 8. [rustc-serialize](https://crates.io/crates/rustc-serialize)
 *Serialization and deserialization support provided by the compiler.*
#### 9. [verification](../verification)
#### 10. [keys](../keys)
#### 11. [script](../script)

***

## Content

### Block Templates (block_assembler.rs)
The Block Template originates from [getblocktemplate - Fundamentals BIP 022](https://github.com/bitcoin/bips/blob/master/bip-0022.mediawiki) Pool mining in Bitcoin was a competitive approach to bitcoin mining where many computers would miner a block in a coordinate pool. Pool mining made the CPU mining approach inefficient. As such, this proposal created a JSON-RPC method for sending the entire block structure, instead of just the block header.

The block template enforces the consensus rules for a block.

The ```pub structs``` are the constraints of the blocks
* version
* previous header hashes
* time
* bits
* height
* transactions
* coinbase value
* size limit
* sigop_limit (the number of signature operations allowed)

We now have an iterator module ```struct FittingTransactionsIterator<'a, T>``` and ```impl<'a, T> FittingTransactionsIterator<'a, T>``` that iterates through each transaction to verify constraints and then stores the transaction in the block with the ```impl BlockAssembler```.

Tests are included under ```mod tests```  


### CPU Miner (cpu_miner.rs)
Creating a block requires preparation of the blockheader and calculation of the **Number Used Only Once** (nonce) which is the proof of work for the discovered hashing solution. As such, this rust file prepares the block header and calculates the nonce.

```
impl BlockHeaderBytes
```
Call the method to Serialize the Block header into memory and the required functions for block data.
```
fn set_merkle_root_hash(&mut self, hash: &H256)
```
Set the merkle root hash of the block
```
fn set_time(&mut self, time: u32)
```
Set the block header time
```
fn set_nonce(&mut self, nonce: u32)
```
Set the block header nonce for the proof of work solution
```
fn hash(&self) -> H256
```
Return the block header hash.
```
pub trait CoinbaseTransactionBuilder
```
Let's set a trait for the functions to build the coinbase of the new block.
```
pub struct Solution
```
Now, let's create a new structure for calculating the nonce.
```
pub fn find_solution<T>(block: &BlockTemplate, mut coinbase_transaction_builder: T, max_extranonce: U256)
```
Then we have a public function that contains the proof of work magic, the coinbase creation and the merkle root hash creation.

Tests are included under mod tests. It's interesting to note that P2SH is used for the Coinbase builder test.
```
pub struct P2shCoinbaseTransactionBuilder
```

### Fee (fee.rs)
We need to pay the miners for their hard work calculating the nonces. We do this via Bitcoin fees.

```
pub fn transaction_fee
```
First, we declare a function for calculating the transaction fees to the miners.
```
let inputs_sum = transaction.inputs.iter().map(|input|
```
Then, we go about to collect the total of the transaction inputs.
```
let outputs_sum = transaction.outputs.iter().map(|output| output.value).sum();
```
Next we collect all the values of the outputs.
```
inputs_sum.saturating_sub(outputs_sum)
```
Subtracting the inputs from the outputs with the `saturating_sub` method yields the tranaction fee. You may learn more about [Saturating subs and other methods](https://doc.rust-lang.org/std/primitive.usize.html#method.saturating_sub)


### Memory Pool (memory_pool.rs)
Wow. This is huge! Okay. I need to take a break and let this file go to someone with time!
