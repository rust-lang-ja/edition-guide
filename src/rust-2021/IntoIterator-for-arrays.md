<!--
# IntoIterator for arrays
-->

# 配列に対する IntoIterator

<!--
## Summary
-->

## 概要

<!--
- Arrays implement `IntoIterator` in *all* editions.
- Calls to `IntoIterator::into_iter` are *hidden* in Rust 2015 and Rust 2018 when using method call syntax
  (i.e., `array.into_iter()`). So, `array.into_iter()` still resolves to `(&array).into_iter()` as it
  has before.
- `array.into_iter()` changes meaning to be the call to `IntoIterator::into_iter` in Rust 2021.
-->

- *すべての*エディションで、配列が `IntoIterator` を実装するようになります。
- Rust 2015 と Rust 2018 では、メソッド呼び出し構文が使われても（つまり `array.into_iter()` と書いても）、
  `IntoIterator::into_iter` は*隠されて*います。
  これにより、`array.into_iter()` は従来どおり `(&array).into_iter()` に解決されます。
- Rust 2021 から、`array.into_iter()` が `IntoIterator::into_iter` を意味するように変更されます。

<!--
## Details
-->

## 詳細

<!--
Until Rust 1.53, only *references* to arrays implement `IntoIterator`.
This means you can iterate over `&[1, 2, 3]` and `&mut [1, 2, 3]`,
but not over `[1, 2, 3]` directly.
-->

Rust 1.53 より前は、配列の*参照*だけが `IntoIterator` を実装していました。
すなわち、`&[1, 2, 3]` と `&mut [1, 2, 3]` に対しては列挙できる一方で、`[1, 2, 3]` に対して列挙することはできませんでした。

<!--
```rust,ignore
for &e in &[1, 2, 3] {} // Ok :)

for e in [1, 2, 3] {} // Error :(
```
-->
```rust,ignore
for &e in &[1, 2, 3] {} // OK :)

for e in [1, 2, 3] {} // エラー :(
```

<!--
This has been [a long-standing issue][25], but the solution is not as simple as it seems.
Just [adding the trait implementation][20] would break existing code.
`array.into_iter()` already compiles today because that implicitly calls
`(&array).into_iter()` due to [how method call syntax works][22].
Adding the trait implementation would change the meaning.
-->

これは[古くからある Issue][25]ですが、見た目ほど解決は簡単ではありません。
[トレイト実装を追加する][20]だけでは、既存のコードが壊れてしまいます。
[メソッド呼び出し構文の仕組み][22]上、`array.into_iter()` は現状でも `(&array).into_iter()` とみなされてコンパイルが通ります。
トレイト実装を追加すると、その意味が変わってしまうのです。

<!--
Usually this type of breakage (adding a trait implementation) is categorized as 'minor' and acceptable.
But in this case there is too much code that would be broken by it.
-->

多くのケースで、この手の互換性破壊（トレイト実装の追加）は「軽微」で許容可能とみなされてきました。
しかし、このケースではあまりにも多くのコードが壊れてしまうのです。

<!--
It has been suggested many times to "only implement `IntoIterator` for arrays in Rust 2021".
However, this is simply not possible.
You can't have a trait implementation exist in one edition and not in another,
since editions can be mixed.
-->

何度も提案されてきたのは、「Rust 2021 でのみ配列に `IntoIterator` を実装する」ことでした。
しかし、これは単に不可能なのです。
エディションは併用されうるので、あるエディションではトレイト実装が存在して、別のエディションでは存在しない、というわけにはいかないからです。

<!--
Instead, the trait implementation was added in *all* editions (starting in Rust 1.53.0)
but with a small hack to avoid breakage until Rust 2021.
In Rust 2015 and 2018 code, the compiler will still resolve `array.into_iter()`
to `(&array).into_iter()` like before, as if the trait implementation does not exist.
This *only* applies to the `.into_iter()` method call syntax.
It does not affect any other syntax such as `for e in [1, 2, 3]`, `iter.zip([1, 2, 3])` or
`IntoIterator::into_iter([1, 2, 3])`.
Those will start to work in *all* editions.
-->

代わりに、(Rust 1.53.0 から)トレイト実装は*すべての*エディションで追加されましたが、
Rust 2021 より前のコードが破壊されないようにちょっとしたハックが行われました。
Rust 2015 と 2018 のコードでは、コンパイラは従来どおり `array.into_iter()` を `(&array).into_iter()` に解決し、あたかもトレイト実装が存在しないかのように振る舞います。
これは `.into_iter()` というメソッド呼び出し構文*だけ*に適用されます。
一方、このルールは `for e in [1, 2, 3]`, `iter.zip([1, 2, 3])`, `IntoIterator::into_iter([1, 2, 3])` といった他の構文には適用されず、
そのような書き方は*全ての*エディションで使えるようになります。

<!--
While it's a shame that this required a small hack to avoid breakage,
this solution keeps the difference between the editions to an absolute minimum.
-->

互換性破壊を防ぐためにちょっとしたハックが必要になったのは残念ですが、
これによりエディション間の違いが最小限になったのです。

[25]: https://github.com/rust-lang/rust/issues/25725
[20]: https://github.com/rust-lang/rust/pull/65819
[22]: https://doc.rust-lang.org/book/ch05-03-method-syntax.html#wheres-the---operator

<!--
## Migration
-->

## 移行

<!--
A lint, `array_into_iter`, gets triggered whenever there is some call to `into_iter()` that will change
meaning in Rust 2021. The `array_into_iter` lint has already been a warning by default on all editions 
since the 1.41 release (with several enhancements made in 1.55). If your code is already warning free, 
then it should already be ready to go for Rust 2021!
-->

`into_iter()` への呼び出しのうち、Rust 2021 で意味が変わるようなものに対しては、
`array_into_iter` というリントが発生します。
1.41 のリリース以降、`array_into_iter` リントはすでにデフォルトで警告として発出されています（1.55 ではさらにいくつかの機能追加が行われました）。
警告が今現在出ていないコードは、今すぐにでも Rust 2021 に進むことができます！

<!--
You can automatically migrate your code to be Rust 2021 Edition compatible or ensure it is already compatible by
running:
-->

コードを自動的に Rust 2021 エディションに適合するよう自動移行するか、既に適合するものであることを確認するためには、以下のように実行すればよいです:

```sh
cargo fix --edition
```

<!--
Because the difference between editions is small, the migration to Rust 2021 is fairly straight-forward.
-->

エディション間の違いが少ないので、Rust 2021 への移行も非常に簡単です。

<!--
For method calls of `into_iter` on arrays, the elements being implemented will change from references to owned values.
-->

配列に対する `into_iter` のメソッド呼び出しに関しては、(<!--訳注-->イテレータの)要素が参照でなく所有権を持った値となります。

<!--
For example:
-->

例えば：

<!--
```rust
fn main() {
  let array = [1u8, 2, 3];
  for x in array.into_iter() {
    // x is a `&u8` in Rust 2015 and Rust 2018
    // x is a `u8` in Rust 2021
  }
}
```
-->

```rust
fn main() {
  let array = [1u8, 2, 3];
  for x in array.into_iter() {
    // Rust 2015 と Rust 2018 では、x は `&u8`
    // Rust 2021 では、x は `u8`
  }
}
```

<!--
The most straightforward way to migrate in Rust 2021, is by keeping the exact behavior from previous editions
by calling `iter()` which also iterates over owned arrays by reference:
-->

移行のための最も簡単な方法は、前のエディションと完全に同じ挙動をするように、
所有権を持った配列上を参照でイテレートするもう一つの関数 `iter()` を呼び出すことです:

<!--
```rust
fn main() {
  let array = [1u8, 2, 3];
  for x in array.iter() { // <- This line changed
    // x is a `&u8` in all editions
  }
}
```
-->

```rust
fn main() {
  let array = [1u8, 2, 3];
  for x in array.iter() { // <- この行を書き換えた
    // x はすべてのエディションで `&u8`
  }
}
```

<!--
### Optional migration
-->

### 必須でない移行

<!--
If you are using fully qualified method syntax (i.e., `IntoIterator::into_iter(array)`) in a previous edition,
this can be upgraded to method call syntax (i.e., `array.into_iter()`).
-->

前のエディションで完全修飾メソッド構文を使っていた場合（例: `IntoIterator::into_iter(array)`）、
これはメソッド呼び出し構文に書き換え可能です（例: `array.into_iter()`）。
