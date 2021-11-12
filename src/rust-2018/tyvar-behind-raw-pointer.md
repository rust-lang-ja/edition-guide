<!--
# Method dispatch for raw pointers to inference variables
-->

# 推論変数への生ポインタに対するメソッドのディスパッチ

<!--
## Summary
-->

## 概要

<!--
- The [`tyvar_behind_raw_pointer`][#46906] lint is now a hard error.
-->

- リント [`tyvar_behind_raw_pointer`][#46906] はハードエラー[^1]になりました。

[#46906]: https://github.com/rust-lang/rust/issues/46906

[^1]: (訳注) ハードエラーとは、`#[allow(...)]` などを使って、ユーザーが無効化することができないエラーのことです。
  例えば、ライフタイムに不整合があったとします。ユーザーが実際はそれが安全であると信じていても、そのようなコードをコンパイルすることは許されていません。 `#[allow(...)]` による上書きも不可能となっています。これがハードエラーです。
  一方、到達不能コード（たとえば関数の途中で `return` をしており、それ以降のコードは決して実行されないような状況）は、コンパイル自体は可能で、時には役に立つこともあるので、ユーザーが `#[allow(dead_code)]` と書くことで警告を抑制できます。これはハードエラーではありません。
  詳細は [rustc book の説明(英語)](https://rustc-dev-guide.rust-lang.org/diagnostics.html#lints-versus-fixed-diagnostics) もご参照ください。

<!--
## Details
-->

## 詳細

<!--
See Rust issue [#46906] for details.
-->

詳細は Rust のイシュー [#46906] を参照してください。

> *訳注*:
> 詳しく解説します。以下のプログラムをご覧ください。
>
> ```rust
> let s = libc::getenv(k.as_ptr()) as *const _;
> s.is_null()
> ```
>
> 1行目の [`libc::getenv`] は、`*mut c_char` 型の生ポインタを返す関数です。
> このポインタは `as` を使って `*const` ポインタに変換できますが、その際これが「何の型のポインタであるか」を `_` にて省略しています。
>
> 2行目では、`*const _` である `s` に対して `.is_null()` を呼び出しています。
> 任意の型 `T` について、プリミティブ型 `*const T` は [`is_null`] という関連メソッド[^2]を持つので、ここで呼び出されるのはこの関連メソッドです。
>
> 問題はこの後です。
> 現在は関連メソッドを呼べる型は[一部の型に限られて][associated-methods]いますが、
> 将来それを任意の型に拡張しようという[提案][arbitrary_self_types-tracking]が出ています。
> この新機能は "arbitrary self types" （self の型の任意化）と呼ばれます。
> しかし、これが導入されると困ったことが起きます。
> 
> 次のような構造体があったとしましょう:
> ```rust
> #![feature(arbitrary_self_types)]
> 
> struct MyType;
> 
> impl MyType {
>     // この関数が問題
>     fn is_null(self: *mut Self) -> bool {
>         println!("?");
>         true
>     }
> }
> ```
>
> すると、最初のプログラムの2行目 `s.is_null` はどうなるでしょうか？
> 変数 `s` はキャストによって `*const _` 、つまり「何かの型への生定数ポインタ」を意味していました。
> そして今や、 `is_null` として呼び出せる関数は2つあります。
> 1つは先程の `*const T` に対して実装された [`is_null`]、
> もう一つは今 `*const MyType` に対して実装された `is_null` です。
> つまり、関連メソッドの呼び出しに曖昧性が生じています。
>
> この問題の解決策は簡単です。キャスト後の型が何への英数ポインタであることを明示すればよいです:
>
> ```rust
> let s = libc::getenv(k.as_ptr()) as *const libc::c_char;
> s.is_null()
> ```
>
> こうすることで、`is_null` の候補は `*const T` だけになります。
> `libc::c_char` は他のクレートで定義された型ですので、
> この型に対して新しく関連メソッドが実装されることはなく、恒久的に曖昧性がなくなります。
>
> こうした理由から、 `*const _` や `*mut _` など、「未知の型への生ポインタ」に対して関連メソッドを呼び出すと、コンパイラがそれを検知するようになりました。
> 最初は警告リントとして導入されましたが、Rust 2018 エディションでハードエラーに格上げされました。これが、本ページで説明されている変更点です。

[^2]: アイテムにドット (`.`) でつなげて関数を呼び出す方法。 `s.is_null()` は `s` の関連メソッド `is_null(...)` を呼び出していることになります。

[`libc::getenv`]: https://docs.rs/libc/0.2.107/i686-pc-windows-msvc/libc/fn.getenv.html
[`is_null`]: https://doc.rust-lang.org/std/primitive.pointer.html#method.is_null-1
[associated-methods]: https://doc.rust-lang.org/reference/items/associated-items.html#methods
[arbitrary_self_types-tracking]: https://github.com/rust-lang/rust/issues/44874

> *訳注*:
> タイトルにある「推論変数」とは、英語では inference variable または existencial variable と呼ばれます。
> Rust には型をある程度明示しなくても自動的に決定する機能（型推論）があります。
> 型推論とは、非常に単純に説明すると、未知の型を変数、プログラム中から得られる手がかりを条件とみなした「型の連立方程式」を解く事に当たります。
> 推論変数とは、この方程式における変数のことです。
> 今回は `*const _`、つまり「未知の型 `_` に対する定数生ポインタ」が出てきています。この `_` が「推論変数」にあたります。
> 詳しくは、[rustc book の型推論の説明 (英語)](https://rustc-dev-guide.rust-lang.org/type-inference.html#a-note-on-terminology)もご参照ください。
