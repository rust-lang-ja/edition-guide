<!--
# Tail expression temporary scope
-->

# 末尾式の一時スコープ

<!--
## Summary
-->

## 概要

<!--
- Temporary values generated in evaluation of the tail expression of a [function] or closure body, or a [block] may now be dropped before local variables, and are sometimes not extended to the next larger temporary scope.
-->

- [関数]・クロージャ本体・[ブロック]の末尾式の評価で生成される一時値は、ローカル変数より先にドロップされるようになり、外側の一時スコープまで生き延びない場合があります。

<!--
[function]: ../../reference/items/functions.html
[block]: ../../reference/expressions/block-expr.html
-->

[関数]: https://doc.rust-lang.org/reference/items/functions.html
[ブロック]: https://doc.rust-lang.org/reference/expressions/block-expr.html

<!--
## Details
-->

## 詳細

<!--
The 2024 Edition changes the drop order of [temporary values] in tail expressions. It often comes as a surprise that, before the 2024 Edition, temporary values in tail expressions can live longer than the block itself, and are dropped later than the local variable bindings, as in the following example:
-->

2024 エディションから、[末尾式] [^1]のドロップ順序が変わります。
2024 エディションより前の直感的でない挙動として、以下のように、末尾式内の一時値がブロックそのものより長生きして、ローカル変数束縛よりも長生きするというものがありました。

<!--
[temporary values]: ../../reference/expressions.html#temporaries
-->

[末尾式]: https://doc.rust-lang.org/reference/expressions.html#temporaries

<!--
```rust,edition2021,compile_fail,E0597
// Before 2024
# use std::cell::RefCell;
fn f() -> usize {
    let c = RefCell::new("..");
    c.borrow().len() // error[E0597]: `c` does not live long enough
}
```
-->

```rust,edition2021,compile_fail,E0597
// 2024 より前
# use std::cell::RefCell;
fn f() -> usize {
    let c = RefCell::new("..");
    c.borrow().len() // error[E0597]: `c` does not live long enough
                     // 訳: [E0597]: `c` は十分に長生きしません
}
```

[^1] 訳注: 関数・クロージャ・ブロックの `{ }` の末尾にある、セミコロン `;` を伴わない式のこと。

<!--
This yields the following error with the 2021 Edition:
-->

2021 エディションでは以下のようなエラーになりました。

```text
error[E0597]: `c` does not live long enough
(訳: エラー[E0597]: `c` は十分に長生きしません)
 --> src/lib.rs:4:5
  |
3 |     let c = RefCell::new("..");
  |         - binding `c` declared here
              (訳: 束縛 `c` はここで定義されています)
4 |     c.borrow().len() // error[E0597]: `c` does not live long enough
  |     ^---------
  |     |
  |     borrowed value does not live long enough
        (訳: ここで借用される値が十分に長生きしません)
  |     a temporary with access to the borrow is created here ...
        (訳: 値を借用する一時値がここで生成されていますが、…)
5 | }
  | -
  | |
  | `c` dropped here while still borrowed
    (訳: `c` が借用中にここでドロップされています)
  | ... and the borrow might be used here, when that temporary is dropped and runs the destructor for type `Ref<'_, &str>`
  |
    (訳: …ここでその一時値がドロップされて `Ref<'_, &str>` 型のデストラクタが実行されるとき、借用が使用中である可能性があります)
  = note: the temporary is part of an expression at the end of a block;
  (訳: 注: この一時値は、ブロック末尾の式の一部です。)
          consider forcing this temporary to be dropped sooner, before the block's local variables are dropped
          (訳: 当該の一時値がドロップされるタイミングが、ブロック内のローカル変数がドロップされる前になるようにするとよいかもしれません)
help: for example, you could save the expression's value in a new local variable `x` and then make `x` be the expression at the end of the block
(訳: ヘルプ: 例えば、式の値を一旦新しいローカル変数 `x` に格納し、それをブロック末尾の式とするとよいです)
          
  |
4 |     let x = c.borrow().len(); x // error[E0597]: `c` does not live long enough
  |     +++++++                 +++

For more information about this error, try `rustc --explain E0597`.
(訳: 詳細に関しては `rustc --explain E0597` をご参照ください。)
```

<!--
In 2021 the local variable `c` is dropped before the temporary created by `c.borrow()`. The 2024 Edition changes this so that the temporary value `c.borrow()` is dropped first, followed by dropping the local variable `c`, allowing the code to compile as expected.
-->

2021 では、ローカル変数 `c` よりも `c.borrow()` で作られた一時値の方が先にドロップされます。
2024 エディションでこの挙動は変わり、一時値 `c.borrow()` が先にドロップされてからローカル変数 `c` がドロップされるようになり、コードが期待通りコンパイルされるよになりました。

<!--
### Temporary scope may be narrowed
-->

### 一時スコープ縮小の可能性

<!--
When a temporary is created in order to evaluate an expression, the temporary is dropped based on the [temporary scope rules]. Those rules define how long the temporary will be kept alive. Before 2024, temporaries from tail expressions of a block would be extended outside of the block to the next temporary scope boundary. In many cases this would be the end of a statement or function body. In 2024, the temporaries of the tail expression may now be dropped immediately at the end of the block (before any local variables in the block).
-->

式の評価のために一時値が作られるとき、一時値は[一時スコープのルール]に則ってドロップされます。
このルールは一時値の生存期間を規定するものです。
2024 より前では、ブロックの末尾式における一時値は次のスコープ境界まで、もっぱら文・関数の末尾まで生存していました。
2024 以降では、末尾式内の一時値はブロック終了時に即座に（ブロック内のローカル変数より前に）ドロップされます。

<!--
This narrowing of the temporary scope may cause programs to fail to compile in 2024. For example:
-->

このように一時スコープが縮小することにより、以下のように 2024 でコンパイルが通らなくなる場合があります。

<!--
```rust,edition2024,E0716,compile_fail
// This example works in 2021, but fails to compile in 2024.
fn main() {
    let x = { &String::from("1234") }.len();
}
```
-->

```rust,edition2024,E0716,compile_fail
// 2021 ではコンパイルが通るが 2024 だと通らない
fn main() {
    let x = { &String::from("1234") }.len();
}
```

<!--
In this example, in 2021, the temporary `String` is extended outside of the block, past the call to `len()`, and is dropped at the end of the statement. In 2024, it is dropped immediately at the end of the block, causing a compile error about the temporary being dropped while borrowed.
-->

このコードだと、2021 では一時値 `String` はブロックの外側と `len()` の呼び出しを超えて生存し、文の末尾でドロップされます。2024 ではブロック末尾で即座にドロップされるため、一時値が借用中にドロップされる旨のコンパイルエラーが出ます。

<!--
The solution for these kinds of situations is to lift the block expression out to a local variable so that the temporary lives long enough:
-->

対処法は、ブロック式をローカル変数に格上げして、一時値が十分長生きするようにすることです。

```rust,edition2024
fn main() {
    let s = { &String::from("1234") };
    let x = s.len();
}
```

<!--
This particular example takes advantage of [temporary lifetime extension]. Temporary lifetime extension is a set of specific rules which allow temporaries to live longer than they normally would. Because the `String` temporary is behind a reference, the `String` temporary is extended long enough for the next statement to call `len()` on it.
-->


このコードでは特に、特定の状況下で一時値を本来よりも延命させる、[一時値の延命]ルールを活用しています。
一時値の `String` が、参照 `&` 下にあるために延命され、次の文で `len()` が呼び出せるようになっています。

<!--
See the [`if let` temporary scope] chapter for a similar change made to temporary scopes of `if let` expressions.
-->

[`if let` の一時スコープ]の節では、`if let` 式の一時スコープによる同様の変更について説明しています。

<!--
[`if let` temporary scope]: temporary-if-let-scope.md
[temporary scope rules]: ../../reference/destructors.html#temporary-scopes
[temporary lifetime extension]: ../../reference/destructors.html#temporary-lifetime-extension
-->

[`if let` temporary scope]: temporary-if-let-scope.md
[temporary scope rules]: https://doc.rust-lang.org/reference/destructors.html#temporary-scopes
[temporary lifetime extension]: https://doc.rust-lang.org/reference/destructors.html#temporary-lifetime-extension

<!--
## Migration
-->

## 移行

<!--
Unfortunately, there are no semantics-preserving rewrites to shorten the lifetime for temporary values in tail expressions[^RFC3606]. The [`tail_expr_drop_order`] lint detects if a temporary value with a custom, non-trivial `Drop` destructor is generated in a tail expression. Warnings from this lint will appear when running `cargo fix --edition`, but will otherwise not automatically make any changes. It is recommended to manually inspect the warnings and determine whether or not you need to make any adjustments.
-->

残念ながら、コードの意味合いを変えずに末尾式中の一時値の生存期間を短くする書き換え方はありません[^RFC3606]。
[`tail_expr_drop_order`]リントは、末尾式中の一時値が独自の非自明な `Drop` デストラクタをもつことを検出します。
`cargo fix --edition` を実行すると警告がでますが、特段の書き換えはなされません。
プログラマ自身が警告内容を確認し、変更が必要かどうか判断する必要があります。

<!--
If you want to manually inspect these warnings without performing the edition migration, you can enable the lint with:
-->

エディション移行を行わずに警告箇所を確認したい場合は、以下のように記述するとリントを有効化できます。

<!--
```rust
// Add this to the root of your crate to do a manual migration.
#![warn(tail_expr_drop_order)]
```
-->

```rust
// クレートのトップレベルに以下を追加すると手動移行できる
#![warn(tail_expr_drop_order)]
```

<!--
[^RFC3606]: Details are documented at [RFC 3606](https://github.com/rust-lang/rfcs/pull/3606)
-->

[^RFC3606]: 詳細は [RFC 3606](https://github.com/rust-lang/rfcs/pull/3606) に説明されています。

<!--
[`tail_expr_drop_order`]: ../../rustc/lints/listing/allowed-by-default.html#tail-expr-drop-order
-->

[`tail_expr_drop_order`]: https://doc.rust-lang.org/rustc/lints/listing/allowed-by-default.html#tail-expr-drop-order
