<!--
# Anonymous trait function parameters deprecated
-->

# トレイト関数の匿名パラメータの非推奨化

<!--
![Minimum Rust version: 1.31](https://img.shields.io/badge/Minimum%20Rust%20Version-1.31-brightgreen.svg)
-->

![導入 Rust バージョン: 1.31](https://img.shields.io/badge/%E5%B0%8E%E5%85%A5%20Rust%20%E3%83%90%E3%83%BC%E3%82%B8%E3%83%A7%E3%83%B3-1.31-brightgreen.svg)

<!--
## Summary
-->

## 概要

<!--
- [Trait function parameters] may use any irrefutable pattern when the function has a body.
-->

- 関数に本体があるとき、[トレイト関数のパラメータ]は、任意の論駁不可能なパターンを使えます。

<!--
[Trait function parameters]: https://doc.rust-lang.org/stable/reference/items/traits.html#parameter-patterns
-->

[トレイト関数のパラメータ]: https://doc.rust-lang.org/stable/reference/items/traits.html#parameter-patterns


<!--
## Details
-->

## 詳細

<!--
In accordance with RFC [#1685](https://github.com/rust-lang/rfcs/pull/1685),
parameters in trait method declarations are no longer allowed to be anonymous.
-->

RFC [#1685](https://github.com/rust-lang/rfcs/pull/1685)に基づいて、トレイト関数のパラメータを匿名にすることはできなくなりました。

<!--
For example, in the 2015 edition, this was allowed:
-->

例えば、2015 エディションでは、以下のように書けました:

```rust
trait Foo {
    fn foo(&self, u8);
}
```

<!--
In the 2018 edition, all parameters must be given an argument name  (even if it's just
`_`):
-->

2018 エディションでは、すべての引数名に（ただの `_` であってもいいので、何らかの）名前がついていなければなりません:

```rust
trait Foo {
    fn foo(&self, baz: u8);
}
```
