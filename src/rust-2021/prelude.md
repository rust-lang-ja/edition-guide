<!--
# Additions to the prelude
-->

# Prelude への追加

<!--
## Summary
-->

## 概要

<!--
- The `TryInto`, `TryFrom` and `FromIterator` traits are now part of the prelude.
- This might make calls to trait methods ambiguous which could make some code fail to compile.
-->

- `TryInto`, `TryFrom`, `FromIterator` がプレリュードに追加されました。
- これにより、トレイトメソッドへの呼び出しに曖昧性が発生して、コンパイルに失敗するようになるコードがあるかもしれません。

<!--
## Details
-->

## 詳細

<!--
The [prelude of the standard library](https://doc.rust-lang.org/stable/std/prelude/index.html)
is the module containing everything that is automatically imported in every module.
It contains commonly used items such as `Option`, `Vec`, `drop`, and `Clone`.
-->

[標準ライブラリの prelude](https://doc.rust-lang.org/stable/std/prelude/index.html) モジュールには、
すべてのモジュールにインポートされるものが余すことなく定義されています。
そこには、`Option`, `Vec`, `drop`, `Clone` などの、頻繁に使われるアイテムが含まれます。

<!--
The Rust compiler prioritizes any manually imported items over those
from the prelude, to make sure additions to the prelude will not break any existing code.
For example, if you have a crate or module called `example` containing a `pub struct Option;`,
then `use example::*;` will make `Option` unambiguously refer to the one from `example`;
not the one from the standard library.
-->

Rust コンパイラは、手動で<!-- TODO: 明示的に、のほうがいいか？-->インポートされたアイテムをプレリュードからのものより優先します。
これにより、プレリュードに追加があっても既存のコードは壊れないようになっています。
たとえば、 `example` という名前のクレートに `pub struct Option;` が含まれていたら、
`use example::*;` とすることで `Option` は曖昧性なく `example` に含まれるものを指し示し、
標準ライブラリのものは意味しません。

<!--
However, adding a _trait_ to the prelude can break existing code in a subtle way.
For example, a call to `x.try_into()` which comes from a `MyTryInto` trait might fail 
to compile if `std`'s `TryInto` is also imported, because the call to `try_into` is now 
ambiguous and could come from either trait. This is the reason we haven't added `TryInto` 
to the prelude yet, since there is a lot of code that would break this way.
-->

ところが、_トレイト_<!-- -->をプレリュードに追加すると、捉えがたい形でコードが壊れることがあります。
たとえば、`MyTryInto` トレイトで定義されている `x.try_into()` という呼び出しは、
`std` の `TryInto` もインポートされているときは、動かなくなる場合があります。
なぜなら、`try_into` の呼び出しは今や曖昧で、どちらのトレイトから来ているかわからないからです。
だからこそ我々は、このような問題が起こりうるコードが沢山あるがために、
`TryInto` を未だにプレリュードに追加していませんでした。 <!--
代案: だからこそ我々は、 `TryInto` を未だにプレリュードに追加していませんでした。
追加してしまうと、多くのコードでそのような問題が起こりうるからです。 -->

<!--
As a solution, Rust 2021 will use a new prelude.
It's identical to the current one, except for three new additions:
-->

解決策として、Rust 2021 では新たなプレリュードが使用されます。
変更点は、以下の3つが追加されたということだけです。

- [`std::convert::TryInto`](https://doc.rust-lang.org/stable/std/convert/trait.TryInto.html)
- [`std::convert::TryFrom`](https://doc.rust-lang.org/stable/std/convert/trait.TryFrom.html)
- [`std::iter::FromIterator`](https://doc.rust-lang.org/stable/std/iter/trait.FromIterator.html)

<!--
The tracking issue [can be found here](https://github.com/rust-lang/rust/issues/85684).
-->

Tracking issue は[こちら](https://github.com/rust-lang/rust/issues/85684)です。

<!--
## Migration 
-->

## 移行

<!--
As a part of the 2021 edition a migration lint, `rust_2021_prelude_collisions`, has been added in order to aid in automatic migration of Rust 2018 codebases to Rust 2021.
-->

Rust 2018 から Rust 2021 への自動移行の支援のため、2021 エディションには、移行用のリント`rust_2021_prelude_collisions` が追加されています。

<!--
In order to have `rustfix` migrate your code to be Rust 2021 Edition compatible, run:
-->

`rustfix` でコードを Rust 2021 エディションに適合させるためには、次のように実行します。

```sh
cargo fix --edition
```

<!--
The lint detects cases where functions or methods are called that have the same name as the methods defined in one of the new prelude traits. In some cases, it may rewrite your calls in various ways to ensure that you continue to call the same function you did before.
-->

このリントは、新しくプレリュードに追加されたトレイトで定義されているメソッドと同名の関数やメソッドが呼び出されていることを検知します。
場合によっては、今までと同じ関数が呼び出されるように、あなたのコードを様々な方法で書き換えることもあります。

<!--
If you'd like to migrate your code manually or better understand what `rustfix` is doing, below we've outlined the situations where a migration is needed along with a counter example of when it's not needed.
-->

コードの移行を手作業で行いたい方や `rustfix` が何を行うかをより詳しく理解したい方のために、どのような状況で移行が必要なのか、逆にどうであれば不要なのを以下に例示していきます。

<!--
### Migration needed
-->

### 移行が必要な場合

<!--
#### Conflicting trait methods
-->

#### トレイトメソッドの衝突

<!--
When two traits that are in scope have the same method name, it is ambiguous which trait method should be used. For example:
-->

あるスコープに、同じメソッド名を持つ2つのトレイトがある場合、どちらのメソッドが使用されるべきかは曖昧です。例えば：


<!--
```rust
trait MyTrait<A> {
  // This name is the same as the `from_iter` method on the `FromIterator` trait from `std`.  
  fn from_iter(x: Option<A>);
}

impl<T> MyTrait<()> for Vec<T> {
  fn from_iter(_: Option<()>) {}
}

fn main() {
  // Vec<T> implements both `std::iter::FromIterator` and `MyTrait` 
  // If both traits are in scope (as would be the case in Rust 2021),
  // then it becomes ambiguous which `from_iter` method to call
  <Vec<i32>>::from_iter(None);
}
```
-->

```rust
trait MyTrait<A> {
  // この関数名は、`std` の `FromIterator` トレイトの `from_iter` メソッドと同名です。
  fn from_iter(x: Option<A>);
}

impl<T> MyTrait<()> for Vec<T> {
  fn from_iter(_: Option<()>) {}
}

fn main() {
  // Vec<T> は `std::iter::FromIterator` と `MyTrait` の両方を実装します
  // もし両方のトレイトがスコープに含まれる場合 (Rust 2021 ではそうなのですが)、
  // どちらの `from_iter` メソッドを呼び出せばいいかが曖昧になります
  <Vec<i32>>::from_iter(None);
}
```

<!--
We can fix this by using fully qualified syntax:
-->

完全修飾構文を使えば、これを修正することができます:

<!--
```rust,ignore
fn main() {
  // Now it is clear which trait method we're referring to
  <Vec<i32> as MyTrait<()>>::from_iter(None);
}
```
-->

```rust,ignore
fn main() {
  // こうすれば、どちらのトレイトメソッドを指し示しているかが明確になります
  <Vec<i32> as MyTrait<()>>::from_iter(None);
}
```

<!--
#### Inherent methods on `dyn Trait` objects
-->

#### `dyn Trait` オブジェクトの固有メソッド

<!--
Some users invoke methods on a `dyn Trait` value where the method name overlaps with a new prelude trait:
-->

`dyn Trait` の値に対してメソッドを呼び出すときに、メソッド名が新しくプレリュードに追加されたトレイトと重複していることがあります:

<!--
```rust
mod submodule {
  pub trait MyTrait {
    // This has the same name as `TryInto::try_into`
    fn try_into(&self) -> Result<u32, ()>;
  }
}

// `MyTrait` isn't in scope here and can only be referred to through the path `submodule::MyTrait`
fn bar(f: Box<dyn submodule::MyTrait>) {
  // If `std::convert::TryInto` is in scope (as would be the case in Rust 2021),
  // then it becomes ambiguous which `try_into` method to call
  f.try_into();
}
```
-->

```rust
mod submodule {
  pub trait MyTrait {
    // これは、 `TryInto::try_into` と同じ名前です
    fn try_into(&self) -> Result<u32, ()>;
  }
}

// `MyTrait` はここではスコープ内になく、パス付きで `submodule::MyTrait` としか利用できません
fn bar(f: Box<dyn submodule::MyTrait>) {
  // `std::convert::TryInto` がスコープ内にあるときは (Rust 2021 ではそうなのですが)、
  // どちらの `try_into` メソッドを呼び出せばいいかが曖昧になります
  f.try_into();
}
```

<!--
Unlike with static dispatch methods, calling a trait method on a trait object does not require that the trait be in scope. The code above works 
as long as there is no trait in scope with a conflicting method name. When the `TryInto` trait is in scope (which is the case in Rust 2021),
this causes an ambiguity. Should the call be to `MyTrait::try_into` or `std::convert::TryInto::try_into`?
-->

静的ディスパッチのときと違って、トレイトオブジェクトに対してトレイトメソッドを呼び出すときは、そのトレイトがスコープ内にある必要はありません。
`TryInto` トレイトがスコープ内にあるときは (Rust 2021 ではそうなのですが)、曖昧性が発生します。
`MyTrait::try_into` と `std::convert::TryInto::try_into` のどちらが呼び出されるべきなのでしょうか？

<!--
In these cases, we can fix this by adding an additional dereferences or otherwise clarify the type of the method receiver. This ensures that 
the `dyn Trait` method is chosen, versus the methods from the prelude trait. For example, turning `f.try_into()` above into `(&*f).try_into()` 
ensures that we're calling `try_into` on the `dyn MyTrait` which can only refer to the `MyTrait::try_into` method.
-->

この場合、さらなる参照外しをするか、もしくはメソッドレシーバーの型を明示することで修正できます。
これにより、`dyn Trait` のメソッドとプレリュードのトレイトのメソッドのどちらが選ばれているかが明確になります。
たとえば、上の `f.try_into()` を `(&*f).try_into()` にすれば、<!-- 意訳。ここの部分の意味がわかりません-->確実に `MyTrait::try_into` を指し示しているとわかるので<!--ここまで-->、
`dyn MyTrait` の方の `try_into` が呼び出されていると決定されます。

<!--
### No migration needed
-->

### 移行が不要な場合

<!--
####  Inherent methods
-->

####  固有メソッド

<!--
Many types define their own inherent methods with the same name as a trait method. For instance, below the struct `MyStruct` implements `from_iter` which shares the same name with the method from the trait `FromIterator` found in the standard library:
-->

トレイトメソッドと同名の固有メソッドを定義しているような型もたくさんあります。
たとえば、以下では `MyStruct` が `from_iter` を実装していますが、
これは標準ライブラリの `FromIterator` トレイトのメソッドと同名です。

<!--
```rust
use std::iter::IntoIterator;

struct MyStruct {
  data: Vec<u32>
}

impl MyStruct {
  // This has the same name as `std::iter::FromIterator::from_iter`
  fn from_iter(iter: impl IntoIterator<Item = u32>) -> Self {
    Self {
      data: iter.into_iter().collect()
    }
  }
}

impl std::iter::FromIterator<u32> for MyStruct {
    fn from_iter<I: IntoIterator<Item = u32>>(iter: I) -> Self {
      Self {
        data: iter.into_iter().collect()
      }
    }
}
```
-->

```rust
use std::iter::IntoIterator;

struct MyStruct {
  data: Vec<u32>
}

impl MyStruct {
  // これは `std::iter::FromIterator::from_iter` と同名
  fn from_iter(iter: impl IntoIterator<Item = u32>) -> Self {
    Self {
      data: iter.into_iter().collect()
    }
  }
}

impl std::iter::FromIterator<u32> for MyStruct {
    fn from_iter<I: IntoIterator<Item = u32>>(iter: I) -> Self {
      Self {
        data: iter.into_iter().collect()
      }
    }
}
```

<!--
Inherent methods always take precedent over trait methods so there's no need for any migration.
-->

固有メソッドは常にトレイトメソッドより優先されるため、移行作業の必要はありません。

<!--
### Implementation Reference
-->

### 実装の参考事項

<!--
The lint needs to take a couple of factors into account when determining whether or not introducing 2021 Edition to a codebase will cause a name resolution collision (thus breaking the code after changing edition). These factors include:
-->

2021 エディションを導入することで名前解決に衝突が生じるかどうか（すなわち、エディションを変えることでコードが壊れるかどうか）を判断するために、このリントはいくつかの要素を考慮する必要があります。たとえば以下のような点です:

<!--
- Is the call a [fully-qualified call] or does it use [dot-call method syntax]?
  - This will affect how the name is resolved due to auto-reference and auto-dereferencing on method call syntax. Manually dereferencing/referencing will allow specifying priority in the case of dot-call method syntax, while fully-qualified call requires specification of the type and the trait name in the method path (e.g. `<Type as Trait>::method`)
- Is this an [inherent method] or [a trait method]?
  - Inherent methods that take `self` will take priority over `TryInto::try_into` as inherent methods take priority over trait methods, but inherent methods that take `&self` or `&mut self` won't take priority due to requiring a auto-reference (while `TryInto::try_into` does not, as it takes `self`)
- Is the origin of this method from `core`/`std`? (As the traits can't have a collision with themselves)
- Does the given type implement the trait it could have a collision against?
- Is the method being called via dynamic dispatch? (i.e. is the `self` type `dyn Trait`)
  - If so, trait imports don't affect resolution, and no migration lint needs to occur
-->

- [完全修飾呼び出し]と[ドット呼び出しメソッド構文]のどちらが使われているか？
  - これは、メソッド呼び出し構文において、自動参照付けと自動参照外しによってどのように名前解決がなされるかに影響します<!--TODO: due to がどこに係るのかわかりません -->。ドット呼び出しメソッド構文では、手動で参照外し/参照付けすることで優先順位を決められますが、完全修飾呼び出しではメソッドパス中に型とトレイト名が指定されていなければなりません (例: `<Type as Trait>::method`)
- [固有メソッド]と[トレイトメソッド]のどちらが呼び出されているか？
  - 固有メソッドはトレイトメソッドより優先されるので、`self` を取るトレイトメソッドは、`TryInto::try_into`より優先されますが、`&self` や `&mut self` をとる固有メソッドは、自動参照付けが必要なので優先されません（もっとも、`TryInto` は `self` を取るので、それは当てはまりませんが）
- そのメソッドは `core` か `std` から来たものか？　（トレイトは自分自身とは衝突しないので）
- その型は、名前が衝突するようなトレイトを実装しているか？
- メソッドが動的ディスパッチによって呼び出されているか？  （つまり、それは `self` 型の `dyn Trait` か？）
  - その場合、トレイトのインポートは名前解決に影響しないので、移行リントを出す必要はありません

<!--
[fully-qualified call]: https://doc.rust-lang.org/reference/expressions/call-expr.html#disambiguating-function-calls
[dot-call method syntax]: https://doc.rust-lang.org/reference/expressions/method-call-expr.html
[inherent method]: https://doc.rust-lang.org/reference/items/implementations.html#inherent-implementations
[a trait method]: https://doc.rust-lang.org/reference/items/implementations.html#trait-implementations
-->

[完全修飾呼び出し]: https://doc.rust-lang.org/reference/expressions/call-expr.html#disambiguating-function-calls
[ドット呼び出しメソッド構文]: https://doc.rust-lang.org/reference/expressions/method-call-expr.html
[固有メソッド]: https://doc.rust-lang.org/reference/items/implementations.html#inherent-implementations
[トレイトメソッド]: https://doc.rust-lang.org/reference/items/implementations.html#trait-implementations
