<!--
# Uniform Paths
-->
# 統一的なパス

![Minimum Rust version: 1.32](https://img.shields.io/badge/Minimum%20Rust%20Version-1.32-brightgreen.svg)

<!--
Rust 2018 added several improvements to the module system. We have one last
tweak landing in 1.32.0. Nicknamed "uniform paths", it permits previously
invalid import path statements to be resolved exactly the same way as
non-import paths. For example:
-->
Rust 2018でモジュールシステムに幾つかの改善が加えられましたが、Rust 1.32.0で最終的な微調整の変更を一つ入れています。
「統一的なパス」と呼ばれるこの機能により、以前は不正だったインポートパス文を、インポート以外のパスと同じ様に解釈できるようになりました。例えば、以下の例を考えます。

```rust,edition2018
enum Color {
    Red,
    Green,
    Blue,
}

use Color::*;
```

<!--
This code did not previously compile, as use statements had to start with
`super`, `self`, or `crate`. Now that the compiler supports uniform paths,
this code will work, and do what you probably expect: import the variants of
the Color enum defined above the `use` statement.
-->
以前は、use文は`super`、`self`あるいは`crate`で始まらなければならなかったので、このコードはコンパイル出来ませんでした。
今ではコンパイラーが統一的なパスをサポートするようになり、このコードはあなたの想像通りの動作をします。
つまり、`use`文の上で宣言されているColor列挙型のヴァリアントをインポートします。
