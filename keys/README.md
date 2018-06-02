# Keys
This crate contains the code necessary to generate, construct, represent, and use keys.

## Conceptual Overview
 The premise of Bitcoin is the transfer of value between addresses. This overview will go into the details of how addresses are generated.

**Public Key Cryptography**

In public key cryptography, all participants own a keypair which consists of a public key and a private key. Consider the mailbox analogy. A public key is like a physical address -- you would give this to a friend who wants to send you a letter. Once you receive the mail, you use your key (private key) to open your mailbox and retrieve the letter.

There are multiple cryptosystems that can be used for public key cryptography including (but not limited to):

* [RSA](https://web.kamihq.com/web/viewer.html?source=extension_pdfhandler&file=http%3A%2F%2Fwww.mathaware.org%2Fmam%2F06%2FKaliski.pdf)
* [El Gamal](https://web.kamihq.com/web/viewer.html?source=extension_pdfhandler&file=http%3A%2F%2Fhomepages.math.uic.edu%2F~leon%2Fmcs425-s08%2Fhandouts%2Fel-gamal.pdf)
* [Elliptic Curve](https://web.kamihq.com/web/viewer.html?state=%7B%22ids%22%3A%5B%221n-R5VU1rz4_V3BrD2b59Bqxc7nn9hGSq%22%5D%2C%22action%22%3A%22open%22%2C%22userId%22%3A%22109785094086647974039%22%7D&filename=elliptic.pdf)

Bitcoin uses Elliptic Curve Cryptography.

**Digital Signatures**

Here I will copy-pasta from [my final paper](https://drive.google.com/file/d/0B-8i6GVPN2ORTXhfNWY5Tm1ZbWtGNWljaVc2UHdwR2RVcS1z/view) for the introductory cryptography class I took Spring 2017.

> Nearly everyday people sign physical documents. This signature verifies the authenticity of the document; it creates a receipt that verifies a transaction is valid or a document is authorized/official.
> In the 21st century, we deal with digital documents, digital messages, and
digital transactions. How can we verify/authenticate these things in the same way we do for physical things? The answer is digital signatures. A digital signature is essentially a mathematical scheme for verifying/authenticating data.

Utilizing public key cryptography, we can create schemes for proving message M was sent from user X by *signing* M with X's private key.

These are implemented in various ways depending on the cryptosystem used.

**Elliptic Curve Cryptography**

To understand Bitcoin and understand the code from this crate, an understanding of Elliptic Curve Cryptography is not necessary. Nice right?

![Guy nodding](https://media.giphy.com/media/WJjLyXCVvro2I/giphy.gif)

Just know the term `secp256k1` -- the specific elliptic curve Bitcoin uses. It looks a bit like this:

![Secp256k1 Curve](https://cdn-images-1.medium.com/max/1600/1*4dcCrlQfGqZECDLy-25fnw.png)

However, if you really want to be an OG Bitcoin dev, learning the nitty-gritty details of elliptic curve cryptography may be a good investment of your time. Here are a few good reads:
* [What is the math behind elliptic curve cryptography?](https://hackernoon.com/what-is-the-math-behind-elliptic-curve-cryptography-f61b25253da3)
* [Elliptic Curve Cryptography
An Implementation Guide](http://www.infosecwriters.com/text_resources/pdf/Elliptic_Curve_AnnopMS.pdf)

**Generating a Bitcoin Address**
1. Randomly generate a private key (just a random 32 byte unsigned integer)
2. Derive the public key using Elliptic Curve math. In Bitcoin there are two types of public keys:
   * Compressed (33 byte, 1 byte 0x03 or 0x02, and 32 bytes corresponding called X)
   * Uncompressed -- older (65 bytes, consisting of constant prefix (0x04), followed by two 32 byte integers called X and Y)
3. SHA256 the public key
4. RIPEMD160 the result of step 3
5. Add a version byte in front of the result of step 4 ([read more here](https://en.bitcoin.it/wiki/List_of_address_prefixes)):
   * P2PKH Mainnet: 0x00
   * P2SH Mainnet: 0x05 
   * P2PKH Testnet: 0x6F 
   * P2SH Testnet: 0xC4
6. Take a copy of step 5 and set it aside
7. SHA256 the result of step 5
8. SHA256 the result of step 7
9. Take the first 4 bytes of step 8 as the [address checksum](http://learnmeabitcoin.com/glossary/checksum)
10. Append step 9 to the end of the copy of step 5 we set aside
11. Convert to a [Base58](https://en.bitcoin.it/wiki/Base58Check_encoding) string

If you ever feel the urge to do this by hand and need a place to check your work, [check out this address validator](http://lenschulwitz.com/base58).

***

## Dependencies
#### 1. [rand](https://crates.io/crates/rand):
*A Rust library for random number generation.
Rand provides utilities to generate random numbers, to convert them to useful types and distributions, and some randomness-related algorithms.
The core random number generation traits of Rand live in the rand_core crate; this crate is most useful when implementing RNGs.*
#### 2. [rustc-serialize](https://crates.io/crates/rustc-serialize):
Serialization and deserialization support provided by the compiler.

**One thing to note**: *This crate is deprecated in favor of [`serde`](https://serde.rs/). No new feature development will happen in this crate, although bug fixes proposed through PRs will still be merged. It is very highly recommended by the Rust Library Team that you use [`serde`](https://serde.rs/), not this crate.*
#### 3. [lazy_static](https://crates.io/crates/lazy_static):
*A macro for declaring lazily evaluated statics in Rust.
Using this macro, it is possible to have statics that require code to be executed at runtime in order to be initialized. This includes anything requiring heap allocations, like vectors or hash maps, as well as anything that requires non-const function calls to be computed.*
#### 4. [base58](https://crates.io/crates/base58):
*Tiny and fast base58 encoding*
#### 5. [eth-secp256k1](https://github.com/paritytech/rust-secp256k1): 
*rust-secp256k1 is a wrapper around libsecp256k1, a C library by Peter Wuille for producing ECDSA signatures using the SECG curve secp256k1*

***

## Context

### network.rs
In Bitcoin, addresses are formatted differently per network. This file simply defines a `Network` enum with two variants, `Mainnet` and `Testnet`.

### generator.rs
Here we define `Generator`, a trait that implements a method called generate and `Random`, a struct.

`Random` is quite simple. It has a single field that holds a variant of a `Network` enum. It has a static `new` method, a method commonly implemented on structs in rust, that returns a new instance of `Random`. Finally `Random` implements the `Generator` trait. It does so by implementing the `generate` function, returning a `Result` with a `SECP256K1` keypair.

### public.rs
Here the `Public` enum is defined along with several helpful methods.

**In coming rust learning moment!**

![Man saying yeah](https://media.giphy.com/media/nFjDu1LjEADh6/giphy.gif)

`Public` itself is an enum with two fields:
* `Normal(H520)`: an uncompressed public key. 
* `Compressed(H264)`: a compressed public key


Both these variants may look a little odd... is `Normal` a function?!? No! These are called [tuple structs](https://doc.rust-lang.org/book/second-edition/ch05-01-defining-structs.html#tuple-structs-without-named-fields-to-create-different-types). In rust tuple structs are just like normal structs except without named parameters. These are useful when you want to create a named type without the verbosity or redundancy of naming the interior fields.

Remember from the primitives crate that `H520` and `H264` are fixed-size hashes.

Now on to `Public`'s methods and trait implementations.

`Public` defines the following methods:
* **from_slice**: takes a raw 33 byte (compressed) or 65 byte (uncompressed) [u8] slice (bytes) and returns a `Public` variant.
* **address_hash**: returns the double SHA256 hash of this address.
* **verify**: takes in a `Message` and a `Signature` and returns a Result with a boolean. This function takes the signature and message data passed in and formats it so that it is able to perform the ECDSA math to verify if the signature + message combo is correct.
* **recover_compact**: derives the public key from a message and a compact signature. 

`Public` implements the following traits:
* **Deref**: allows for custom immutable dereferencing. Read more [here](https://doc.rust-lang.org/beta/std/ops/trait.Deref.html).
* **PartialEq**: defines how partial equality works. Allows you to compare two public keys.
* **fmt::Debug**: defines how this would be represented when printed using: 
   
   `println!("{:?}", val);`
* **fmt::Display**:  defines how this would be represented when printed normally: 

   `println!("{}", val);`


### private.rs
Here the `Private` struct, representing private keys, is defined.

This struct has the following fields:
* **network**: the network on which this key should be used.
* **secret**: the ECDSA key.
* **compressed**: a bool which will be true if this private key represents a compressed address

The primary methods for this struct are the signing methods: `sign` and `sign_compact`. These are relatively straight forward since they just use the rust `SECP256K1` crate. 

`Private` implements `DisplayLayout` which has two methods, `layout` and `from_layout`, that encode/decode the private key from `Private` <-> `&[u8]` (bytes). See above in the **Conceptual Overview** section for how this key is generated.


### keypair.rs
Here the `KeyPair` struct is defined along with several helpful methods.

`KeyPair` itself is a struct with two fields:
* `private`: an variant of the `Private` enum
* `public`: an variant of the `Public` enum

Besides a couple basic getter methods and convenient debugging/display trait implementations (`fmt::Debug` and `fmt::Display`), `KeyPair` also has the following, more interesting, methods:
* `from_private`: takes in a Bitcoin private key and returns a Result with a `KeyPair`.
   1. Deconstructs Bitcoin private key into a SECP256K1 private key
   2. Derives a SECP256K1 public key from the SECP256K1 private key
   3. Constructs the Bitcoin public key from the SECP256K1 public key (slightly different depending on whether or not the SECP256K1 public key is compressed/uncompressed)

* `from_keypair`: takes in a SECP256K1 keypair along with the network and returns a Bitcoin `KeyPair`. This is used by the `Random` struct above in its implementation of `Generator`. *Note*: this produces only uncompressed public keys. 

### signature.rs
This file does not actually implement the signing function... that happens in the imported [SECP256K1 crate](https://github.com/paritytech/rust-secp256k1). 

Instead, it defines two tuple structs `Signature` and `CompactSignature` with various trait implentations for easy use.

Here I will copy-pasta an excellent explanation of the difference between these two from the [Bitcoin StackExchange](https://bitcoin.stackexchange.com/questions/12554/why-the-signature-is-always-65-13232-bytes-long).

Credits to Peter Wuille.

> There are two different encodings used.
> 
>Everything in the Bitcoin protocol, including transaction signatures and alert signatures, uses DER encoding. This results in 71 bytes signatures (on average), as there are several header bytes, and the R and S valued are variable length.
>
>For message signatures, a custom encoding is used which is more compact (and more recent) and supports public key recovery (given a message and a signature, find which public key would have created it). The code you're referring to in the question is for creating such signatures.
>
>A correct DER-encoded signature has the following form:
>
>* 0x30: a header byte indicating a compound structure.
>* A 1-byte length descriptor for all what follows.
>* 0x02: a header byte indicating an integer.
>* A 1-byte length descriptor for the R value
>* The R coordinate, as a big-endian integer.
>* 0x02: a header byte indicating an integer.
>* A 1-byte length descriptor for the S value.
>* The S coordinate, as a big-endian integer.
>
>Where initial 0x00 bytes for R and S are not allowed, except when their first byte would otherwise be above 0x7F (in which case a single 0x00 in front is required). Also note that inside transaction signatures, an extra hashtype byte follows the actual signature data.


### display.rs
Defines the `DisplayLayout` trait. This trait implements two methods: `layout` and `from_layout` which describe how to encode/decode a value.

### error.rs
Here an enum is defined with all possible errors that may result from methods in this crate. This is useful for capturing errors of different types, allowing functions to always return a `Result` where the `Err` type is consistent.
