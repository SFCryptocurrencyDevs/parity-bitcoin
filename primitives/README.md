# Primitives
This crate contains the lowest level Bitcoin primitives, which many other crates within this repo depend on.

## Conceptual Overview
Primitives are like atoms -- they are the most basic, indivisible building blocks that make up everything else.

Here we define three fundamental primitives:
* **bytes**: a wrapper around `Vec<u8>`
* **compact**: a compact representation of `U256`
* **hash**: fixed size hashes

We will give an overview of bits/bytes here, but forego a deep dive into **compact** or **hash** since most of the overview will happen in the *Content* section.

### Bytes
**Bit**: the smallest unit of storage that exists as a 1 or a 0.

**Byte:** is a collection of 8 bits.

You can brush up on bits and bytes [here](https://web.stanford.edu/class/cs101/bits-bytes.html).

In this crate, a byte is represented as a wrapper around a `Vec<u8>`.

`u8`: *the 8-bit unsigned integer type* -- the [Rust Book](https://doc.rust-lang.org/1.15.1/std/primitive.u8.html) Definition

`vector`: a vector in rust is a collection of one type of data where each value is stored sequentially in memory. It is similar to your typical array or list except that, as stated, it can only store one type. One way to hack around the single type is to create a vector of a custom enum type:

```rust
enum AnyType {
    Int(i32),
    Float(f64),
    Text(String),
}

let list = vec![
    AnyType::Float(1.27),
    AnyType::Text(String::from("pizza")),
    AnyType::Int(42),
];
```

You can read more about vectors in rust [here](https://doc.rust-lang.org/book/second-edition/ch08-01-vectors.html).


## Dependencies
#### 1. [heapsize](https://crates.io/crates/heapsize): infrastructure for measuring the total runtime size of an object on the heap
#### 2. [rustc-serialize](https://crates.io/crates/rustc-serialize): Serialization and deserialization support provided by the compiler.

**One thing to note**: *This crate is deprecated in favor of [`serde`](https://serde.rs/). No new feature development will happen in this crate, although bug fixes proposed through PRs will still be merged. It is very highly recommended by the Rust Library Team that you use [`serde`](https://serde.rs/), not this crate.*
#### 3. [byteorder](https://crates.io/crates/byteorder): convenience methods for encoding and decoding numbers in either big-endian or little-endian order.

*So what are big-endian and little-endian?*

Big-endian and little-endian describe the order in which sequences of bytes are stored in computer memory -- [ref](https://searchnetworking.techtarget.com/definition/big-endian-and-little-endian). 

Consider the hexadecimal number `2EA1`.  

**big-endian**: most significant stored first. Thus, `2E` will be stored at `0` and `A1` will be stored at `1`.

**little-endian**: least significant stored first. Thus, `A1` will be stored at `0` and `2E` will be stored at `1`.

#### 4. [bigint](https://crates.io/crates/bigint) a crate developed by Parity Technologies that provides large fixed-size integers arithmetics

The availible unsigned integers are:
* u128 (*as of rust 1.26 this is [now part of the std library](https://doc.rust-lang.org/std/u128/)*)
* u256
* u512

**Note:** there are no internal dependencies here ✊

## Content

### Bytes (bytes.rs)
This defines two types (as structs), `Bytes` and `TaggedBytes`. 

For `Bytes` it implements many different traits including:
* varieties of the [std::convert::From](https://doc.rust-lang.org/std/convert/trait.From.html). Essentially this just defines how the compiler would convert from one type to another.
* [std::str::FromStr](https://doc.rust-lang.org/std/str/trait.FromStr.html). Defines how the compiler should create a `Bytes` type from a str.
  * **str**: immutable, fixed-size string
  * **String**: growable, heap-allocated string
  * Strings in rust are quite complex and not straight forward. [Here](http://www.ameyalokare.com/rust/2017/10/12/rust-str-vs-String.html) is an explanation of `str` vs `&str` vs `String` vs `&String`. [Here](https://doc.rust-lang.org/book/second-edition/ch08-02-strings.html) is a more general explanation of strings as a collection.

* [HeapSizeOf](https://docs.rs/heapsize/0.2.4/heapsize/trait.HeapSizeOf.html): *Measure the size of any heap-allocated structures that hang off this value, but not the space taken up by the value itself*
* [std::io::Write](https://doc.rust-lang.org/std/io/trait.Write.html): *attempt to write some data into the object, returning how many bytes were successfully written*
* [std::fmt::Debug](https://doc.rust-lang.org/std/fmt/trait.Debug.html): *when used with the alternate format specifier `#?`, the output is pretty-printed*
* [std::ops::Deref](https://doc.rust-lang.org/beta/std/ops/trait.Deref.html): a trait implemented for [smart pointers](https://doc.rust-lang.org/book/second-edition/ch15-00-smart-pointers.html) that defines how data should be accessed via the *the (unary) `*` operator in immutable contexts*
* [std::ops::DerefMut](https://doc.rust-lang.org/beta/std/ops/trait.DerefMut.html): same as `Deref` but mutable
* [AsRef](https://doc.rust-lang.org/std/convert/trait.AsRef.html): *used to convert a value to a reference value within generic code* in the context of converting *to a reference of another type*. Different than borrowing which is *is more related to the notion of taking the reference*. `AsRef` *is useful when wishing to abstract over the type of reference (`&T`, `&mut T`) or allow both the referenced and owned type to be treated in the same manner*.
* [AsMut](https://doc.rust-lang.org/std/convert/trait.AsMut.html): same as `AsRef` but mutable

`TaggedBytes` is also a wrapper around `Vec<u8>` however it also represents an [associated type](https://doc.rust-lang.org/book/first-edition/associated-types.html). In rust, an associated type essentially groups multiple types together into a sort of "type family."

So, if we look at the definition of `TaggedBytes` we see:
```rust
pub struct TaggedBytes<T> {
	bytes: Bytes,
	label: marker::PhantomData<T>,
}
```

where we define bytes as `Bytes` and label as `marker::PhantomData<T>` where in the Rust Book, `marker::PhantomData<T>` is defined as:

>Adding a PhantomData<T> field to your type tells the compiler that your type acts as though it stores a value of type T, even though it doesn't really. This information is used when computing certain safety properties.

Thus, `TaggedBytes` are `Bytes` with an associated, or tagged type. 


### Compact
This file contains the `Compact` type, a wrapper around a `u32` that represents a compact `u256`.

As you can probably see, the difficulty here will be losing significant digits when converting from u32 <-> u256 (in fact this is basically all that happens in this file). If you are interested in how this actually works, well, it's all in the code :).

**One thing to note:** the `Compact::to_u256` method is used to compute the *target [0, T] that a blockhash must land in to be valid*. If you are interested in the significance of this, checkout the **miner** crate.

### Hash
Before we can jump into this, we will need to take a deep dive into some pretty gnarly rust. So put on your hard hat, it's going get wild! **If you are just skimming and not interested in learning rust, feel free to skip to the TLDR at the end!**

![hard hat guy pointing](https://media.giphy.com/media/11FbZuRZ3vS19K/giphy.gif)

#### [Macros](https://doc.rust-lang.org/1.7.0/book/macros.html) in Rust: What, Why, How?
In rust, there are a few basic ways to abstract and reuse code:
* type signatures
* trait bounds
* overloaded functions

Macros allow for a more general abstraction of code, specifically, a *macro invocation is shorthand for an "expanded" syntactic form*.

Sounds great right? Well, before making any conclusions, consider the falling drawbacks:
* *fewer of the built-in rules apply.*
* harder to debug: compiler errors *describe problems in the expanded code, not the source-level form that developers use*
* *it can be difficult to design a well-behaved macro!*

Alright, let's begin!

**Note**: macros are cool, but not easy, so we won't go into the nitty gritty details here; instead we will go over the parts of macros required to understand this crate.

* **Defining a macro:** to define a macro, use the `macro_rules!` follwed by the macro name:

```rust
macro_rules! foo { // insert_code_here }
```
* **Keywords**: within macros, there are certain keywords used to denote various "special variables/objects" (in quotes because I am unsure of the proper label). [Here](https://doc.rust-lang.org/1.7.0/reference.html#macro-by-example) is a more complete list, however only the following are important for us:
   * [ident](https://doc.rust-lang.org/1.7.0/reference.html#identifiers): an identifier -- *any nonempty Unicode2 string*
   * [expr](https://doc.rust-lang.org/1.7.0/reference.html#expressions): a rust expression -- *an expression may have two roles: it always produces a value, and it may have effects*

* **What goes in the body?** In the body of the macro goes abstract/reusable code you wish to use over and over again. In this crate in particular, the majority of the body is implementing traits for a hash function; essentially all these traits are implemented in the same way, however the `$name` identifier and the `$size` expression "variables" fill in the parts that are different per hash implementation.

**TLDR**: the macro defined in this file, `impl_hash!` take in two "variables" (rough use of the word variables here) and fills in these variables per `impl_hash!`, constructing eight slightly different, fixed-size hash functions. We use a macro here instead of a more common, or "core" rust abstract tool, because we are looking to replicate an entire code section, not just a single function or trait.
