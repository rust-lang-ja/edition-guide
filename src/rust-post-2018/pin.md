<!--
# Pinning
-->
# ピン留め

![Minimum Rust version: 1.33](https://img.shields.io/badge/Minimum%20Rust%20Version-1.33-brightgreen.svg)

<!--
Rust 1.33 introduced a new concept, implemented as two types: 
-->
Rust 1.33で新しい概念が導入され、二つの型で実装されました。

<!--
* [`Pin<P>`](https://doc.rust-lang.org/std/pin/struct.Pin.html), a wrapper
  around a kind of pointer which makes that pointer "pin" its value in place,
  preventing the value referenced by that pointer from being moved.
* [`Unpin`](https://doc.rust-lang.org/std/marker/trait.Unpin.html), types that
  are safe to be moved, even if they're pinned.
  -->
* [`Pin<P>`](https://doc.rust-lang.org/std/pin/struct.Pin.html) ポインタ的なモノを包むラッパで、その値が動かないように「ピン留め」します。
それにより、そのポインタにより参照されている値がムーブされるのを防ぎます。
* [`Unpin`](https://doc.rust-lang.org/std/marker/trait.Unpin.html) ピン留めされていても安全にムーブできる型です。

<!--
Most users will not interact with pinning directly, and so we won't explain
more here. For the details, see the [documentation for
`std::pin`](https://doc.rust-lang.org/std/pin/index.html).
-->
ほとんどのユーザはピン留めを直接扱うことは無いと思うので、これ以上の説明は控えます。
より詳細を知りたければ[`std::pin`のドキュメンテーション](https://doc.rust-lang.org/std/pin/index.html)を見てください。

<!--
What *is* useful to know about pinning is that it's a pre-requisite for
`async`/`await`. Folks who write async libraries may need to learn about
pinning, but folks using them generally shouldn't need to interact with this
feature at all.
-->
ピン留めは`async` / `await`の前提条件になっているということは知っておくと良いでしょう。
非同期のライブラリを作成する人はピン留めを学んでおく必要があるかも知れませんが、そのライブラリを使うだけの人はこの機能に直接触れることは無いでしょう。

