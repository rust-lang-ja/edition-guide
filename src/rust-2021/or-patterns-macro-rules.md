<!--
# Or patterns in macro-rules
-->

# マクロ規則における OR パターン

<!--
## Summary
-->

## 概要

<!--
- How patterns work in `macro_rules` macros changes slightly:
	- `$_:pat` in `macro_rules` now matches usage of `|` too: e.g. `A | B`.
	- The new `$_:pat_param` behaves like `$_:pat` did before; it does not match (top level) `|`.
	- `$_:pat_param` is available in all editions.
-->

- `macro_rules` におけるパターンの挙動がほんの少し変更されました:
	- `macro_rules` において、`$_:pat` で `|` を使ったパターンにもマッチするようになりました。例えば、`A | B` にマッチします。
	- 新しく導入された `$_:pat_param` は、かつての `$_:pat` と同じ挙動を再現します。すなわち、こちらは（トップレベルの）`|` にはマッチしません。
	- `$_:pat_param` は全てのエディションで使用可能です。



<!--
## Details
-->

## 詳細

<!--
Starting in Rust 1.53.0, [patterns](https://doc.rust-lang.org/stable/reference/patterns.html)
are extended to support `|` nested anywhere in the pattern.
This enables you to write `Some(1 | 2)` instead of `Some(1) | Some(2)`.
Since this was simply not allowed before, this is not a breaking change.
-->

Rust 1.53.0 から、[パターン](https://doc.rust-lang.org/stable/reference/patterns.html)中のどこでも、`|` をネストして使えるようになりました。
これにより、`Some(1) | Some(2)` でなく `Some(1 | 2)` と書くことができるようになりました。
今まではこうは書けなかったので、これは破壊的変更ではありません。

<!--
However, this change also affects [`macro_rules` macros](https://doc.rust-lang.org/stable/reference/macros-by-example.html).
Such macros can accept patterns using the `:pat` fragment specifier.
Currently, `:pat` does *not* match top level `|`, since before Rust 1.53,
not all patterns (at all nested levels) could contain a `|`.
Macros that accept patterns like `A | B`,
such as [`matches!()`](https://doc.rust-lang.org/1.51.0/std/macro.matches.html)
use something like `$($_:pat)|+`. 
-->

ところが、この変更は [`macro_rules` マクロ](https://doc.rust-lang.org/stable/reference/macros-by-example.html) にも影響します。
`macro_rules` では、`:pat` というフラグメント指定子で、パターンを受け付けることができます。
現在のところ、`:pat` はトップレベルの `|` にマッチ*しません*。
なぜなら Rust 1.53 以前は、全てのパターンが（どのネストレベルにでも）`|` を含むことができるわけではなかったからです。
[`matches!()`](https://doc.rust-lang.org/1.51.0/std/macro.matches.html) のように、
`A | B` のようなパターンを受け付けるマクロを書くには、
`$($_:pat)|+` のような書き方をしなくてはなりませんでした。

<!--
Because this would potentially break existing macros, the meaning of `:pat` did 
not change in Rust 1.53.0 to include `|`. Instead, that change happens in Rust 2021. 
In the new edition, the `:pat` fragment specifier *will* match `A | B`.
-->

既存のマクロを壊す可能性があるため、Rust 1.53.0 では `:pat` が `|` を含むことができるようには変更されませんでした。
代わりに、Rust 2021 で変更がなされました。
新しいエディションでは、`:pat` フラグメント指定子は `A | B` にマッチ*します*。

<!--
`$_:pat` fragments in Rust 2021 cannot be followed by an explicit `|`. Since there are times 
that one still wishes to match pattern fragments followed by a `|`, the fragment specified `:pat_param` 
has been added to retain the older behavior.
-->

Rust 2021 では、`$_:pat` フラグメントに `|` そのものが後続することはできません。
パターンフラグメントに `|` が後続するものにマッチさせたいような場合は、新しく追加された `:pat_param` が過去と同じ挙動を示すようになっています。

<!--
It's important to remember that editions are _per crate_, so the only relevant edition is the edition
of the crate where the macro is defined. The edition of the crate where the macro is used does not 
change how the macro works.
-->

ただし、エディションは<!-- -->_クレートごとに_<!-- -->設定されることに注意してください。
つまり、マクロが定義されているクレートのエディションだけが関係します。
マクロを使用する方のクレートのエディションは、マクロの挙動に影響しません。

<!--
## Migration 
-->

## 移行

<!--
A lint, `rust_2021_incompatible_or_patterns`, gets triggered whenever there is a use `$_:pat` which
will change meaning in Rust 2021. 
-->

`$_:pat` が使われている場所のうち、Rust 2021 で意味が変わるようなものに対しては、`rust_2021_incompatible_or_patterns` というリントが発生します。

<!--
You can automatically migrate your code to be Rust 2021 Edition compatible or ensure it is already compatible by
running:
-->

コードを自動的に Rust 2021 エディションに適合するよう自動移行するか、既に適合するものであることを確認するためには、以下のように実行すればよいです:

```sh
cargo fix --edition
```

<!--
If you have a macro which relies on `$_:pat` not matching the top level use of `|` in patterns, 
you'll need to change each occurrence of `$_:pat` to `$_:pat_param`.
-->

あなたのマクロが、`$_:pat` がトップレベルの `|` にマッチしないという挙動に依存している場合は、
`$_:pat` を `$_:pat_param` に書き換える必要があります。

<!--
For example:
-->

例えば以下のようになります。

<!--
```rust
macro_rules! my_macro { 
	($x:pat | $y:pat) => {
		// TODO: implementation
	} 
}

// This macro works in Rust 2018 since `$x:pat` does not match against `|`:
my_macro!(1 | 2);

// In Rust 2021 however, the `$_:pat` fragment matches `|` and is not allowed
// to be followed by a `|`. To make sure this macro still works in Rust 2021
// change the macro to the following:
macro_rules! my_macro { 
	($x:pat_param | $y:pat) => { // <- this line is different
		// TODO: implementation
	} 
}
```
-->

```rust
macro_rules! my_macro { 
	($x:pat | $y:pat) => {
		// TODO: 実装
	} 
}

// Rust 2018 では、`$x:pat` が `|` にマッチしないので、以下のマクロは正常に動きます:
my_macro!(1 | 2);

// 一方 Rust 2021 では、`$_:pat` フラグメントは `|` にもマッチし、
// `|` が後続してはいけなくなりました。
// Rust 2021 でもマクロが動作するためには、マクロを以下のように変更しなくてはなりません:
macro_rules! my_macro { 
	($x:pat_param | $y:pat) => { // <- この行を変えた
		// TODO: 実装
	} 
}
```
