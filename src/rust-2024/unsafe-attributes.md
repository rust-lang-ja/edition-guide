> **Rust Edition Guide は現在 Rust 2024 のアップデート作業に向けて翻訳作業中です。本ページはある時点での英語版をコピーしていますが、一部のリンクが動作しない場合や、最新情報が更新されていない場合があります。問題が発生した場合は、[原文（英語版）](https://doc.rust-lang.org/nightly/edition-guide/introduction.html)をご参照ください。**

# Unsafe attributes

## Summary

- The following attributes must now be marked as `unsafe`:
    - [`export_name`]
    - [`link_section`]
    - [`no_mangle`]

[`export_name`]: https://doc.rust-lang.org/reference/abi.html#the-export_name-attribute
[`link_section`]: https://doc.rust-lang.org/reference/abi.html#the-link_section-attribute
[`no_mangle`]: https://doc.rust-lang.org/reference/abi.html#the-no_mangle-attribute

## Details

Rust 1.82 added the ability in all editions to mark certain attributes as `unsafe` to indicate that they have soundness requirements that must be upheld.[^RFC3325] The syntax for an unsafe attribute looks like this:

```rust
// SAFETY: there is no other global function of this name
#[unsafe(no_mangle)]
pub fn example() {}
```

Marking the attribute with `unsafe` highlights that there are safety requirements that must be upheld that the compiler cannot verify on its own.

Starting with the 2024 Edition, it is now required to mark these attributes as `unsafe`. The following section describes the safety requirements for these attributes.

[^RFC3325]: See [RFC 3325](https://rust-lang.github.io/rfcs/3325-unsafe-attributes.html) for the original proposal.

### Safety requirements

The [`no_mangle`], [`export_name`], and [`link_section`] attributes influence the symbol names and linking behavior of items. Care must be taken to ensure that these attributes are used correctly.

Because the set of symbols across all linked libraries is a global namespace, there can be issues if there is a symbol name collision between libraries. Typically this isn't an issue for normally defined functions because [symbol mangling] helps ensure that the symbol name is unique. However, attributes like `export_name` can upset that assumption of uniqueness.

For example, in previous editions the following crashes on most Unix-like platforms despite containing only safe code:

```rust,no_run,edition2021
fn main() {
    println!("Hello, world!");
}

#[export_name = "malloc"]
fn foo() -> usize { 1 }
```

In the 2024 Edition, it is now required to mark these attributes as unsafe to emphasize that it is required to ensure that the symbol is defined correctly:

```rust,edition2024
// SAFETY: There should only be a single definition of the loop symbol.
#[unsafe(export_name="loop")]
fn arduino_loop() {
    // ...
}
```

[symbol mangling]: https://doc.rust-lang.org/rustc/symbol-mangling/index.html
[`unsafe_attr_outside_unsafe`]: https://doc.rust-lang.org/rustc/lints/listing/allowed-by-default.html#unsafe-attr-outside-unsafe

## Migration

The [`unsafe_attr_outside_unsafe`] lint can update these attributes to use the `unsafe(...)` format. The lint is part of the `rust-2024-compatibility` lint group which is included in the automatic edition migration. In order to migrate your code to be Rust 2024 Edition compatible, run:

```sh
cargo fix --edition
```

Just beware that this automatic migration will not be able to verify that these attributes are being used correctly. It is still your responsibility to manually review their usage.

Alternatively, you can manually enable the lint to find places where these attributes need to be updated.

```rust
// Add this to the root of your crate to do a manual migration.
#![warn(unsafe_attr_outside_unsafe)]
```
