<!--
# Never type fallback change
-->

# never 型のフォールバック先の変更

<!--
## Summary
-->

## 概要

<!--
- Never type (`!`) to any type ("never-to-any") coercions fall back to never type (`!`) rather than to unit type (`()`).
- The [`never_type_fallback_flowing_into_unsafe`] lint is now `deny` by default.
-->

- never 型（`!`）から任意型への（never-to-any の）型強制において、失敗時のフォールバック先が never 型（`!`）でなくユニット型（`()`）となります。
- [`never_type_fallback_flowing_into_unsafe`] のリントレベルはデフォルトで `deny` （必ずエラー）となりました。

<!--
[`never_type_fallback_flowing_into_unsafe`]: ../../rustc/lints/listing/warn-by-default.html#never-type-fallback-flowing-into-unsafe
-->

[`never_type_fallback_flowing_into_unsafe`]: https://doc.rust-lang.org/rustc/lints/listing/warn-by-default.html#never-type-fallback-flowing-into-unsafe

<!--
## Details
-->

## 詳細

<!--
When the compiler sees a value of type `!` (never) in a [coercion site][], it implicitly inserts a coercion to allow the type checker to infer any type:
-->

[型強制サイト][]（型強制可能な場所）に `!` (never) と呼ばれる値があるとき、型検査機が任意の型を推論できるように、コンパイラはそこにある種の変換をさし挟みます。

<!--
```rust,should_panic
# #![feature(never_type)]
// This:
let x: u8 = panic!();

// ...is (essentially) turned by the compiler into:
let x: u8 = absurd(panic!());

// ...where `absurd` is the following function
// (it's sound because `!` always marks unreachable code):
fn absurd<T>(x: !) -> T { x }
```
-->

```rust,should_panic
# #![feature(never_type)]
// 以下のコードは
let x: u8 = panic!();

// コンパイラによって（実質的に）以下のように書き換えられる
let x: u8 = yaba(panic!());

// ここで、`yaba` 関数は以下のような関数
//（`!` は到達不能なコードに対応するので、これは正当なコード）
fn yaba<T>(x: !) -> T { x }
```

<!--
This can lead to compilation errors if the type cannot be inferred:
-->

このままだと型推論に失敗したときにコンパイルエラーが発生してしまいます。

<!--
```rust,compile_fail,E0282
# #![feature(never_type)]
# fn absurd<T>(x: !) -> T { x }
// This:
{ panic!() };

// ...gets turned into this:
{ absurd(panic!()) }; //~ ERROR can't infer the type of `absurd`
```
-->

```rust,compile_fail,E0282
# #![feature(never_type)]
# fn yaba<T>(x: !) -> T { x }
// これが
{ panic!() };

// これになる
{ yaba(panic!()) }; //~ ERROR can't infer the type of `yaba`
                    //~ (訳) エラー: `yaba` の型が推論できません
```

<!--
To prevent such errors, the compiler remembers where it inserted `absurd` calls, and if it can't infer the type, it uses the fallback type instead:
-->

このようなエラーを避けるため、コンパイラには、「`yaba` が挿入された場所で型推論に失敗した場合には、代わりにそれをこの型と見做して処理を続ける」というような「失敗時のデフォルト」用の型（フォールバック先の型）が決まっています。

<!--
```rust,should_panic
# #![feature(never_type)]
# fn absurd<T>(x: !) -> T { x }
type Fallback = /* An arbitrarily selected type! */ !;
{ absurd::<Fallback>(panic!()) }
```
-->

```rust,should_panic
# #![feature(never_type)]
# fn absurd<T>(x: !) -> T { x }
type Fallback = /* フォールバック先をどの型にするかはコンパイラの自由！*/ !;
{ absurd::<Fallback>(panic!()) }
```

<!--
This is what is known as "never type fallback".
-->

この現象は、never 型のフォールバックと呼ばれます。

<!--
Historically, the fallback type has been `()` (unit).  This caused `!` to spontaneously coerce to `()` even when the compiler would not infer `()` without the fallback.  That was confusing and has prevented the stabilization of the `!` type.
-->

かつてはフォールバック先の型は `()` 型（ユニット型）でした。
これにより、フォールバックが発生しなければコンパイラが `()` を推論としないはずの場面であっても、実際には `!` が勝手に `()` に強制されてしまいました。
この動作は非直感的で、`!` 型の安定化も妨げていました。

<!--
In the 2024 edition, the fallback type is now `!`.  (We plan to make this change across all editions at a later date.)  This makes things work more intuitively.  Now when you pass `!` and there is no reason to coerce it to something else, it is kept as `!`.
-->

2024 エディションから、フォールバック先の型が `!` 型になりました。
（将来的には全エディションで採用される予定です。）
これによりコンパイラの動作がより直感的になります。
これからは、別の型に強制される特段の理由がない限り、`!` は `!` のままです。

<!--
In some cases your code might depend on the fallback type being `()`, so this can cause compilation errors or changes in behavior.
-->

`()` へのフォールバックを前提としたコードは、本変更によってコンパイルエラーになったり、挙動が変わったりするおそれがあります。

<!--
[coercion site]: ../../reference/type-coercions.html#coercion-sites
-->

[型強制サイト]: https://doc.rust-lang.org/reference/type-coercions.html#coercion-sites

### `never_type_fallback_flowing_into_unsafe`

<!--
The default level of the [`never_type_fallback_flowing_into_unsafe`] lint has been raised from `warn` to `deny` in the 2024 Edition. This lint helps detect a particular interaction with the fallback to `!` and `unsafe` code which may lead to undefined behavior. See the link for a complete description.
-->

2024 エディションでは、[`never_type_fallback_flowing_into_unsafe`] のデフォルトレベルが `warn`（警告）から `deny`（必ずエラー）に格上げされました。
このリントは、`unsafe` 文脈における `!` へのフォールバックが関わる動作のうち、未定義動作を引き起こす可能性のあるものを検知します。
詳細説明はリンク先をご参照ください。

<!--
## Migration
-->

## 移行

<!--
There is no automatic fix, but there is automatic detection of code that will be broken by the edition change.  While still on a previous edition you will see warnings if your code will be broken.
-->

自動修正はできませんが、エディションの更新によって壊れうるコードを自動検知することはできます。
また、過去のエディションにおいてはコードが壊れうる場合に警告が出ます。

<!--
The fix is to specify the type explicitly so that the fallback type is not used.  Unfortunately, it might not be trivial to see which type needs to be specified.
-->

修正方法は、フォールバックが起こらないように強制先となるべき型を明示することです。
残念ながら、どの型を指定すべきかは自明ではありません。

<!--
One of the most common patterns broken by this change is using `f()?;` where `f` is generic over the `Ok`-part of the return type:
-->

壊れうるコードの中では比較的よく見られるパターンとしては、`f()?;` という式において、特に `f` の戻り値の型が `Ok` 側の型引数に対してジェネリックな場合が挙げられます。

```rust
# #![allow(dependency_on_unit_never_type_fallback)]
# fn outer<T>(x: T) -> Result<T, ()> {
fn f<T: Default>() -> Result<T, ()> {
    Ok(T::default())
}

f()?;
# Ok(x)
# }
```

<!--
You might think that, in this example, type `T` can't be inferred.  However, due to the current desugaring of the `?` operator, it was inferred as `()`, and it will now be inferred as `!`.
-->

一見、型 `T` は推論不可能に見えますが、`?` 構文の脱糖 (desugaring) 方法の関係で、かつては `()` と、今は `!` と推論されます。

<!--
To fix the issue you need to specify the `T` type explicitly:
-->

対策としては、`T` の型を明示するとよいです。

<!--
```rust,edition2024
# #![deny(dependency_on_unit_never_type_fallback)]
# fn outer<T>(x: T) -> Result<T, ()> {
# fn f<T: Default>() -> Result<T, ()> {
#     Ok(T::default())
# }
f::<()>()?;
// ...or:
() = f()?;
# Ok(x)
# }
```
-->

```rust,edition2024
# #![deny(dependency_on_unit_never_type_fallback)]
# fn outer<T>(x: T) -> Result<T, ()> {
# fn f<T: Default>() -> Result<T, ()> {
#     Ok(T::default())
# }
f::<()>()?;
// または
() = f()?;
# Ok(x)
# }
```

<!--
Another relatively common case is panicking in a closure:
-->

他のよくある例としては、クロージャ中でパニックする場合が挙げられます。

```rust,should_panic
# #![allow(dependency_on_unit_never_type_fallback)]
trait Unit {}
impl Unit for () {}

fn run<R: Unit>(f: impl FnOnce() -> R) {
    f();
}

run(|| panic!());
```

<!--
Previously `!` from the `panic!` coerced to `()` which implements `Unit`.  However now the `!` is kept as `!` so this code fails because `!` doesn't implement `Unit`.  To fix this you can specify the return type of the closure:
-->

かつては、`panic!` の返す `!` は `Unit` を実装する `()` に強制されていました。
本エディションからは、`!` が `!` のままなので、`!` が `Unit` を実装しない以上コンパイルに失敗してしまいます。
このエラーは、クロージャの返り値の型を明示すると解決できます。

```rust,edition2024,should_panic
# #![deny(dependency_on_unit_never_type_fallback)]
# trait Unit {}
# impl Unit for () {}
#
# fn run<R: Unit>(f: impl FnOnce() -> R) {
#     f();
# }
run(|| -> () { panic!() });
```

<!--
A similar case to that of `f()?` can be seen when using a `!`-typed expression in one branch and a function with an unconstrained return type in the other:
-->

`f()?` と似た例として、条件分岐等の一方が `!` 型の式で、もう片方が返り値の型が指定されていない関数の呼び出しである場合が挙げられます。

```rust
# #![allow(dependency_on_unit_never_type_fallback)]
if true {
    Default::default()
} else {
    return
};
```

<!--
Previously `()` was inferred as the return type of `Default::default()` because `!` from `return` was spuriously coerced to `()`.  Now, `!` will be inferred instead causing this code to not compile because `!` does not implement `Default`.
-->

かつては `return` の型である `!` が誤って `()` に強制されていたため、`Default::default()` の戻り値型として `()` が推論されていました。
本エディションからは `!` が `!` のままと推論されるようになり、`!` は `Default` を実装しないのでコンパイルエラーになります。

<!--
Again, this can be fixed by specifying the type explicitly:
-->

これも同じく、型を明示することで解決できます。

<!--
```rust,edition2024
# #![deny(dependency_on_unit_never_type_fallback)]
() = if true {
    Default::default()
} else {
    return
};

// ...or:

if true {
    <() as Default>::default()
} else {
    return
};
```
-->

```rust,edition2024
# #![deny(dependency_on_unit_never_type_fallback)]
() = if true {
    Default::default()
} else {
    return
};

// または

if true {
    <() as Default>::default()
} else {
    return
};
```
