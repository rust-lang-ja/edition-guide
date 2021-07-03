<!--
# No jemalloc by default
-->
# デフォルトでjemallocを使わない

![Minimum Rust version: 1.32](https://img.shields.io/badge/Minimum%20Rust%20Version-1.32-brightgreen.svg)

<!--
Long, long ago, Rust had a large, Erlang-like runtime. We chose to use
jemalloc instead of the system allocator, because it often improved
performance over the default system one. Over time, we shed more and more of
this runtime, and eventually almost all of it was removed, but jemalloc was
not. We didn't have a way to choose a custom allocator, and so we couldn't
really remove it without causing a regression for people who do need
jemalloc.
 -->
遠い遠い昔、RustはErlang風のとても大きなランタイムを持っていました。
その際、性能面での改善が得られていたので、システムの標準のアロケータではなく、jemallocを使うという選択をしました。
それ以来、我々はランタイムの機能を徐々に削ぎ落とし、最終的にはほとんど何もなくなりましたが、jemallocは残されていました。
カスタムのアロケータを選ぶ方法がなく、jemallocを必要としている人に影響を与えることなくそれを削除することが出来なかったからです。

<!--
Also, saying that jemalloc was always the default is a bit UNIX-centric, as
it was only the default on some platforms. Notably, the MSVC target on
Windows has shipped the system allocator for a long time.
-->
さらに、jemallocは特定のプラットフォームでのみデフォルトなので、それが常にデフォルトだと言うのはややUnix偏重主義でしょう。
特にWindows上のMSVCターゲットは、長い間、システムのアロケータで出荷されています。

<!--
While jemalloc usually has great performance, that's not always the case.
Additionally, it adds about 300kb to every Rust binary. We've also had a host
of other issues with jemalloc in the past. It has also felt a little strange
that a systems language does not default to the system's allocator.
-->
jemallocは通常とても良い性能を出しますが、常にというわけではありません。
加えて、jemallocを使うとRustのすべてのバイナリーが300kb大きくなります。
そして、過去にはjemallocにまつわる問題が多くありました。
また、システム言語であるRustが、システムが提供するアロケータを使わないのは少し変だとも考えられていました。

<!--
For all of these reasons, once Rust 1.28 shipped a way to choose a global
allocator, we started making plans to switch the default to the system
allocator, and allow you to use jemalloc via a crate. In Rust 1.32, we've
finally finished this work, and by default, you will get the system allocator
for your programs.
-->
これらの理由により、Rust 1.28でグローバルアロケーターを選択できるようになってから、我々はシステムの提供するアロケータをデフォルトにして、jemallocはクレートを通じて使えるようにする計画を立て始めました。
Rust 1.32でこれが完成し、それぞれのプラットフォームでシステムが提供するアロケータがデフォルトで使われるようになりました。

<!--
If you'd like to continue to use jemalloc, use the jemallocator crate. In
your Cargo.toml:
-->
もしjemallocを使い続けたいのであれば、jemallocatorクレートを使ってください。
あなたのCargo.tomlに以下のように書き、

```toml
jemallocator = "0.1.8"
```

<!--
And in your crate root:
-->
あなたのクレートのルートで以下のように書きます。

```rust,ignore
#[global_allocator]
static ALLOC: jemallocator::Jemalloc = jemallocator::Jemalloc;
```

<!--
That's it! If you don't need jemalloc, it's not forced upon you, and if you
do need it, it's a few lines of code away.
-->
これだけです。あなたがjemallocを必要としていなければ強制されませんし、もし必要であれば数行追加するだけです。
