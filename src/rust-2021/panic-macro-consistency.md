<!--
# Panic macro consistency
-->

# panic マクロの一貫性

<!--
## Summary
-->

## 概要

<!--
- `panic!(..)` now always uses `format_args!(..)`, just like `println!()`.
- `panic!("{")` is no longer accepted, without escaping the `{` as `{{`.
- `panic!(x)` is no longer accepted if `x` is not a string literal.
  - Use `std::panic::panic_any(x)` to panic with a non-string payload.
  - Or use `panic!("{}", x)` to use `x`'s `Display` implementation.
- The same applies to `assert!(expr, ..)`.
-->

- `panic!(..)` では常に `format_args!(..)` が使われるようになりました。つまり、`println!()` と同じ書き方をすることになります。
- `panic!("{")` と書くことはできなくなりました。`{` を `{{` とエスケープしなくてはなりません。
- `x` が文字列リテラルでないときに、 `panic!(x)` と書くことはできなくなりました。
  - 文字列でないペイロード付きでパニックしたい場合、 `std::panic::panic_any(x)` を使うようにしてください。
  - もしくは、`x` の `Display` 実装を用いて、`panic!("{}", x)` と書いてください。
- `assert!(expr, ..)` に関しても同様です。

<!--
## Details
-->

## 詳細

<!--
The `panic!()` macro is one of Rust's most well known macros.
However, it has [some subtle surprises](https://github.com/rust-lang/rfcs/blob/master/text/3007-panic-plan.md)
that we can't just change due to backwards compatibility.
-->

`panic!()` は、Rust で最もよく知られたマクロの一つです。
しかし、このマクロには[いくぶん非直感的な挙動](https://github.com/rust-lang/rfcs/blob/master/text/3007-panic-plan.md)がありましたが、
今までは後方互換性の問題から修正できませんでした。

```rust,ignore
// Rust 2018
panic!("{}", 1); // Ok, panics with the message "1"
                 // OK。 "1" というメッセージと共にパニックする
panic!("{}"); // Ok, panics with the message "{}"
              // OK。 "{}" というメッセージと共にパニックする
```

<!--
The `panic!()` macro only uses string formatting when it's invoked with more than one argument.
When invoked with a single argument, it doesn't even look at that argument.
-->

`panic!()` マクロは、2つ以上の引数が渡されたときだけ、フォーマット文字列を使用します。
引数が1つのときは、引数の中身に見向きもしません。

```rust,ignore
// Rust 2018
let a = "{";
println!(a); // Error: First argument must be a format string literal
             // エラー: 第一引数は文字列リテラルでなくてはならない
panic!(a); // Ok: The panic macro doesn't care
           // OK: panicマクロは気にしない
```

<!--
It even accepts non-strings such as `panic!(123)`, which is uncommon and rarely useful since it
produces a surprisingly unhelpful message: `panicked at 'Box<Any>'`.
-->

その上、このマクロは `panic!(123)` のように文字列でない引数を渡すこともできました。
このような使い方は稀で、ほとんど役に立ちません。
というのも、この呼び出しで出力されるメッセージはあきれるほど役に立たないからです: `panicked at 'Box<Any>'` (訳: `'Box<Any>` でパニックしました)。

<!--
This will especially be a problem once
[implicit format arguments](https://rust-lang.github.io/rfcs/2795-format-args-implicit-identifiers.html)
are stabilized.
That feature will make `println!("hello {name}")` a short-hand for `println!("hello {}", name)`.
However, `panic!("hello {name}")` would not work as expected,
since `panic!()` doesn't process a single argument as format string.
-->

これで特に困るのは、[暗黙のフォーマット引数](https://rust-lang.github.io/rfcs/2795-format-args-implicit-identifiers.html)が安定化されたときです。
この機能により、`println!("hello {}", name)` の代わりに `println!("hello {name}")` と略記できるようになります。
しかし、`panic!("hello {name}")` は期待される挙動を示しません。
なぜなら、`panic!()` は引数が1つだけ与えられたときにそれをフォーマット文字列として扱わないからです。

<!--
To avoid that confusing situation, Rust 2021 features a more consistent `panic!()` macro.
The new `panic!()` macro will no longer accept arbitrary expressions as the only argument.
It will, just like `println!()`, always process the first argument as format string.
Since `panic!()` will no longer accept arbitrary payloads,
[`panic_any()`](https://doc.rust-lang.org/stable/std/panic/fn.panic_any.html)
will be the only way to panic with something other than a formatted string.
-->

この紛らわしい状況を解決するために、Rust 2021 の `panic!()` マクロはより一貫したものになりました。
新しい `panic!()` マクロは、単一引数として任意の式を受け付けることがなくなりました。
代わりに、`println!()` と同様に、常に最初の引数をフォーマット文字列として処理するようになりました。
`panic!()` マクロが任意のペイロードを受け付けるわけではなくなったので、
フォーマット文字列以外のペイロードと共にパニックさせる唯一の方法は、
[`panic_any()`](https://doc.rust-lang.org/stable/std/panic/fn.panic_any.html)を使うことだけになりました。

```rust,ignore
// Rust 2021
panic!("{}", 1); // Ok, panics with the message "1"
                 // Ok。"1" というメッセージと共にパニックする
panic!("{}"); // Error, missing argument
              // エラー。引数が足りない
panic!(a); // Error, must be a string literal
           // エラー。文字列リテラルでないといけない
```

<!--
In addition, `core::panic!()` and `std::panic!()` will be identical in Rust 2021.
Currently, there are some historical differences between those two,
which can be noticeable when switching `#![no_std]` on or off.
-->

さらに、`core::panic!()` と `std::panic!()` は Rust 2021 で同一のものになります。
現在は、これらの間には歴史的な違いがあり、
`#![no_std]` のオンオフを切り替えることで見て取ることができます。

<!--
## Migration
-->

## 移行

<!--
A lint, `non_fmt_panics`, gets triggered whenever there is some call to `panic` that uses some 
deprecated behavior that will error in Rust 2021. The `non_fmt_panics` lint has already been a warning 
by default on all editions since the 1.50 release (with several enhancements made in later releases). 
If your code is already warning free, then it should already be ready to go for Rust 2021!
-->

`panic` への呼び出しのうち、非推奨の挙動を使用していて Rust 2021 ではエラーになる場所に対して、`non_fmt_panics` というリントが発生します。
Rust 1.50 以降、`non_fmt_panics` リントはすでにデフォルトで警告として発出されています（後のバージョンではさらにいくつかの機能追加が行われました）。
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
Should you choose or need to manually migrate, you'll need to update all panic invocations to either use the same 
formatting as `println` or use `std::panic::panic_any` to panic with non-string data.
-->

手動で移行することを選んだり、そうする必要がある場合には、各 panic の呼び出しについて、 `println` と同様のフォーマットに書き換えるか、`std::panic::panic_any` を用いて非文字列型のデータと共にパニックさせるかを選ぶ必要があります。

<!--
For example, in the case of `panic!(MyStruct)`, you'll need to convert to using `std::panic::panic_any` (note
that this is a function not a macro): `std::panic::panic_any(MyStruct)`.
-->

例えば、`panic!(MyStruct)` の場合、`std::panic::panic_any` （これはマクロでなく関数であることに注意）を使うよう書き換えて、`std::panic::panic_any(MyStruct)` とすればよいです。

<!--
In the case of panic messages that include curly braces but the wrong number of arguments (e.g., `panic!("Some curlies: {}")`), 
you can panic with the string literal by either using the same syntax as `println!` (i.e., `panic!("{}", "Some curlies: {}")`) 
or by escaping the curly braces (i.e., `panic!("Some curlies: {{}}")`).
-->

パニックのメッセージに波括弧が含まれているのに、引数の個数が一致しない場合は（例: `panic!("Some curlies: {}")`）、
`println!` と同じ構文を使うか（つまり `panic!("{}", "Some curlies: {}")` とするか）、波括弧をエスケープすれば（つまり `panic!("Some curlies: {{}}")` とすれば）、
その文字列リテラルを用いてパニックすることができます。
