<!--
# Reserved syntax
-->

# 構文の予約

<!--
## Summary
-->

## 概要

<!--
- `any_identifier#`, `any_identifier"..."`, `any_identifier'...'`, and `'any_identifier#` are now reserved syntax, and no longer tokenize.
- This is mostly relevant to macros. E.g. `quote!{ #a#b }` is no longer accepted.
- It doesn't treat keywords specially, so e.g. `match"..." {}` is no longer accepted.
- Insert whitespace between the identifier and the subsequent `#`, `"`, or `'`
  to avoid errors.
- Edition migrations will help you insert whitespace in such cases.
-->

- `shikibetsushi#`, `shikibetsushi"..."`, `shikibetsushi'...'`, `'any_identifier#` の4つの構文が新たに予約され、トークン分割されなくなりました。
- 主に影響を受けるのはマクロです。例えば、`quote!{ #a#b }` と書くことはできなくなりました。
- キーワードが特別扱いされることもないので、例えば `match"..." {}` と書くこともできなくなりました。
- 識別子と後続の `#`, `"`, `'` の間に空白文字を挿入することで、エラーを回避できます。
- エディション移行ツールは、必要な場所に空白を挿入してくれます。

<!--
## Details
-->

## 詳細

<!--
To make space for new syntax in the future,
we've decided to reserve syntax for prefixed identifiers, literals, and lifetimes:
`prefix#identifier`, `prefix"string"`, `prefix'c'`, `prefix#123`, and `'prefix#`,
where `prefix` can be any identifier.
(Except those prefixes that already have a meaning, such as `b'...'` (byte
chars) and `r"..."` (raw strings).)
-->

私達は、将来新しい構文を導入する余地を残すため、接頭辞付きの識別子・リテラル・ライフタイムの構文を予約することにしました。
予約されたのは、任意の識別子 `prefix` を用いて `prefix#identifier`, `prefix"string"`, `prefix'c'`, `prefix#123`, `'prefix#` のいずれかの形で書かれるものです。
(ただし、`b'...'`（バイト文字）や`r"..."`（生文字列）のように、すでに意味が割り当てられているものを除きます。）

<!--
This provides syntax we can expand into in the future without requiring an
edition boundary. We may use this for temporary syntax until the next edition,
or for permanent syntax if appropriate.
-->

これにより、将来エディションをまたくごとなく構文を拡張できるようになります。
これを、次のエディションまでの一時的な構文のために使ったり、もし適切なら、恒久的な構文のために使ったりするでしょう。

<!--
Without an edition, this would be a breaking change, since macros can currently
accept syntax such as `hello"world"`, which they will see as two separate
tokens: `hello` and `"world"`. The (automatic) fix is simple though: just
insert a space: `hello "world"`. Likewise, `prefix#ident` should become
`prefix #ident`. Edition migrations will help with this fix.
-->

エディションの区切りがないと、これは破壊的変更に当たります。
なぜなら、現在のマクロは、例えば `hello"world"` という構文を、 `hello` と `"world"` の2つのトークンとして受け入れるからです。
もっとも、（自動）修正はシンプルで、`hello "world"` のように空白を入れるだけです。
同様に、`prefix#ident` は `prefix #ident` とする必要があります。
エディション移行ツールは、そのように修正してくれます。

<!--
Other than turning these into a tokenization error,
[the RFC][10] does not attach a meaning to any prefix yet.
Assigning meaning to specific prefixes is left to future proposals,
which will now&mdash;thanks to reserving these prefixes&mdash;not be breaking changes.
-->

[これが提案された RFC][10] は、このような書き方をトークン分割エラーにすると決めているだけで、
特定の接頭辞に意味を持たせることはまだしていません。
接頭辞に何らかの役割を割り当てるのは、将来の提案にその余地が残されています。
接頭辞が予約されたおかげで、今後は破壊的変更なく新しい構文を導入できます。

<!--
Some new prefixes you might potentially see in the future (though we haven't
committed to any of them yet):
-->

例えば、以下のような接頭辞が使えるようになるかもしれません（ただし、いずれもまだ提案が固まったわけではありません）:

<!--
- `k#keyword` to allow writing keywords that don't exist yet in the current edition.
  For example, while `async` is not a keyword in edition 2015,
  this prefix would've allowed us to accept `k#async` in edition 2015
  without having to wait for edition 2018 to reserve `async` as a keyword.
-->

- `k#keyword` で、現在のエディションにまだ導入されていないキーワードを書けるようにする。
  たとえば、2015 エディションでは `async` はキーワードではありませんが、
  このような接頭辞が使えたのならば、2018 エディションで `async` が予約語になるのを待たずに、2015 エディションでも `k#async` が使えたということになります。

<!--
- `f""` as a short-hand for a format string.
  For example, `f"hello {name}"` as a short-hand for the equivalent `format!()` invocation.
-->

- `f""` で、フォーマット文字列の略記とする。
  例えば、`f"hello {name}"` と書いたら、それと等価な `format!()` の呼び出しと見なす。

<!--
- `s""` for `String` literals.
-->

- `s""` で `String` リテラルを表す。

[10]: https://github.com/rust-lang/rfcs/pull/3101


<!--
## Migration 
-->

## 移行

<!--
As a part of the 2021 edition a migration lint, [`rust_2021_prefixes_incompatible_syntax`], has been added in order to aid in automatic migration of Rust 2018 codebases to Rust 2021.
-->

Rust 2018 のコードベースから Rust 2021 への自動移行の支援のため、2021 エディションには、移行用のリント[`rust_2021_prefixes_incompatible_syntax`] が追加されています。

<!--
In order to migrate your code to be Rust 2021 Edition compatible, run:
-->

コードを Rust 2021 エディションに適合させるためには、次のように実行します。

```sh
cargo fix --edition
```

<!--
Should you want or need to manually migrate your code, migration is fairly straight-forward.
-->

コード移行を手で行いたいか、行う必要があっても、移行は非常に簡単です。

<!--
Let's say you have a macro that is defined like so:
-->

例えば、次のように定義されたマクロがあったとしましょう：

```rust
macro_rules! my_macro {
    ($a:tt $b:tt) => {};
}
```

<!--
In Rust 2015 and 2018 it's legal for this macro to be called like so with no space between the first token tree and the second:
-->

Rust 2015 と 2018 では、以下のように、1つ目と2つ目のトークンの間に空白を入れることなくマクロを呼び出しても問題ありませんでした:

```rust,ignore
my_macro!(z"hey");
```

<!--
This `z` prefix is no longer allowed in Rust 2021, so in order to call this macro, you must add a space after the prefix like so:
-->

Rust 2021 では `z` という接頭辞は許されないので、このマクロを呼び出すためには、以下のように接頭辞の後にスペースを入れる必要があります:

```rust,ignore
my_macro!(z "hey");
```

<!--
[`rust_2021_prefixes_incompatible_syntax`]: ../../rustc/lints/listing/allowed-by-default.html#rust-2021-prefixes-incompatible-syntax
-->

[`rust_2021_prefixes_incompatible_syntax`]: https://doc.rust-lang.org/rustc/lints/listing/allowed-by-default.html#rust-2021-prefixes-incompatible-syntax
