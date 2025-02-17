<!--
# Match ergonomics reservations
-->

# マッチの利便性機能への新制約

<!--
## Summary
-->

## 概要

<!--
- Writing `mut`, `ref`, or `ref mut` on a binding is only allowed within a pattern when the pattern leading up to that binding is fully explicit (i.e. when it does not use match ergonomics).
  - Put differently, when the default binding mode is not `move`, writing `mut`, `ref`, or `ref mut` on a binding is an error.
- Reference patterns (`&` or `&mut`) are only allowed within the fully-explicit prefix of a pattern.
  - Put differently, reference patterns can only match against references in the scrutinee when the default binding mode is `move`.
-->

- パターン中の `mut`、`ref`、`ref mut` 束縛は、その束縛に至るまでのパターンが完全であるとき（マッチの利便性機能を使っていないとき）だけ使用できます。
  - つまり、現在の束縛モードが `move` 以外のとき、`mut`、`ref`、`ref mut` 束縛の使用はエラーになります。
- 参照パターン (`&` や `&mut`) は、その束縛に至るまでのパターンが完全であるときだけ使用できます。
  - つまり、参照パターンが被検査子中の参照にマッチできるのは、現在の束縛モードが `move` であるときのみです。

## Details

### Background

<!--
Within `match`, `let`, and other constructs, we match a *pattern* against a *scrutinee*.  E.g.:
-->

`match` や `let` などの束縛構文は、以下のように、評価対象の式である **被検査子** が、**パターン** にマッチさせる（合致するかどうか照合する）という構造になっています。

<!--
```rust
let &[&mut [ref x]] = &[&mut [()]]; // x: &()
//  ~~~~~~~~~~~~~~~   ~~~~~~~~~~~~
//      Pattern        Scrutinee
```
-->

```rust
let &[&mut [ref x]] = &[&mut [()]]; // x: &()
//  ~~~~~~~~~~~~~~~   ~~~~~~~~~~~~
//     パターン         被検査子
```

<!--
Such a pattern is called fully explicit because it does not elide (i.e. "skip" or "pass") any references within the scrutinee.  By contrast, this otherwise-equivalent pattern is not fully explicit:
-->

上記のようなパターンは、被検査子中の全参照を明示的に記述していることから、「完全な」パターンと呼びます。
一方で、以下のパターンは上記と等価ながら完全ではありません。

```rust
let [[x]] = &[&mut [()]]; // x: &()
```

<!--
Patterns such as this are said to be using match ergonomics, originally introduced in [RFC 2005][].
-->

このようなパターンは、元は [RFC 2005][] で導入された「マッチの利便性機能」を利用しています。

<!--
Under match ergonomics, as we incrementally match a pattern against a scrutinee, we keep track of the default binding mode.  This mode can be one of `move`, `ref mut`, or `ref`, and it starts as `move`.  When we reach a binding, unless an explicit binding mode is provided, the default binding mode is used to decide the binding's type.
-->

この利便性機能において、コンパイラは、被検査子をパターンに逐次マッチさせながら、「現在の束縛モード」を状態として持ちます。
束縛モードは最初 `move` で、`move`、`ref mut`、`ref` のいずれかをとります。
束縛に到達したとき、明示的な束縛モードの上書きがない限り、現在の束縛モードにより束縛方式が決まります。

<!--
For example, here we provide an explicit binding mode, causing `x` to be bound by reference:
-->

以下は明示的な束縛モードを指定して、`x` を参照束縛にしている例です。

```rust
let ref x = (); // &()
```

<!--
By contrast:
-->

一方で、

```rust
let [x] = &[()]; // &()
```

<!--
Here, in the pattern, we pass the outer shared reference in the scrutinee.  This causes the default binding mode to switch from `move` to `ref`.  Since there is no explicit binding mode specified, the `ref` binding mode is used when binding `x`.
-->

こちらでは、被検査子中の外部の共有参照をパターン中で引き渡しています。
これにより現在の束縛モードが `move` から `ref` に切り替わります。
束縛モードが明示されていないので、`ref` モードが `x` の束縛に使われます。

[RFC 2005]: https://github.com/rust-lang/rfcs/pull/2005

<!--
### `mut` restriction
-->

### `mut` を書ける場所への制限

<!--
In Rust 2021 and earlier editions, we allow this oddity:
-->

Rust 2021 以前のエディションでは、以下の不思議な挙動が仕様でした。

```rust,edition2021
let [x, mut y] = &[(), ()]; // x: &(), mut y: ()
```

<!--
Here, because we pass the shared reference in the pattern, the default binding mode switches to `ref`.  But then, in these editions, writing `mut` on the binding resets the default binding mode to `move`.
-->

パターンに共有参照を渡した時点では、現在の束縛モードが `ref` に切り替わっていますが、Rust 2021 以前では、束縛に `mut` を書くことで現在の束縛モードが `move` に戻ってしまいます。

<!--
This can be surprising as it's not intuitive that mutability should affect the type.
-->

変数を可変にするかどうかで型が変わってしまいました。非直感的な挙動ですね。

<!--
To leave space to fix this, in Rust 2024 it's an error to write `mut` on a binding when the default binding mode is not `move`.  That is, `mut` can only be written on a binding when the pattern (leading up to that binding) is fully explicit.
-->

Rust 2024 では、将来的にこの問題を修正するために、現在の束縛モードが `move` でないときに `mut` を束縛中に書くことはエラーになりました。
つまり、`mut` 束縛が書けるのは（そこに至るまでの）パターンが完全であるときだけです。

<!--
In Rust 2024, we can write the above example as:
-->

Rust 2024 では、上記のコードは以下のように書けます。

```rust
let &[ref x, mut y] = &[(), ()]; // x: &(), mut y: ()
```

### `ref` / `ref mut` restriction

<!--
In Rust 2021 and earlier editions, we allow:
-->

Rust 2021 以前のエディションでは、以下のように書けました。

```rust,edition2021
let [ref x] = &[()]; // x: &()
```

<!--
Here, the `ref` explicit binding mode is redundant, as by passing the shared reference (i.e. not mentioning it in the pattern), the binding mode switches to `ref`.
-->

ここで、`ref` 束縛モードを明示するのは冗長です。共有参照を渡した時点で（それがパターン側で言及されていないため）、束縛モードが `ref` に変わっているからです。

<!--
To leave space for other language possibilities, we are disallowing explicit binding modes where they are redundant in Rust 2024.  We can rewrite the above example as simply:
-->

今後の言語仕様の行く末を見越して、Rust 2024 では冗長な束縛モードの指定がエラーになります。
実際、上記のコードは以下で十分です。

```rust
let [x] = &[()]; // x: &()
```

### Reference patterns restriction

<!--
In Rust 2021 and earlier editions, we allow this oddity:
-->

Rust 2021 以前のエディションでは、以下の不思議な挙動が仕様でした。

```rust,edition2021
let [&x, y] = &[&(), &()]; // x: (), y: &&()
```

<!--
Here, the `&` in the pattern both matches against the reference on `&()` and resets the default binding mode to `move`.  This can be surprising because the single `&` in the pattern causes a larger than expected change in the type by removing both layers of references.
-->

ここで、パターン中の `&` には、`&()` にマッチした上で、現在の束縛モードを `move` に切り替える効果がありました。
一つ `&` を書いただけで、参照が二重に剥かれて、思いもよらぬ型になってしまいました。非直感的な挙動ですね。

<!--
To leave space to fix this, in Rust 2024 it's an error to write `&` or `&mut` in the pattern when the default binding mode is not `move`.  That is, `&` or `&mut` can only be written when the pattern (leading up to that point) is fully explicit.
-->

将来的にこの問題を修正するために、Rust 2024 では、現在の束縛モードが `move` でないときに `&` や `&mut` をパターン中に書くのはエラーになりました。
逆に、`&` と `&mut` が書けるのは、（そこに至るまでの）パターンが完全であるときだけです。

<!--
In Rust 2024, we can write the above example as:
-->

Rust 2024 では、上記のコードは以下のように書けます。

```rust
let &[&x, ref y] = &[&(), &()];
```

<!--
## Migration
-->

## 移行

<!--
The [`rust_2024_incompatible_pat`][] lint flags patterns that are not allowed in Rust 2024.  This lint is part of the `rust-2024-compatibility` lint group which is automatically applied when running `cargo fix --edition`.  This lint will automatically convert affected patterns to fully explicit patterns that work correctly in Rust 2024 and in all prior editions.
-->

[`rust_2024_incompatible_pat`][] リントで、Rust 2024 で新たに禁止されるパターンを検出できます。
`cargo fix --edition` を実行すると自動適用される `rust-2024-compatibility` リントグループの一部です。
このリントは、影響を受けるパターンを、Rust 2024 を含む全エディションで正しく動作するパターンに書き換えます。

<!--
To migrate your code to be compatible with Rust 2024, run:
-->

コードを Rust 2024 互換に移行するには、以下を実行します。

```sh
cargo fix --edition
```

<!--
For example, this will convert this...
-->

これにより、以下のコードは

```rust,edition2021
let [x, mut y] = &[(), ()];
let [ref x] = &[()];
let [&x, y] = &[&(), &()];
```

<!--
...into this:
-->

以下のコードに変換されます。

```rust
let &[ref x, mut y] = &[(), ()];
let &[ref x] = &[()];
let &[&x, ref y] = &[&(), &()];
```

<!--
Alternatively, you can manually enable the lint to find patterns that will need to be migrated:
-->

あるいは、移行が必要なパターンを検出するために手動でリンクを有効化できます。

<!--
```rust
// Add this to the root of your crate to do a manual migration.
#![warn(rust_2024_incompatible_pat)]
```
-->

```rust
// クレートのトップレベルに以下を追加すると手動移行できる
#![warn(rust_2024_incompatible_pat)]
```

[`rust_2024_incompatible_pat`]: ../../rustc/lints/listing/allowed-by-default.html#rust-2024-incompatible-pat
