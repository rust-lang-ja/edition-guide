<!-- 
# Unsafe functions 
-->

# unsafe 関数

<!-- 
## Summary 
-->

## 概要

<!-- 
    - The following functions are now marked [`unsafe`]:
    - [`std::env::set_var`]
    - [`std::env::remove_var`]
    - [`std::os::unix::process::CommandExt::before_exec`] 
-->

- 以下の関数が新たに [`unsafe`] としてマークされました：
- [`std::env::set_var`]
- [`std::env::remove_var`]
- [`std::os::unix::process::CommandExt::before_exec`]


[`unsafe`]: ../../reference/unsafe-keyword.html#unsafe-functions-unsafe-fn
[`std::env::set_var`]: ../../std/env/fn.set_var.html
[`std::env::remove_var`]: ../../std/env/fn.remove_var.html
[`std::os::unix::process::CommandExt::before_exec`]: ../../std/os/unix/process/trait.CommandExt.html#method.before_exec 


<!-- 
## Details 
-->

## 詳細

<!-- 
Over time it has become evident that certain functions in the standard library should have been marked as `unsafe`. However, adding `unsafe` to a function can be a breaking change since it requires existing code to be placed in an `unsafe` block. To avoid the breaking change, these functions are marked as `unsafe` starting in the 2024 Edition, while not requiring `unsafe` in previous editions. 
-->

標準ライブラリのいくつかの関数は、本来 `unsafe` であるべきであることが時間とともに明らかになってきました。しかし、関数を unsafe に変更すると、既存のコードでは `unsafe` ブロックで囲む必要が生じるため、破壊的変更となる可能性があります。そのため、これらの関数は 2024 Edition から `unsafe` にマークされますが、以前のエディションでは `unsafe` を要求しません。

<!-- 
### `std::env::{set_var, remove_var}` 
-->

### `std::env::{set_var, remove_var}`

<!-- 
It can be unsound to call [`std::env::set_var`] or [`std::env::remove_var`] in a multi-threaded program due to safety limitations of the way the process environment is handled on some platforms. The standard library originally defined these as safe functions, but it was later determined that was not correct. 
-->

[`std::env::set_var`] や [`std::env::remove_var`] をマルチスレッド環境で呼び出すことは、一部のプラットフォームにおける プロセス環境の管理方法の制約により、安全でない可能性があります。標準ライブラリではこれらの関数をもともと安全な関数として定義していましたが、それが適切でなかったことが後に判明しました。

<!-- 
It is important to ensure that these functions are not called when any other thread might be running. See the [Safety] section of the function documentation for more details. 
-->

これらの関数を呼び出す際は 他のスレッドが動作していないことを確実に保証する必要があります。詳細は、各関数の [Safety] セクションを参照してください。

[Safety]: ../../std/env/fn.set_var.html#safety

<!-- 
### `std::os::unix::process::CommandExt::before_exec` 
-->

### `std::os::unix::process::CommandExt::before_exec`

<!-- 
The [`std::os::unix::process::CommandExt::before_exec`] function is a unix-specific function which provides a way to run a closure before calling `exec`. This function was deprecated in the 1.37 release, and replaced with [`pre_exec`] which does the same thing, but is marked as `unsafe`. 
-->

[`std::os::unix::process::CommandExt::before_exec`] は、Unix 固有の関数であり、`exec` を呼び出す前にクロージャを実行するための手段を提供します。この関数は Rust 1.37 で非推奨となり、代わりに [`pre_exec`] が導入されました。`pre_exec` は同じ機能を提供しますが、明示的に `unsafe` としてマークされています。

<!-- 
Even though `before_exec` is deprecated, it is now correctly marked as `unsafe` starting in the 2024 Edition. This should help ensure that any legacy code which has not already migrated to `pre_exec` to require an `unsafe` block. 
-->

`before_exec` はすでに非推奨ですが、2024 Edition から `unsafe` として正しくマークされるようになりました。これにより、まだ `pre_exec` に移行していない レガシーコードに対しても `unsafe` ブロックが必要となるよう強制されます。

<!-- 
There are very strict safety requirements for the `before_exec` closure to satisfy. See the [Safety section][pre-exec-safety] for more details. 
-->

なお、`before_exec` のクロージャには 非常に厳格な安全性の要件があります。詳細は [Safety セクション][pre-exec-safety] を参照してください。

[`pre_exec`]: ../../std/os/unix/process/trait.CommandExt.html#tymethod.pre_exec
[pre-exec-safety]: ../../std/os/unix/process/trait.CommandExt.html#notes-and-safety

<!-- 
## Migration 
-->

## 移行

<!-- 
To make your code compile in both the 2021 and 2024 editions, you will need to make sure that these functions are called only from within `unsafe` blocks. 
-->

2021 Edition と 2024 Edition の両方でコードをコンパイル可能にするには、これらの関数を `unsafe` ブロック内でのみ呼び出すようにする必要があります。

<!-- 
**⚠ Caution**: It is important that you manually inspect the calls to these functions and possibly rewrite your code to satisfy the preconditions of those functions. In particular, `set_var` and `remove_var` should not be called if there might be multiple threads running. You may need to elect to use a different mechanism other than environment variables to manage your use case. 
-->

**⚠ 注意：** これらの関数を使用している箇所を 手動で確認し、必要に応じてコードを書き換えることが重要です。特に、`set_var` や `remove_var` は 複数のスレッドが動作している可能性がある場合には呼び出すべきではありません。環境変数の操作が必要な場合は、別の方法を検討することを推奨します。

<!-- 
The [`deprecated_safe_2024`] lint will automatically modify any use of these functions to be wrapped in an `unsafe` block so that it can compile on both editions. This lint is part of the `rust-2024-compatibility` lint group, which will automatically be applied when running `cargo fix --edition`. To migrate your code to be Rust 2024 Edition compatible, run: 
-->

[`deprecated_safe_2024`] リントは、これらの関数の呼び出しを自動的に `unsafe` ブロックで囲むよう修正します。これは `rust-2024-compatibility` リントグループに含まれており、次のコマンドを実行することで適用できます：

```sh
cargo fix --edition
```

<!-- 
For example, this will change: 
-->

たとえば、以下のコード：

```rust
fn main() {
    std::env::set_var("FOO", "123");
}
```

<!-- 
to be: 
-->

は、自動修正により次のように変更されます：

```rust
fn main() {
    // TODO: Audit that the environment access only happens in single-threaded code.
    // TODO: 環境変数の操作がシングルスレッドのコード内でのみ行われていることを確認する。
    unsafe { std::env::set_var("FOO", "123") };
}
```

<!-- 
Just beware that this automatic migration will not be able to verify that these functions are being used correctly. It is still your responsibility to manually review their usage. 
-->

ただし、この自動移行では、これらの関数が正しく使用されているかどうかを検証することはできません。よって、最終的な責任は開発者にあり手動でコードを確認する必要があります。

<!-- 
Alternatively, you can manually enable the lint to find places these functions are called: 
-->

または、リントを手動で有効にして、これらの関数が呼び出されている箇所を特定することもできます。

```rust
// Add this to the root of your crate to do a manual migration.
// クレートのルートに以下を追加して手動移行をサポート
#![warn(deprecated_safe_2024)]
```

[`deprecated_safe_2024`]: ../../rustc/lints/listing/allowed-by-default.html#deprecated-safe-2024
