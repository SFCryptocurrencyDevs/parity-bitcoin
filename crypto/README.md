# Crypto
Building off of the primitives crate, this crate provides a toolkit of cryptographic hash functions.

## Dependencies
#### 1. [rust-crypto](https://crates.io/crates/rust-crypto): A (mostly) pure-Rust implementation of various common cryptographic algorithms. 

**One thing to note:** *Rust-Crypto has not been thoroughly audited for correctness, so any use where security is important is not recommended at this time.*

#### 2. [siphasher](https://crates.io/crates/siphasher): SipHash is a general-purpose hashing function: it runs at a good speed (competitive with Spooky and City) and permits strong _keyed_ hashing.

**One thing to note:** *Although the SipHash algorithm is considered to be generally strong, it is not intended for cryptographic purposes. As such, all cryptographic uses of this implementation are _strongly discouraged_.*

Reference:
* [Research Paper](https://web.kamihq.com/web/viewer.html?state=%7B%22ids%22%3A%5B%221xWTF87NjRf-eEmmjBTG-2DQt8aeAQllR%22%5D%2C%22action%22%3A%22open%22%2C%22userId%22%3A%22109785094086647974039%22%7D&filename=siphash.pdf)
* [Slides](https://web.kamihq.com/web/viewer.html?state=%7B%22ids%22%3A%5B%221rKfLlA30LoQEOHn3psVha2qkGFGVnllo%22%5D%2C%22action%22%3A%22open%22%2C%22userId%22%3A%22109785094086647974039%22%7D&filename=slides.pdf)

#### 3. [primitives](https://github.com/robertDurst/parity-bitcoin/tree/master/primitives): ref another crate within this project

## Structure
Lib.rs: This embodies the entirety of the crypto crate.

## Content
**DHash160**: a struct containing two fields:
* *SHA256* -- an imported type from rust-crypto representing a SHA256 hash object
	Read more about SHA256 [here.](https://en.bitcoin.it/wiki/SHA-256)
* *Ripemd160* -- an imported type from rust-crypto representing a Ripemd160 hash object
	Read more about Ripemd160 [here.](https://en.bitcoin.it/wiki/RIPEMD-160)

**DHash160** is used to SHA256 + Ripemd160 a message, a common function in this Bitcoin library.

This type implements the Digest trait from the rust-crypto crate. This means it must implement the following characteristics:
* **input**: takes an array of bytes and sets the message data
* **result**: retrieves the digest result -- this method may be called multiple times
* **reset**: resets digest -- must be called after result before supplying more data
* **output_bits**: output size (bits)
* **block_size**: block size (bits)

In rust, a trait is 

> a collection of methods defined for an unknown type: `Self`. They can access other methods declared in the same trait.

Read more about traits [here.](https://doc.rust-lang.org/rust-by-example/trait.html)

**DHash256**: a struct containing two fields:
* *Hasher* -- an imported type from rust-crypto representing a SHA256 hash object
	Read more about SHA256 [here.](https://en.bitcoin.it/wiki/SHA-256)

This also implements the **Digest** trait from the rust-crypto crate.

**DHash256** is used to double SHA256 a message, a common function in this Bitcoin library.

***
#### Using the above, we create some common cryptographic functions that are used throughout the Parity Bitcoin client:
* **ripemd160** (input message) --> output hash160
* **sha1** (input message) --> output hash160
* **sha256** (input message) --> output hash256
* **dhash160** (input message) --> output hash160
* **dhash256** (input message) --> output hash256
* **siphash24** (input key0, key1, message) --> unsigned int 64
* **checksum** (input message) --> output hash32

*Note: In the above, hash160 means a 160-bit hash value.*

Most of these functions are straight forward. The two most interesting are:

**siphash24:** this computes a 64-bit message authentication code from a message and two 64-bit keys. This is an efficient way of preventing DOS attacks and a safer alternative to the unkeyed SHA hash function since SHA is only collision-resistant if the entire output is used ([ref - wikipedia](https://en.wikipedia.org/wiki/SipHash#cite_note-11)).

**checksum:** the checksum is a short (32-bit) hash that confirms the correctness of another piece of data. For example, bitcoin addresses have a checksum at the end that confirms the validity of the address -- if even one character is incorrect, the checksum will not match up. A checksum is created by double-hashing (dhash256) some message and then taking the first four bytes.
