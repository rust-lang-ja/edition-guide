<!-- 
# Add `IntoIterator` for `Box<[T]>` 
-->

# Box<[T]> に対する IntoIterator の追加

<!-- 
## Summary 
-->

## 概要

<!-- 
- Boxed slices implement [`IntoIterator`] in *all* editions.
- Calls to [`IntoIterator::into_iter`] are *hidden* in editions prior to 2024 when using method call syntax (i.e., `boxed_slice.into_iter()`). So, `boxed_slice.into_iter()` still resolves to `(&(*boxed_slice)).into_iter()` as it has before.
- `boxed_slice.into_iter()` changes meaning to call [`IntoIterator::into_iter`] in Rust 2024. 
-->

- ボックス化されたスライスは、*すべての* エディションで [`IntoIterator`] を実装します。
- 2024年以前のエディションでは、メソッド呼び出し構文（例： `boxed_slice.into_iter()` ）で [`IntoIterator::into_iter`] への呼び出しが *隠蔽* されるため、これまでどおり `boxed_slice.into_iter()` は `(&(*boxed_slice)).into_iter()` として解釈されます。
- Rust 2024 では、`boxed_slice.into_iter()` の意味が `IntoIterator::into_iter` を呼び出すものに変わります。

[`IntoIterator`]: ../../std/iter/trait.IntoIterator.html
[`IntoIterator::into_iter`]: ../../std/iter/trait.IntoIterator.html#tymethod.into_iter

<!-- 
## Details 
-->

## 詳細

<!-- 
Until Rust 1.80, `IntoIterator` was not implemented for boxed slices. In prior versions, if you called `.into_iter()` on a boxed slice, the method call would automatically dereference from `Box<[T]>` to `&[T]`, and return an iterator that yielded references of `&T`. For example, the following worked in prior versions: 
-->

Rust 1.80 以前は、ボックス化されたスライスに対して `IntoIterator` が実装されていませんでした。以前のバージョンでは、ボックス化されたスライスに対して `.into_iter()` を呼び出すと、メソッド呼び出しが自動的に `Box<[T]>` から `&[T]` への参照外しを行い、`&T` の参照を返すイテレータが生成されました。たとえば、以下のコードは以前のエディションで動作していました：

```rust
// Example of behavior in previous editions.
// 以前のエディションでの動作例
let my_boxed_slice: Box<[u32]> = vec![1, 2, 3].into_boxed_slice();
// Note: .into_iter() was required in versions older than 1.80
// 注意：1.80以前のバージョンでは .into_iter() の呼び出しが必要でした
for x in my_boxed_slice.into_iter() {
    // x is of type &u32 in editions prior to 2024
    // Rust 2024以前のエディションでは、x の型は &u32 です
}
```

<!-- 
In Rust 1.80, implementations of `IntoIterator` were added for boxed slices. This allows iterating over elements of the slice by-value instead of by-reference: 
-->

Rust 1.80 では、ボックス化されたスライスに対して `IntoIterator` の実装が追加されました。これにより、スライスの要素を参照ではなく値としてイテレートできるようになります：

```rust
// NEW as of 1.80, all editions
// 1.80以降、すべてのエディションでの新しい動作
let my_boxed_slice: Box<[u32]> = vec![1, 2, 3].into_boxed_slice();
for x in my_boxed_slice { // notice no need for calling .into_iter()
                          // .into_iter() の呼び出しは不要です
    // x is of type u32
    // x の型は u32 です
}
```

<!-- 
This example is allowed on all editions because previously this was an error since `for` loops do not automatically dereference like the `.into_iter()` method call does. 
-->

この例は、すべてのエディションで許可されています。というのも、以前は `for` ループが `.into_iter()` のように自動的に参照外しを行わなかったため、そもそもエラーになっていたからです。

<!-- 
However, this would normally be a breaking change because existing code that manually called `.into_iter()` on a boxed slice would change from having an iterator over references to an iterator over values. To resolve this problem, method calls of `.into_iter()` on boxed slices have edition-dependent behavior. In editions before 2024, it continues to return an iterator over references, and starting in Edition 2024 it returns an iterator over values. 
-->

しかし、本来であれば、これは破壊的変更となる可能性があります。なぜなら、これまで `.into_iter()` を明示的に呼び出していたコードの動作が、参照を返すイテレータから値を返すイテレータへと変わってしまうからです。この問題を解決するために、ボックス化されたスライスの `.into_iter()` の動作はエディションによって異なります。2024年以前のエディションでは、これまでどおり参照を返すイテレータを生成します。Rust 2024 以降では、値を返すイテレータを生成します。

```rust,edition2024
// Example of changed behavior in Edition 2024
// Edition 2024 での動作変更の例
let my_boxed_slice: Box<[u32]> = vec![1, 2, 3].into_boxed_slice();
// Example of old code that still manually calls .into_iter()
// 以前のコードで .into_iter() を明示的に呼び出していた場合
for x in my_boxed_slice.into_iter() {
    // x is now type u32 in Edition 2024
    // Edition 2024 では、x の型は u32 になります
}
```

<!-- 
## Migration 
-->

## 移行

<!-- 
The [`boxed_slice_into_iter`] lint will automatically modify any calls to `.into_iter()` on boxed slices to call `.iter()` instead to retain the old behavior of yielding references. This lint is part of the `rust-2024-compatibility` lint group, which will automatically be applied when running `cargo fix --edition`. To migrate your code to be Rust 2024 Edition compatible, run: 
-->

[`boxed_slice_into_iter`] リントは、ボックス化されたスライスに対する `.into_iter()` の呼び出しを `.iter()` に自動で置き換え、従来どおり参照を返すように修正します。このリントは `rust-2024-compatibility` リントグループの一部であり、`cargo fix --edition` を実行すると自動的に適用されます。Rust 2024 エディションに対応するために、次のコマンドを実行してください。

```sh
cargo fix --edition
```

<!-- 
For example, this will change: 
-->

例えば、以下のコードは：

```rust
fn main() {
    let my_boxed_slice: Box<[u32]> = vec![1, 2, 3].into_boxed_slice();
    for x in my_boxed_slice.into_iter() {
        // x is of type &u32
        // x の型は &u32
    }
}
```

<!-- 
to be: 
-->

次のように修正されます：

```rust
fn main() {
    let my_boxed_slice: Box<[u32]> = vec![1, 2, 3].into_boxed_slice();
    for x in my_boxed_slice.iter() {
        // x is of type &u32
        // x の型は &u32
    }
}
```

<!-- 
The [`boxed_slice_into_iter`] lint is defaulted to warn on all editions, so unless you have manually silenced the lint, you should already see it before you migrate. 
-->

[`boxed_slice_into_iter`] リントはすべてのエディションでデフォルトで警告を出す設定になっているため、手動でリントを無効化していなければ、移行前にすでに警告が表示されるはずです。

[`boxed_slice_into_iter`]: ../../rustc/lints/listing/warn-by-default.html#boxed-slice-into-iter
