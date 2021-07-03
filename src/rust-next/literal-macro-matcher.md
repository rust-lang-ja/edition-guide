<!--
# literal macro matcher
-->
# リテラルマクロマッチャ

![Minimum Rust version: 1.32](https://img.shields.io/badge/Minimum%20Rust%20Version-1.32-brightgreen.svg)

<!--
A new `literal` matcher was added for macros:
-->
リテラル（`literal`）マッチャがマクロに付け加えられました。

```rust
macro_rules! m {
    ($lt:literal) => {};
}

fn main() {
    m!("some string literal");
}
```

<!--
`literal` matches against literals of any type; string literals, numeric
literals, `char` literals.
-->
`literal`は、文字列リテラル、数値リテラル、文字リテラルなど、どんなタイプのリテラルにもマッチします。
