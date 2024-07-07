<p align="center">
    <img src="https://raw.githubusercontent.com/rkyv/rkyv/master/media/logo_text_color.svg" alt="rkyv">
</p>
<p align="center">
    rkyv (<em>archive</em>) is a zero-copy deserialization framework for Rust
</p>
<p align="center">
    <a href="https://discord.gg/65F6MdnbQh">
        <img src="https://img.shields.io/discord/822925794249539645" alt="Discord">
    </a>
    <a href="https://docs.rs/rkyv">
        <img src="https://img.shields.io/docsrs/rkyv.svg" alt="docs.rs">
    </a>
    <a href="https://crates.io/crates/rkyv">
        <img src="https://img.shields.io/crates/v/rkyv.svg" alt="crates.io">
    </a>
    <a href="https://github.com/rkyv/rkyv/blob/master/LICENSE">
        <img src="https://img.shields.io/badge/license-MIT-blue.svg" alt="MIT license">
    </a>
    <a href="https://blog.rust-lang.org/2021/07/29/Rust-1.54.0.html"
        <img src="https://img.shields.io/badge/rustc-1.54+-lightgray.svg" alt="rustc 1.54+">
    </a>
</p>

# Resources

## Learning Materials

- The [rkyv book](https://rkyv.github.io/rkyv) covers the motivation, architecture, and major
  features of rkyv
- The [rkyv discord](https://discord.gg/65F6MdnbQh) is a great place to get help with specific issues and meet
  other people using rkyv

## Documentation

- [rkyv](https://docs.rs/rkyv), the core library
- [rkyv_dyn](https://docs.rs/rkyv_dyn), which adds trait object support to rkyv

## Benchmarks

- The [rust serialization benchmark](https://github.com/djkoloski/rust_serialization_benchmark) is a
  shootout style benchmark comparing many rust serialization solutions. It includes special
  benchmarks for zero-copy serialization solutions like rkyv.

## Sister Crates

- [bytecheck](https://github.com/rkyv/bytecheck), which rkyv uses for validation
- [ptr_meta](https://github.com/rkyv/ptr_meta), which rkyv uses for pointer manipulation
- [rend](https://github.com/rkyv/rend), which rkyv uses for endian-agnostic features

# Example

```rust
use rkyv::{Archive, Deserialize, Serialize, rancor::Error};

#[derive(Archive, Deserialize, Serialize, Debug, PartialEq)]
#[rkyv(
    // This will generate a PartialEq impl between our unarchived and archived
    // types:
    compare(PartialEq),
    // bytecheck can be used to validate your data if you want. To use the safe
    // API, you have to derive CheckBytes for the archived type:
    check_bytes,
)]
// Derives can be passed through to the generated type:
#[rkyv_derive(Debug)]
struct Test {
    int: u8,
    string: String,
    option: Option<Vec<i32>>,
}

let value = Test {
    int: 42,
    string: "hello world".to_string(),
    option: Some(vec![1, 2, 3, 4]),
};

// Serializing is as easy as a single function call
let bytes = rkyv::to_bytes::<Error>(&value).unwrap();

// Or you can customize your serialization for better performance
// and compatibility with #![no_std] environments
use rkyv::ser::{Serializer, serializers::AllocSerializer};

let mut serializer = AllocSerializer::<0>::default();
serializer.serialize_value(&value).unwrap();
let bytes = serializer.into_serializer().into_inner();

// You can use the safe API for fast zero-copy deserialization
let archived = rkyv::check_archived_root::<Test>(&bytes[..]).unwrap();
assert_eq!(archived, &value);

// Or you can use the unsafe API for maximum performance
let archived = unsafe { rkyv::archived_root::<Test>(&bytes[..]) };
assert_eq!(archived, &value);

// And you can always deserialize back to the original type
let deserialized: Test = archived.deserialize(&mut rkyv::Infallible).unwrap();
assert_eq!(deserialized, value);
```

_Note: the safe API requires the `bytecheck` feature (enabled by default)_
_Read more about [available features](https://docs.rs/rkyv/latest/rkyv/#features)._

# Thanks

Thanks to all the sponsors that keep development sustainable. Special thanks to the following sponsors for going above and beyond supporting rkyv:

## Platinum Sponsors

<p align="center">
    <a href="https://dusk.network">
        <img src="https://raw.githubusercontent.com/rkyv/rkyv/master/media/sponsors/dusk_network.png" alt="Dusk Network">
    </a>
</p>

> Dusk Network is the first privacy blockchain for financial applications. Our mission is to enable any size enterprise to collaborate at scale, meet compliance requirements, and ensure that transaction data remains confidential.
