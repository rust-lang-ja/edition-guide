> **Rust Edition Guide は現在 Rust 2024 のアップデート作業に向けて翻訳作業中です。本ページは英語版をコピーしていますが、一部のリンクが動作しないなどの問題が発生する場合があります。問題が発生した場合は、[原文（英語版）](https://doc.rust-lang.org/nightly/edition-guide/introduction.html)をご参照ください。**

# Rustfmt: Raw identifier sorting

## Summary

`rustfmt` now properly sorts [raw identifiers].

[raw identifiers]: ../../reference/identifiers.html#raw-identifiers

## Details

The [Rust Style Guide] includes [rules for sorting][sorting] that `rustfmt` applies in various contexts, such as on imports.

Prior to the 2024 Edition, when sorting rustfmt would use the leading `r#` token instead of the ident which led to unwanted results. For example:

```rust,ignore
use websocket::client::ClientBuilder;
use websocket::r#async::futures::Stream;
use websocket::result::WebSocketError;
```

In the 2024 Edition, `rustfmt` now produces:

```rust,ignore
use websocket::r#async::futures::Stream;
use websocket::client::ClientBuilder;
use websocket::result::WebSocketError;
```

[Rust Style Guide]: ../../style-guide/index.html
[sorting]: ../../style-guide/index.html#sorting

## Migration

The change can be applied automatically by running `cargo fmt` or `rustfmt` with the 2024 Edition. See the [Style edition] chapter for more information on migrating and how style editions work.

[Style edition]: rustfmt-style-edition.md
