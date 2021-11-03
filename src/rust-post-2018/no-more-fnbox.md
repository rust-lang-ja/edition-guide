<!--
# No more FnBox
-->
# FnBoxは不要に

![Minimum Rust version: 1.35](https://img.shields.io/badge/Minimum%20Rust%20Version-1.35-brightgreen.svg)

<!--
The book used to have this code in Chapter 20, section 2:
-->
かつて、この本の20章2節には以下のコードがありました。

```rust
trait FnBox {
    fn call_box(self: Box<Self>);
}

impl<F: FnOnce()> FnBox for F {
    fn call_box(self: Box<F>) {
        (*self)()
    }
}

type Job = Box<dyn FnBox + Send + 'static>;
```

<!--
Here, we define a new trait called `FnBox`, and then implement it for all
`FnOnce` closures. All the implementation does is call the closure. These
sorts of hacks were needed because a `Box<dyn FnOnce>` didn't implement
`FnOnce`. This was true for all three posibilities:
-->
ここで、`FnBox`という新しいトレイトを定義し、全ての`FnOnce`クロージャに対して実装をしています。
そして全ての実装がクロージャを呼び出します。
このハックは、`Box<dyn FnOnce>` が`FnOnce`を実装していなかったので必要でした。
そしてこれは以下の3つのパターンで必要でした。

<!--
* `Box<dyn Fn>` and `Fn`
* `Box<dyn FnMut>` and `FnMut`
* `Box<dyn FnOnce>` and `FnOnce`
-->
* `Box<dyn Fn>` と `Fn`
* `Box<dyn FnMut>` と `FnMut`
* `Box<dyn FnOnce>` と `FnOnce`

<!--
However, as of Rust 1.35, these traits are implemented for these types,
and so the `FnBox` trick is no longer required. In the latest version of
the book, the `Job` type looks like this:
-->
しかし、Rust 1.35からこれらのトレイトがこれらの型に実装されたので、`FnBox`トリックは必要なくなりました。
この本の最新版では`Job`型はこのようになります。

```rust
type Job = Box<dyn FnOnce() + Send + 'static>;
```

<!--
No need for all that other code.
-->
これ以外のコードは不要です。
