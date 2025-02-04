<!--
# RPIT lifetime capture rules
-->

# RPIT におけるライフタイムキャプチャ規則

<!--
This chapter describes changes related to the **Lifetime Capture Rules 2024** introduced in [RFC 3498], including how to use opaque type *precise capturing* (introduced in [RFC 3617]) to migrate your code.
-->

本節では、[RFC 3498] で導入された**ライフタイムキャプチャ規則 2024** に関する変更事項を説明します。
また、（[RFC 3617] で導入された）不透明型への**精密なキャプチャ**を使ったコードへの移行方法も説明します。

[RFC 3498]: https://github.com/rust-lang/rfcs/pull/3498
[RFC 3617]: https://github.com/rust-lang/rfcs/pull/3617

<!--
## Summary
-->

## 概要

<!--
- In Rust 2024, *all* in-scope generic parameters, including lifetime parameters, are implicitly captured when the `use<..>` bound is not present.
- Uses of the `Captures` trick (`Captures<..>` bounds) and of the outlives trick (e.g. `'_` bounds) can be replaced by `use<..>` bounds (in all editions) or removed entirely (in Rust 2024).
-->

- Rust 2024 では、`use<..>` を書かない限り、スコープ内の **すべての** ジェネリックパラメータ（ライフタイムパラメータを含む）が暗黙にキャプチャされます。
- `Captures` パターン（`Captures<..>` 境界）や outlive パターン（`'_` 境界）は、（どのエディションでも）`use<..>` 境界に書き換えたり、（Rust 2024 では）削除したりできます。

<!--
## Details
-->

## 詳細

<!--
### Capturing
-->

### キャプチャ

<!--
*Capturing* a generic parameter in an RPIT (return-position impl Trait) opaque type allows for that parameter to be used in the corresponding hidden type.  In Rust 1.82, we added `use<..>` bounds that allow specifying explicitly which generic parameters to capture.  Those will be helpful for migrating your code to Rust 2024, and will be helpful in this chapter for explaining how the edition-specific implicit capturing rules work.  These `use<..>` bounds look like this:
-->

戻り値としての `impl Trait` (return-position impl Trait, 以下 RPIT) 不透明型が「ジェネリックパラメータをキャプチャする」とは、不透明型に隠蔽された型がそのパラメータを使用できる、ということです。
Rust 1.82 では `use<..>` 境界が導入され、どのジェネリックパラメータがキャプチャされているのかを明示することができるようになりました。
この構文は Rust 2024 への移行にも活用できますし、本節では各エディションにおける暗黙のキャプチャ規則を説明するためにも用います。
`use<..>` 境界は次のように使用します。

<!--
```rust
# #![feature(precise_capturing)]
fn capture<'a, T>(x: &'a (), y: T) -> impl Sized + use<'a, T> {
    //                                ~~~~~~~~~~~~~~~~~~~~~~~
    //                             This is the RPIT opaque type.
    //
    //                                It captures `'a` and `T`.
    (x, y)
  //~~~~~~
  // The hidden type is: `(&'a (), T)`.
  //
  // This type can use `'a` and `T` because they were captured.
}
```
-->

```rust
# #![feature(precise_capturing)]
fn capture<'a, T>(x: &'a (), y: T) -> impl Sized + use<'a, T> {
    //                                ~~~~~~~~~~~~~~~~~~~~~~~
    //                                これが RPIT 不透明型です。
    //
    //                        `'a` と `T` をキャプチャしています。
    (x, y)
  //~~~~~~
  // 型 `(&'a (), T)` が隠蔽されています。
  //
  // この型が `'a` と `T` を使えるのは、これらがキャプチャされているからです。
}
```

<!--
The generic parameters that are captured affect how the opaque type can be used.  E.g., this is an error because the lifetime is captured despite the fact that the hidden type does not use the lifetime:
-->

どのジェネリックパラメータがキャプチャされるか次第で、不透明型が使える場面も決まります。
例えば、以下のコードではライフタイム `'a` が使用されていないにもかかわらず、`'a` がキャプチャされているためエラーになります。

```rust,compile_fail
# #![feature(precise_capturing)]
fn capture<'a>(_: &'a ()) -> impl Sized + use<'a> {}

fn test<'a>(x: &'a ()) -> impl Sized + 'static {
    capture(x)
    //~^ ERROR lifetime may not live long enough
    // エラー: ライフタイムが十分に長生きしません
}
```

<!--
Conversely, this is OK:
-->

一方で、こうすれば大丈夫です。

```rust
# #![feature(precise_capturing)]
fn capture<'a>(_: &'a ()) -> impl Sized + use<> {}

fn test<'a>(x: &'a ()) -> impl Sized + 'static {
    capture(x) //~ OK
}
```

<!--
### Edition-specific rules when no `use<..>` bound is present
-->

### `use<..>` がない場合のエディションごとの挙動の違い

<!--
If the `use<..>` bound is not present, then the compiler uses edition-specific rules to decide which in-scope generic parameters to capture implicitly.
-->

`use<..>` が明示されていない場合、スコープ内のジェネリックパラメータのうちどれがキャプチャされるかは、エディションによって挙動が異なります。

<!--
In all editions, all in-scope type and const generic parameters are captured implicitly when the `use<..>` bound is not present.  E.g.:
-->

スコープ内のジェネリックな型パラメータと const パラメータは、`use<..>` が明示されていないときはエディションにかかわらず全て暗黙にキャプチャされます。

<!--
```rust
# #![feature(precise_capturing)]
fn f_implicit<T, const C: usize>() -> impl Sized {}
//                                    ~~~~~~~~~~
//                         No `use<..>` bound is present here.
//
// In all editions, the above is equivalent to:
fn f_explicit<T, const C: usize>() -> impl Sized + use<T, C> {}
```
-->

```rust
# #![feature(precise_capturing)]
fn f_implicit<T, const C: usize>() -> impl Sized {}
//                                    ~~~~~~~~~~
//                          `use<..>` が明示されていません。
//
// どのエディションでも、上記は以下と等価です。
fn f_explicit<T, const C: usize>() -> impl Sized + use<T, C> {}
```

<!--
In Rust 2021 and earlier editions, when the `use<..>` bound is not present, generic lifetime parameters are only captured when they appear syntactically within a bound in RPIT opaque types in the signature of bare functions and associated functions and methods within inherent impls.  However, starting in Rust 2024, these in-scope generic lifetime parameters are unconditionally captured.  E.g.:
-->

ジェネリックなライフタイムパラメータに関しては、Rust 2021 以前のエディションでは、`use<..>` が明示されていない場合、トップレベルの関数か固有 impl [^1] 内の関連関数・メソッドにおける RPIT 不透明型の境界に現れるものだけがキャプチャされます。一方、Rust 2024 以降では、スコープ内のジェネリックなライフタイムパラメータは全てキャプチャされます。例えば以下の通りです。

<!--
```rust
# #![feature(precise_capturing)]
fn f_implicit(_: &()) -> impl Sized {}
// In Rust 2021 and earlier, the above is equivalent to:
fn f_2021(_: &()) -> impl Sized + use<> {}
// In Rust 2024 and later, it's equivalent to:
fn f_2024(_: &()) -> impl Sized + use<'_> {}
```
-->

```rust
# #![feature(precise_capturing)]
fn f_implicit(_: &()) -> impl Sized {}
// Rust 2021 以前では、上記の定義は以下と等価です。
fn f_2021(_: &()) -> impl Sized + use<> {}
// Rust 2024 以降では、上記の定義は以下と等価です。
fn f_2024(_: &()) -> impl Sized + use<'_> {}
```

<!--
This makes the behavior consistent with RPIT opaque types in the signature of associated functions and methods within trait impls, uses of RPIT within trait definitions (RPITIT), and opaque `Future` types created by `async fn`, all of which implicitly capture all in-scope generic lifetime parameters in all editions when the `use<..>` bound is not present.
-->

これは、他のシグネチャにおける RPIT 不透明型の挙動に合わせたものです。
実際、トレイト impl [^2] 内の関連関数・メソッド、トレイト定義内の RPIT（Return-Position Impl Trait In Trait; RPITIT）、`asnyc fn` によって生成される不透明な `Future` 型は、エディションにかかわらず、スコープ内のジェネリックなライフタイムパラメータを全て暗黙にキャプチャするようになっています。

[^1]: 訳注: 固有 impl とは、`struct MyStruct;` に対する `impl MyStruct { }` ブロックのように、型に対して直接定義される `impl` ブロックのことです。
[^2]: 訳注: トレイト impl とは、`trait MyTrait` に対する `impl MyTrait for MyStruct { }` ブロックのように、トレイトの実装を与えるための `impl` ブロックのことです。

<!--
### Outer generic parameters
-->

### 外側のジェネリックパラメータ

<!--
Generic parameters from an outer impl are considered to be in scope when deciding what is implicitly captured.  E.g.:
-->

暗黙にキャプチャされるジェネリックパラメータを決定するとき、外側の impl で定義されたパラメータはスコープ内のパラメータと見なされます。
例えば以下の通りです。

<!--
```rust
# #![feature(precise_capturing)]
struct S<T, const C: usize>((T, [(); C]));
impl<T, const C: usize> S<T, C> {
//   ~~~~~~~~~~~~~~~~~
// These generic parameters are in scope.
    fn f_implicit<U>() -> impl Sized {}
    //            ~       ~~~~~~~~~~
    //            ^ This generic is in scope too.
    //                    ^
    //                    |
    //     No `use<..>` bound is present here.
    //
    // In all editions, it's equivalent to:
    fn f_explicit<U>() -> impl Sized + use<T, U, C> {}
}
```
-->

```rust
# #![feature(precise_capturing)]
struct S<T, const C: usize>((T, [(); C]));
impl<T, const C: usize> S<T, C> {
//   ~~~~~~~~~~~~~~~~~
// これらのジェネリックパラメータはスコープ内にあります。
    fn f_implicit<U>() -> impl Sized {}
    //            ~       ~~~~~~~~~~
    //            ^ これもスコープ内です。
    //                    ^
    //                    |
    //     `use<..>` 境界が明示されていません。
    //
    // どのエディションでも、これは以下と等価です。
    fn f_explicit<U>() -> impl Sized + use<T, U, C> {}
}
```

<!--
### Lifetimes from higher-ranked binders
-->

### 高階束縛のライフタイム

<!--
Similarly, generic lifetime parameters introduced into scope by a higher-ranked `for<..>` binder are considered to be in scope.  E.g.:
-->

同様に、高階の `for<..>` 束縛で定義されたライフタイムパラメータも、スコープ内のパラメータと見なされます。
例えば以下の通りです。

<!--
```rust
# #![feature(precise_capturing)]
trait Tr<'a> { type Ty; }
impl Tr<'_> for () { type Ty = (); }

fn f_implicit() -> impl for<'a> Tr<'a, Ty = impl Copy> {}
// In Rust 2021 and earlier, the above is equivalent to:
fn f_2021() -> impl for<'a> Tr<'a, Ty = impl Copy + use<>> {}
// In Rust 2024 and later, it's equivalent to:
//fn f_2024() -> impl for<'a> Tr<'a, Ty = impl Copy + use<'a>> {}
//                                        ~~~~~~~~~~~~~~~~~~~~
// However, note that the capturing of higher-ranked lifetimes in
// nested opaque types is not yet supported.
```
-->

```rust
# #![feature(precise_capturing)]
trait Tr<'a> { type Ty; }
impl Tr<'_> for () { type Ty = (); }

fn f_implicit() -> impl for<'a> Tr<'a, Ty = impl Copy> {}
// Rust 2021 以前では、上記は以下と等価です。
fn f_2021() -> impl for<'a> Tr<'a, Ty = impl Copy + use<>> {}
// Rust 2024 以降では、上記は以下と等価です。
//fn f_2024() -> impl for<'a> Tr<'a, Ty = impl Copy + use<'a>> {}
//                                        ~~~~~~~~~~~~~~~~~~~~
// ただし、ネストされた不透明型における高階ライフタイムのキャプチャは
// まだサポート外であることにご注意ください。
```

<!--
### Argument position impl Trait (APIT)
-->

### 引数としての impl Trait (APIT)

<!--
Anonymous (i.e. unnamed) generic parameters created by the use of APIT (argument position impl Trait) are considered to be in scope.  E.g.:
-->

引数としての impl Trait (argument position impl Trait, 以下 APIT) で作られる匿名（無名）ジェネリックパラメータも、スコープ内のパラメータと見なされます。
例えば以下の通りです。

<!--
```rust
# #![feature(precise_capturing)]
fn f_implicit(_: impl Sized) -> impl Sized {}
//               ~~~~~~~~~~
//           This is called APIT.
//
// The above is *roughly* equivalent to:
fn f_explicit<_0: Sized>(_: _0) -> impl Sized + use<_0> {}
```
-->

```rust
# #![feature(precise_capturing)]
fn f_implicit(_: impl Sized) -> impl Sized {}
//               ~~~~~~~~~~
//           これが APIT です。
//
// 上記は「概ね」以下と等価です。
fn f_explicit<_0: Sized>(_: _0) -> impl Sized + use<_0> {}
```

<!--
Note that the former is not *exactly* equivalent to the latter because, by naming the generic parameter, turbofish syntax can now be used to provide an argument for it.  There is no way to explicitly include an anonymous generic parameter in a `use<..>` bound other than by converting it to a named generic parameter.
-->

ただし、これは「完全に」等価ではありません。後者のようにジェネリックパラメータを名前付きにした場合、turbofish `::<>` 構文で引数を明示することができてしまうからです。
実際には `use<..>` 境界で匿名ジェネリックパラメータを明示したい場合、名前付きジェネリックパラメータとして定義し直すしかありません。

<!--
## Migration
-->

## 移行

<!--
### Migrating while avoiding overcapturing
-->

### 移行時にキャプチャしすぎを回避する

<!--
The `impl_trait_overcaptures` lint flags RPIT opaque types that will capture additional lifetimes in Rust 2024.  This lint is part of the `rust-2024-compatibility` lint group which is automatically applied when running `cargo fix --edition`.  In most cases, the lint can automatically insert `use<..>` bounds where needed such that no additional lifetimes are captured in Rust 2024.
-->

Rust 2024 で新たにキャプチャされるライフタイムは、`impl_trait_overcaptures` リントで検出可能です。
このリントは `cargo fix --edition` で自動適用される `rust-2024-compatibility` リントの一部です。
ほとんどの場合、リントが必要に応じて `use<..>` を自動的に挿入し、Rust 2024 への移行時にもキャプチャされるライフタイムが増えないようにします。

<!--
To migrate your code to be compatible with Rust 2024, run:
-->

コードを Rust 2024 互換に移行するには、以下を実行します。

```sh
cargo fix --edition
```

<!--
For example, this will change:
-->

例えば、

```rust
fn f<'a>(x: &'a ()) -> impl Sized { *x }
```

<!--
...into:
-->

は

```rust
# #![feature(precise_capturing)]
fn f<'a>(x: &'a ()) -> impl Sized + use<> { *x }
```

に書き換えられます。

<!--
Without this `use<>` bound, in Rust 2024, the opaque type would capture the `'a` lifetime parameter.  By adding this bound, the migration lint preserves the existing semantics.
-->

`use<>` 境界がないと、返り値の不透明型が `'a` をキャプチャしてしまいますが、移行リントが `use<>` を追加したことにより、意味合いが変わらないようになっています。

<!--
### Migrating cases involving APIT
-->

## APIT が絡む移行

<!--
In some cases, the lint cannot make the change automatically because a generic parameter needs to be given a name so that it can appear within a `use<..>` bound.  In these cases, the lint will alert you that a change may need to be made manually.  E.g., given:
-->

ジェネリックパラメータが名無しであるために `use<..>` で参照できず、リントが自動移行に失敗する場合があります。
この場合、手動操作が必要であるとリントが報告します。
たとえば、以下のコードを考えます。

<!--
```rust,edition2021
fn f<'a>(x: &'a (), y: impl Sized) -> impl Sized { (*x, y) }
//   ^^                ~~~~~~~~~~
//               This is a use of APIT.
//
//~^ WARN `impl Sized` will capture more lifetimes than possibly intended in edition 2024
//~| NOTE specifically, this lifetime is in scope but not mentioned in the type's bounds
#
# fn test<'a>(x: &'a (), y: ()) -> impl Sized + 'static {
#     f(x, y)
# }
```
-->

```rust,edition2021
fn f<'a>(x: &'a (), y: impl Sized) -> impl Sized { (*x, y) }
//   ^^                ~~~~~~~~~~
//               ここで APIT が使われています。
//
//~^ WARN `impl Sized` will capture more lifetimes than possibly intended in edition 2024
//~| NOTE specifically, this lifetime is in scope but not mentioned in the type's bounds
// (訳)
//~^ 警告: 2024 エディションにすると、`impl Sized` が想定より多くのライフタイムをキャプチャする可能性があります。
//~| 情報: 特に、このライフタイムはスコープ内にありますが、型境界に明示れていません
#
# fn test<'a>(x: &'a (), y: ()) -> impl Sized + 'static {
#     f(x, y)
# }
```

<!--
The code cannot be converted automatically because of the use of APIT and the fact that the generic type parameter must be named in the `use<..>` bound.  To convert this code to Rust 2024 without capturing the lifetime, you must name that type parameter.  E.g.:
-->
ここで、変換が失敗するのは、ジェネリックパラメータを全て `use<..>` 境界で列挙したいにもかかわらず、無名型を用いた APIT が使われているからです。
コードを Rust 2024 に変換しつつ、ライフタイムがキャプチャされないようにするには、例えば以下のように、型引数を名前付きに変更する必要があります。

<!--
```rust
# #![feature(precise_capturing)]
# #![deny(impl_trait_overcaptures)]
fn f<'a, T: Sized>(x: &'a (), y: T) -> impl Sized + use<T> { (*x, y) }
//       ~~~~~~~~
// The type parameter has been named here.
#
# fn test<'a>(x: &'a (), y: ()) -> impl Sized + use<> {
#     f(x, y)
# }
```
-->

```rust
# #![feature(precise_capturing)]
# #![deny(impl_trait_overcaptures)]
fn f<'a, T: Sized>(x: &'a (), y: T) -> impl Sized + use<T> { (*x, y) }
//       ~~~~~~~~
// 型引数に名前がつきました。
#
# fn test<'a>(x: &'a (), y: ()) -> impl Sized + use<> {
#     f(x, y)
# }
```

<!--
Note that this changes the API of the function slightly as a type argument can now be explicitly provided for this parameter using turbofish syntax.  If this is undesired, you might consider instead whether you can simply continue to omit the `use<..>` bound and allow the lifetime to be captured.  This might be particularly desirable if you might in the future want to use that lifetime in the hidden type and would like to save space for that.
-->

なお、これは API の若干の変更にあたります。turbofish `::<>` 構文で型引数が明示的に指定できるようになったからです。
これを避けたい場合は、引き続き `use<..>` を明示しないことでライフタイムがキャプチャされるようになっても良いか判断する必要があります。
特に、隠蔽された型が将来的にライフタイムをキャプチャしうる余地を残したい場合は、そうした方がよいでしょう。

<!--
### Migrating away from the `Captures` trick
-->

### `Captures` パターンをやめる

<!--
Prior to the introduction of precise capturing `use<..>` bounds in Rust 1.82, correctly capturing a lifetime in an RPIT opaque type often required using the `Captures` trick.  E.g.:
-->

Rust 1.82 で `use<..>` 境界を用いた精密なキャプチャが使えるようになるまで、RPIT 不透明型がライフタイムを正しくキャプチャするためによく使われていたのが `Captures` パターンです。
例えば以下の通りです。

<!--
```rust
#[doc(hidden)]
pub trait Captures<T: ?Sized> {}
impl<T: ?Sized, U: ?Sized> Captures<T> for U {}

fn f<'a, T>(x: &'a (), y: T) -> impl Sized + Captures<(&'a (), T)> {
//                                           ~~~~~~~~~~~~~~~~~~~~~
//                            This is called the `Captures` trick.
    (x, y)
}
#
# fn test<'t, 'x>(t: &'t (), x: &'x ()) {
#     f(t, x);
# }
```
-->

```rust
#[doc(hidden)]
pub trait Captures<T: ?Sized> {}
impl<T: ?Sized, U: ?Sized> Captures<T> for U {}

fn f<'a, T>(x: &'a (), y: T) -> impl Sized + Captures<(&'a (), T)> {
//                                           ~~~~~~~~~~~~~~~~~~~~~
//                               これがいわゆる Captures パターンです。
    (x, y)
}
#
# fn test<'t, 'x>(t: &'t (), x: &'x ()) {
#     f(t, x);
# }
```

<!--
With the `use<..>` bound syntax, the `Captures` trick is no longer needed and can be replaced with the following in all editions:
-->

`use<..>` 境界構文が導入された今、`Captures` パターンを使う必要はなくなり、どのエディションでも以下のように書き換えることができるようになりました。

```rust
# #![feature(precise_capturing)]
fn f<'a, T>(x: &'a (), y: T) -> impl Sized + use<'a, T> {
    (x, y)
}
#
# fn test<'t, 'x>(t: &'t (), x: &'x ()) {
#     f(t, x);
# }
```

<!--
In Rust 2024, the `use<..>` bound can often be omitted entirely, and the above can be written simply as:
-->

Rust 2024 では `use<..>` はそもそも省略可能な場合が多いです。上記は以下のように書き換えられます。

```rust,edition2024
# #![feature(lifetime_capture_rules_2024)]
fn f<'a, T>(x: &'a (), y: T) -> impl Sized {
    (x, y)
}
#
# fn test<'t, 'x>(t: &'t (), x: &'x ()) {
#     f(t, x);
# }
```

<!--
There is no automatic migration for this, and the `Captures` trick still works in Rust 2024, but you might want to consider migrating code manually away from using this old trick.
-->

このような移行は自動ではなされず、また Rust 2024 でも `Captures` パターンは使用可能ですが、古い書き方からは脱却したほうがよいでしょう。

<!--
### Migrating away from the outlives trick
-->

### outlive パターンをやめる

<!--
Prior to the introduction of precise capturing `use<..>` bounds in Rust 1.82, it was common to use the "outlives trick" when a lifetime needed to be used in the hidden type of some opaque.  E.g.:
-->

Rust 1.82 で `use<..>` 境界を用いた精密なキャプチャが使えるようになるまで、不透明型に隠蔽された型がライフタイムを要求する場合に outlive パターンが広く使われていました。
例えば以下の通りです。

<!--
```rust
fn f<'a, T: 'a>(x: &'a (), y: T) -> impl Sized + 'a {
    //    ~~~~                                 ~~~~
    //    ^                     This is the outlives trick.
    //    |
    // This bound is needed only for the trick.
    (x, y)
//  ~~~~~~
// The hidden type is `(&'a (), T)`.
}
```
-->

```rust
fn f<'a, T: 'a>(x: &'a (), y: T) -> impl Sized + 'a {
    //    ~~~~                                 ~~~~
    //    ^          ここで outlive パターンを使っています。
    //    |
    // このパターンのためだけに、追加の境界が必要になっています。
    (x, y)
//  ~~~~~~
// 型 `(&'a (), T)` が隠蔽されています。
}
```

<!--
This trick was less baroque than the `Captures` trick, but also less correct.  As we can see in the example above, even though any lifetime components within `T` are independent from the lifetime `'a`, we're required to add a `T: 'a` bound in order to make the trick work.  This created undue and surprising restrictions on callers.
-->

このパターンは `Captures` パターンほど古の産物ではないですが、`Captures` パターンほど正しくもありませんでした。
上の例のように、`T` を構成するライフタイムは `'a` と無関係であるのに、パターンを適用するために `T: 'a` を指定せざるを得ないからです。
関数を使う側としては、思いがけない不当な制約となってしまいます。

<!--
Using precise capturing, you can write the above instead, in all editions, as:
-->

精密なキャプチャを使えば、上記のコードはどのエディションでも以下のように書き換えられます。

```rust
# #![feature(precise_capturing)]
fn f<T>(x: &(), y: T) -> impl Sized + use<'_, T> {
    (x, y)
}
#
# fn test<'t, 'x>(t: &'t (), x: &'x ()) {
#    f(t, x);
# }
```

<!--
In Rust 2024, the `use<..>` bound can often be omitted entirely, and the above can be written simply as:
-->

Rust 2024 では、 `use<..>` 境界は多くの場合で明示が不要なので、次のように書くこともできます。

```rust,edition2024
# #![feature(precise_capturing)]
# #![feature(lifetime_capture_rules_2024)]
fn f<T>(x: &(), y: T) -> impl Sized {
    (x, y)
}
#
# fn test<'t, 'x>(t: &'t (), x: &'x ()) {
#    f(t, x);
# }
```

<!--
There is no automatic migration for this, and the outlives trick still works in Rust 2024, but you might want to consider migrating code manually away from using this old trick.
-->

このような移行は自動ではなされず、また Rust 2024 でも outlive パターンは使用可能ですが、古い書き方からは脱却したほうがよいでしょう。
