<!--
# ? operator in macros
-->
# マクロ内の?演算子

![Minimum Rust version: 1.32](https://img.shields.io/badge/Minimum%20Rust%20Version-1.32-brightgreen.svg)

<!--
`macro_rules` macros can use `?`, like this:
-->
`macro_rules` マクロ内で、`?`演算子を以下のように使えます。

```rust
macro_rules! bar {
    ($(a)?) => {}
}
```

<!--
The `?` will match zero or one repetitions of the pattern, similar to the
already-existing `*` for "zero or more" and `+` for "one or more."
-->
既に利用可能な `*`演算子が「0回以上」、`+`演算子が「1回以上」の繰り返しパターンを表すのと同様に、`?`演算子は0もしくは1回のパターンを表します。
