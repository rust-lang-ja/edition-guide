<!--
# Path and module system changes
-->

パスとモジュールシステムへの変更

<!--
![Minimum Rust version: 1.31](https://img.shields.io/badge/Minimum%20Rust%20Version-1.31-brightgreen.svg)
-->

![導入 Rust バージョン: 1.31](https://img.shields.io/badge/%E5%B0%8E%E5%85%A5%20Rust%20%E3%83%90%E3%83%BC%E3%82%B8%E3%83%A7%E3%83%B3-1.31-brightgreen.svg)

<!--
## Summary
-->

## 概要

<!--
- Paths in `use` declarations now work the same as other paths.
- Paths starting with `::` must now be followed with an external crate.
- Paths in `pub(in path)` visibility modifiers must now start with `crate`, `self`, or `super`.
-->

- `use` 宣言中のパスが、他のパスと同じように扱われるようになりました。
- `::` から始まるパスの直後には、外部クレートが指定されなければならなくなりました。
- `pub(in path)` という可視性修飾子中の<!-- TODO: in がわかりません -->パスは、`crate`, `self`, `super` のいずれかで始まらなくてはならなくなりました。

<!--
## Motivation
-->

## 動機

<!--
The module system is often one of the hardest things for people new to Rust. Everyone
has their own things that take time to master, of course, but there's a root
cause for why it's so confusing to many: while there are simple and
consistent rules defining the module system, their consequences can feel
inconsistent, counterintuitive and mysterious.
-->

Rust を習って間もない人にとって、モジュールシステムは最も難しいものであることが多いです。
もちろん、習得に時間がかかるものは誰にでもあります。
しかし、モジュールシステムが多くの人を混乱させる根本的原因は、モジュールシステムの定義そのものは単純で首尾一貫しているにも関わらず、その帰結が一貫性のない、非直感的な、不思議なものに感じられてしまうことにあります。

<!--
As such, the 2018 edition of Rust introduces a few new module system
features, but they end up *simplifying* the module system, to make it more
clear as to what is going on.
-->

ですので、Rust の 2018 エディションにおけるモジュールシステムの新機能は、モジュールシステムを*単純化*し、仕組みをよりわかりやくするために導入されました。

<!--
Here's a brief summary:
-->

簡単にまとめると、

<!--
* `extern crate` is no longer needed in 99% of circumstances.
* The `crate` keyword refers to the current crate.
* Paths may start with a crate name, even within submodules.
* Paths starting with `::` must reference an external crate.
* A `foo.rs` and `foo/` subdirectory may coexist; `mod.rs` is no longer needed
  when placing submodules in a subdirectory.
* Paths in `use` declarations work the same as other paths.
-->

* `extern crate` は九割九分の場面で必要なくなりました。
* `crate` キーワードは、自身のクレートを指します。
* サブモジュール内であっても、パスをクレート名から始めることができます。
* `::` から始まるパスは、常に外部クレートを指します。
* `foo.rs` と `foo/` サブディレクトリは共存できます。サブディレクトリにサブモジュールを置く場合でも、`mod.rs` は必要なくなりました。
* `use` 宣言におけるパスも、他の場所のパスと同じように書けます。

These may seem like arbitrary new rules when put this way, but the mental
model is now significantly simplified overall. Read on for more details!

こうして並べられると、一見これらの新しい規則はてんでバラバラですが、これにより、総じて今までよりはるかに「こうすればこう動くだろう」という直感が通じるようになりました。
詳しく説明していきましょう！

<!--
## More details
-->

## 詳しく説明

<!--
Let's talk about each new feature in turn.
-->

新機能を一つずつ取り上げていきます。

<!--
### No more `extern crate`
-->

### さようなら、`extern crate`

<!--
This one is quite straightforward: you no longer need to write `extern crate` to
import a crate into your project. Before:
-->

これは非常に単純な話です。
プロジェクトにクレートをインポートするために `extern crate` を書く必要はもはやありません。
今まで:

```rust,ignore
// Rust 2015

extern crate futures;

mod submodule {
    use futures::Future;
}
```

これから:

```rust,ignore
// Rust 2018

mod submodule {
    use futures::Future;
}
```

<!--
Now, to add a new crate to your project, you can add it to your `Cargo.toml`,
and then there is no step two. If you're not using Cargo, you already had to pass
`--extern` flags to give `rustc` the location of external crates, so you'd just
keep doing what you were doing there as well.
-->

今や、プロジェクトに新しくクレートを追加したかったら、`Cargo.toml` に追記して、それで終わりです。
Cargo を使用していない場合は、`rustc` に外部クレートの場所を `--extern` フラグを渡しているでしょうが、それをそのまま続けていればよいです。

<!--
> One small note here: `cargo fix` will not currently automate this change. We may
> have it do this for you in the future.
-->

> 一つ注意ですが、`cargo fix` は今の所この変更を自動では行いません。
> 将来、自動で変更がなされるようになるかもしれません。

<!--
#### An exception
-->

#### 例外

<!--
There's one exception to this rule, and that's the "sysroot" crates. These are the
crates distributed with Rust itself.
-->

このルールには一つだけ例外があります。それは、"sysroot" のクレートです。
これらのクレートは、Rust に同梱されています。

<!--
Usually these are only needed in very specialized situations. Starting in
1.41, `rustc` accepts the `--extern=CRATE_NAME` flag which automatically adds
the given crate name in a way similar to `extern crate`. Build tools may use
this to inject sysroot crates into the crate's prelude. Cargo does not have a
general way to express this, though it uses it for `proc_macro` crates.
-->

通常は、これらが必要なのは非常に特殊な状況です。
1.41 以降、`rustc` は `--extern=CRATE_NAME` フラグを受け付けるようになりました。
このフラグを指定すると、`extern crate` で指定するのと同じように、与えられたクレート名が自動的に追加されます。
ビルドツールは、このフラグを用いてクレートのプレリュードに sysroot のクレートを注入できます。
Cargo にはこれを表現する汎用的な方法はありませんが、`proc_macro` のクレートではこれが使用されます。

<!--
Some examples of needing to explicitly import sysroot crates are:
-->

例えば、以下のような場合には明示的に sysroot のクレートをインポートする必要があります:

* [`std`]: Usually this is not neccesary, because `std` is automatically
  imported unless the crate is marked with [`#![no_std]`][no_std].
* [`core`]: Usually this is not necessary, because `core` is automatically
  imported, unless the crate is marked with [`#![no_core]`][no_core]. For
  example, some of the internal crates used by the standard library itself
  need this.
* [`proc_macro`]: This is automatically imported by Cargo if it is a
  proc-macro crate starting in 1.42. `extern crate proc_macro;` would be
  needed if you want to support older releases, or if using another build tool
  that does not pass the appropriate `--extern` flags to `rustc`.
* [`alloc`]: Items in the `alloc` crate are usually accessed via re-exports in
  the `std` crate. If you are working with a `no_std` crate that supports
  allocation, then you may need to explicitly import `alloc`.
* [`test`]: This is only available on the [nightly channel], and is usually
  only used for the unstable benchmark support.

* [`std`]: 通常はは不要です。
  クレートに [`#![no_std]`][no_std] が指定されていない限り、`std` は自動的にインポートされるからです。
* [`core`]: 通常はは不要です。
  クレートに [`#![no_core]`][no_core] が指定されていない限り、`core` は自動的にインポートされるからです。
  例えば、標準ライブラリに使用されている一部の内部クレートではこれが必要です。
* [`proc_macro`]: 1.42 以降、proc-macro ではこれは自動的にインポートされます。
  それより古いリリースをサポートしたい場合か、Cargo 以外のビルドツールを使っていてそれが `rustc` に適切な `--extern` フラグを渡さない場合は、`extern crate proc_macro;` と書く必要があります。
* [`alloc`]: `alloc` クレート内のアイテムは、通常は `std` を通して公開されたものが使用されます。
  アロケーションをサポートする `no_std` なクレートにおいては、明示的に `alloc` をインポートすることが必要になります。
* [`test`]: これは [nightly チャンネル]でのみ使用可能で、まだ安定化されていないベンチマークのために主に使われます。

<!--
[`alloc`]: ../../alloc/index.html
[`core`]: ../../core/index.html
[`proc_macro`]: ../../proc_macro/index.html
[`std`]: ../../std/index.html
[`test`]: ../../test/index.html
[nightly channel]: ../../book/appendix-07-nightly-rust.html
[no_core]: https://github.com/rust-lang/rust/issues/29639
[no_std]: ../../reference/names/preludes.html#the-no_std-attribute
-->
[`alloc`]: https://doc.rust-lang.org/alloc/index.html
[`core`]: https://doc.rust-lang.org/core/index.html
[`proc_macro`]: https://doc.rust-lang.org/proc_macro/index.html
[`std`]: https://doc.rust-lang.org/std/index.html
[`test`]: https://doc.rust-lang.org/test/index.html
[nightly チャンネル]: https://doc.rust-jp.rs/book-ja/appendix-07-nightly-rust.html
[no_core]: https://github.com/rust-lang/rust/issues/29639
[no_std]: https://doc.rust-lang.org/reference/names/preludes.html#the-no_std-attribute

#### Macros

One other use for `extern crate` was to import macros; that's no longer needed.
Macros may be imported with `use` like any other item. For example, the
following use of `extern crate`:

```rust,ignore
#[macro_use]
extern crate bar;

fn main() {
    baz!();
}
```

Can be changed to something like the following:

```rust,ignore
use bar::baz;

fn main() {
    baz!();
}
```

#### Renaming crates

If you've been using `as` to rename your crate like this:

```rust,ignore
extern crate futures as f;

use f::Future;
```

then removing the `extern crate` line on its own won't work. You'll need to do this:

```rust,ignore
use futures as f;

use self::f::Future;
```

This change will need to happen in any module that uses `f`.

### The `crate` keyword refers to the current crate

In `use` declarations and in other code, you can refer to the root of the
current crate with the `crate::` prefix. For instance, `crate::foo::bar` will
always refer to the name `bar` inside the module `foo`, from anywhere else in
the same crate.

The prefix `::` previously referred to either the crate root or an external
crate; it now unambiguously refers to an external crate. For instance,
`::foo::bar` always refers to the name `bar` inside the external crate `foo`.

### Extern crate paths

Previously, using an external crate in a module without a `use` import
required a leading `::` on the path.

```rust,ignore
// Rust 2015

extern crate chrono;

fn foo() {
    // this works in the crate root
    let x = chrono::Utc::now();
}

mod submodule {
    fn function() {
        // but in a submodule it requires a leading :: if not imported with `use`
        let x = ::chrono::Utc::now();
    }
}
```

Now, extern crate names are in scope in the entire crate, including
submodules.

```rust,ignore
// Rust 2018

fn foo() {
    // this works in the crate root
    let x = chrono::Utc::now();
}

mod submodule {
    fn function() {
        // crates may be referenced directly, even in submodules
        let x = chrono::Utc::now();
    }
}
```

### No more `mod.rs`

In Rust 2015, if you have a submodule:

```rust,ignore
// This `mod` declaration looks for the `foo` module in
// `foo.rs` or `foo/mod.rs`.
mod foo;
```

It can live in `foo.rs` or `foo/mod.rs`. If it has submodules of its own, it
*must* be `foo/mod.rs`. So a `bar` submodule of `foo` would live at
`foo/bar.rs`.

In Rust 2018 the restriction that a module with submodules must be named
`mod.rs` is lifted. `foo.rs` can just be `foo.rs`,
and the submodule is still `foo/bar.rs`. This eliminates the special
name, and if you have a bunch of files open in your editor, you can clearly
see their names, instead of having a bunch of tabs named `mod.rs`.

<table>
  <thead>
    <tr>
      <th>Rust 2015</th>
      <th>Rust 2018</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>
<pre>
.
├── lib.rs
└── foo/
    ├── mod.rs
    └── bar.rs
</pre>
    </td>
    <td>
<pre>
.
├── lib.rs
├── foo.rs
└── foo/
    └── bar.rs
</pre>
      </td>
    </tr>
  </tbody>
</table>

### `use` paths

![Minimum Rust version: 1.32](https://img.shields.io/badge/Minimum%20Rust%20Version-1.32-brightgreen.svg)

Rust 2018 simplifies and unifies path handling compared to Rust 2015. In Rust
2015, paths work differently in `use` declarations than they do elsewhere. In
particular, paths in `use` declarations would always start from the crate
root, while paths in other code implicitly started from the current scope.
Those differences didn't have any effect in the top-level module, which meant
that everything would seem straightforward until working on a project large
enough to have submodules.

In Rust 2018, paths in `use` declarations and in other code work the same way,
both in the top-level module and in any submodule. You can use a relative path
from the current scope, a path starting from an external crate name, or a path
starting with `crate`, `super`, or `self`.

Code that looked like this:

```rust,ignore
// Rust 2015

extern crate futures;

use futures::Future;

mod foo {
    pub struct Bar;
}

use foo::Bar;

fn my_poll() -> futures::Poll { ... }

enum SomeEnum {
    V1(usize),
    V2(String),
}

fn func() {
    let five = std::sync::Arc::new(5);
    use SomeEnum::*;
    match ... {
        V1(i) => { ... }
        V2(s) => { ... }
    }
}
```

will look exactly the same in Rust 2018, except that you can delete the `extern
crate` line:

```rust,ignore
// Rust 2018

use futures::Future;

mod foo {
    pub struct Bar;
}

use foo::Bar;

fn my_poll() -> futures::Poll { ... }

enum SomeEnum {
    V1(usize),
    V2(String),
}

fn func() {
    let five = std::sync::Arc::new(5);
    use SomeEnum::*;
    match ... {
        V1(i) => { ... }
        V2(s) => { ... }
    }
}
```

The same code will also work completely unmodified in a submodule:

```rust,ignore
// Rust 2018

mod submodule {
    use futures::Future;

    mod foo {
        pub struct Bar;
    }

    use foo::Bar;

    fn my_poll() -> futures::Poll { ... }

    enum SomeEnum {
        V1(usize),
        V2(String),
    }

    fn func() {
        let five = std::sync::Arc::new(5);
        use SomeEnum::*;
        match ... {
            V1(i) => { ... }
            V2(s) => { ... }
        }
    }
}
```

This makes it easy to move code around in a project, and avoids introducing
additional complexity to multi-module projects.

If a path is ambiguous, such as if you have an external crate and a local
module or item with the same name, you'll get an error, and you'll need to
either rename one of the conflicting names or explicitly disambiguate the path.
To explicitly disambiguate a path, use `::name` for an external crate name, or
`self::name` for a local module or item.
