<!--
# Disjoint capture in closures
-->

# クロージャはフィールドごとにキャプチャする
  <!-- 代案: クロージャのフィールドごとのキャプチャ -->
  <!-- 代案: クロージャの非交差キャプチャ -->
  <!-- 代案: クロージャの素(集合)キャプチャ -->

<!--
## Summary
-->

## 概要

<!--
- `|| a.x + 1` now captures only `a.x` instead of `a`.
- This can cause things to be dropped at different times or affect whether closures implement traits like `Send` or `Clone`.
  - If possible changes are detected, `cargo fix` will insert statements like `let _ = &a` to force a closure to capture the entire variable.
-->

- `|| a.x + 1` が `a` でなく `a.x` だけをキャプチャするようになりました。
- これにより、ドロップのタイミングが変わったり、クロージャが `Send` や `Clone` を実装するかどうかが変わったりします。
  - `cargo fix` は、可能だと判断したときに、 `let _ = &a` のような文を挿入して、クロージャが変数全体をキャプチャするように強制します。

<!--
## Details
-->

## 詳細

<!--
[Closures](https://doc.rust-lang.org/book/ch13-01-closures.html)
automatically capture anything that you refer to from within their body.
For example, `|| a + 1` automatically captures a reference to `a` from the surrounding context.
-->

[クロージャ](https://doc.rust-lang.org/book/ch13-01-closures.html)は、本体の中で使用しているすべてのものを自動的にキャプチャします。<!--TODO: 日本語版の TRPL に置き換える？ -->
例えば、`|| a + 1` と書くと、周囲のコンテキスト中の `a` への参照が自動的にキャプチャされます。

<!--
In Rust 2018 and before, closures capture entire variables, even if the closure only uses one field.
For example, `|| a.x + 1` captures a reference to `a` and not just `a.x`.
Capturing `a` in its entirety prevents mutation or moves from other fields of `a`, so that code like this does not compile:
-->

Rust 2018 以前では、クロージャに使われているのが1つのフィールドだけであっても、クロージャは変数全体をキャプチャします。
例えば、 `|| a.x + 1` は `a.x` への参照だけでなく、`a` への参照をキャプチャします。
`a` 全体がキャプチャされると、`a` の他のフィールドの値を書き換えたりムーブしたりできなくなります。従って以下のようなコードはコンパイルに失敗します:

<!--
```rust,ignore
let a = SomeStruct::new();
drop(a.x); // Move out of one field of the struct
println!("{}", a.y); // Ok: Still use another field of the struct
let c = || println!("{}", a.y); // Error: Tries to capture all of `a`
c();
```
-->

```rust,ignore
let a = SomeStruct::new();
drop(a.x); // 構造体のフィールドの1つをムーブする
println!("{}", a.y); // OK: 構造体の他のフィールドは、まだ使える
let c = || println!("{}", a.y); // エラー: `a` 全体をキャプチャしようとする
c();
```

<!--
Starting in Rust 2021, closures captures are more precise. Typically they will only capture the fields they use (in some cases, they might capture more than just what they use, see the Rust reference for full details). Therefore, the above example will compile fine in Rust 2021.
-->

Rust 2021 からは、クロージャのキャプチャはより精密になります。 特に、使用されるフィールドだけがキャプチャされるようになります
（場合によっては、使用する変数以外にもキャプチャすることもあり得ます。詳細に関しては Rust リファレンスを参照してください）。
したがって、上記のコードは Rust 2021 では問題ありません。

<!--
Disjoint capture was proposed as part of [RFC 2229](https://github.com/rust-lang/rfcs/blob/master/text/2229-capture-disjoint-fields.md) and the RFC contains details about the motivation.
-->

フィールドごとのキャプチャは [RFC 2229](https://github.com/rust-lang/rfcs/blob/master/text/2229-capture-disjoint-fields.md) の一部として提案されました。この RFC にはより詳しい動機が記載されています。

<!--
## Migration
-->

## 移行

<!--
As a part of the 2021 edition a migration lint, `rust_2021_incompatible_closure_captures`, has been added in order to aid in automatic migration of Rust 2018 codebases to Rust 2021.
-->

Rust 2018 から Rust 2021 への自動移行の支援のため、2021 エディションには、移行用のリント`rust_2021_incompatible_closure_captures` が追加されています。

<!--
In order to have `rustfix` migrate your code to be Rust 2021 Edition compatible, run:
-->

`rustfix` でコードを Rust 2021 エディションに適合させるためには、次のように実行します。

```sh
cargo fix --edition
```

<!--
Below is an examination of how to manually migrate code to use closure captures that are compatible with Rust 2021 should the automatic migration fail 
or you would like to better understand how the migration works.
-->

以下では、クロージャによるキャプチャが出現するコードについて、自動移行が失敗した場合に手動で Rust 2021 に適合するように移行するにはどうすればいいかを考察します。
移行がどのようになされるか知りたい人も以下をお読みください。

<!--
Changing the variables captured by a closure can cause programs to change behavior or to stop compiling in two cases:
-->

クロージャによってキャプチャされる変数が変わると、プログラムの挙動が変わったりコンパイルできなくなったりすることがありますが、その原因は以下の2つです:

<!--
- changes to drop order, or when destructors run ([details](#drop-order));
- changes to which traits a closure implements ([details](#trait-implementations)).
-->

- ドロップの順序や、デストラクタが走るタイミングが変わる場合（[詳細](#drop-order)）
- クロージャが実装するトレイトが変わる場合（[詳細](#drop-order)）

<!--
Whenever any of the scenarios below are detected, `cargo fix` will insert a "dummy let" into your closure to force it to capture the entire variable:
-->

以下のような状況を検知すると、`cargo fix` は "ダミーの let" をクロージャの中に挿入して、強制的に全ての変数がキャプチャされるようにします:

<!--
```rust
let x = (vec![22], vec![23]);
let c = move || {
    // "Dummy let" that forces `x` to be captured in its entirety
    let _ = &x;

    // Otherwise, only `x.0` would be captured here
    println!("{:?}", x.0);
};
```
-->

```rust
let x = (vec![22], vec![23]);
let c = move || {
    // `x` 全体が強制的にキャプチャされるための "ダミーの let"
    let _ = &x;

    // それがないと、`x.0` だけがここでキャプチャされる
    println!("{:?}", x.0);
};
```

<!--
This is a conservative analysis: in many cases, these dummy lets can be safely removed and your program will work fine.
-->

この解析は保守的です。ほとんどの場合、ダミーの let は問題なく消すことができ、消してもプログラムはきちんと動きます。

<!--
### Wild Card Patterns
-->

### ワイルドカードパターン

<!--
Closures now only capture data that needs to be read, which means the following closures will not capture `x`:
-->

クロージャは本当に読む必要のあるデータだけをキャプチャするようになったので、次のコードは `x` をキャプチャしません:

<!--
```rust
let x = 10;
let c = || {
    let _ = x; // no-op
};

let c = || match x {
    _ => println!("Hello World!")
};
```
-->

```rust
let x = 10;
let c = || {
    let _ = x; // 何もしない
};

let c = || match x {
    _ => println!("Hello World!")
};
```

<!--
The `let _ = x` statement here is a no-op, since the `_` pattern completely ignores the right-hand side, and `x` is a reference to a place in memory (in this case, a variable).
-->

`_` パターンは右辺を無視するので、この `let _ = x` は何もせず、`x` はメモリ上のある場所（この場合は変数）を指し示します<!--TODO: 最後どういう意味なんでしょうか。これで訳合ってますか？-->。

<!--
This change by itself (capturing fewer values) doesn't trigger any suggestions, but it may do so in conjunction with the "drop order" change below.
-->

この変更（いくつかの値がキャプチャされなくなること）そのものによってコード変更の提案がなされることはありませんが、後で説明する「ドロップ順序」の変更と組み合わせると、提案がなされる場合もあります。

<!--
**Subtle:** There are other similar expressions, such as the "dummy lets" `let _ = &x` that we insert, which are not no-ops. This is because the right-hand side (`&x`) is not a reference to a place in memory, but rather an expression that must first be evaluated (and whose result is then discarded).
-->

**ちなみに:** 似たような式の中には、同じく自動挿入される "ダミーの let" であっても、`let _ = &x` のように「何もしない」わけではない文もあります。なぜかというと、右辺（`&x`）はメモリ上のある場所を指し示すのではなく、値が評価されるべき式となるからです（その評価結果は捨てられますが）。

<!--
### Drop Order
-->

### ドロップの順序

<!--
When a closure takes ownership of a value from a variable `t`, that value is then dropped when the closure is dropped, and not when the variable `t` goes out of scope:
-->

クロージャが変数 `t` の所有権を取るとき、`t` がドロップされるのは `t` がスコープ外に出たときではなく、そのクロージャがドロップされたときになります:

<!--
```rust
# fn move_value<T>(_: T){}
{
    let t = (vec![0], vec![0]);

    {
        let c = || move_value(t); // t is moved here
    } // c is dropped, which drops the tuple `t` as well
} // t goes out of scope here
```
-->

```rust
# fn move_value<T>(_: T){}
{
    let t = (vec![0], vec![0]);

    {
        let c = || move_value(t); // t はここでムーブされる
    } // c がドロップされ、そのときにタプル `t` もまたドロップされる
} // t はここでスコープを抜ける
```

<!--
The above code will run the same in both Rust 2018 and Rust 2021. However, in cases where the closure only takes ownership of _part_ of a variable, there can be differences:
-->

上記のコードの挙動は Rust 2018 と Rust 2021 で同じです。ところが、クロージャが変数の<!-- -->_一部_<!-- -->の所有権を取るとき、違いが発生します:

<!--
```rust
# fn move_value<T>(_: T){}
{
    let t = (vec![0], vec![0]);

    {
        let c = || {
            // In Rust 2018, captures all of `t`.
            // In Rust 2021, captures only `t.0`
            move_value(t.0);
        };

        // In Rust 2018, `c` (and `t`) are both dropped when we
        // exit this block.
        //
        // In Rust 2021, `c` and `t.0` are both dropped when we
        // exit this block.
    }

// In Rust 2018, the value from `t` has been moved and is
// not dropped.
//
// In Rust 2021, the value from `t.0` has been moved, but `t.1`
// remains, so it will be dropped here.
}
```
-->

```rust
# fn move_value<T>(_: T){}
{
    let t = (vec![0], vec![0]);

    {
        let c = || {
            // Rust 2018 では、`t` 全体がキャプチャされる。
            // Rust 2018 では、`t.0` だけがキャプチャされる
            move_value(t.0);
        };

        // Rust 2018 では、 `c` (と `t`) の両方が
        // このブロックを抜けるときにドロップされる。
        //
        // Rust 2021 では、 `c` と `t.0` の両方が
        // このブロックを抜けるときにドロップされる。
    }

// Rust 2018 では、`t` はすでにムーブされており、ここではドロップされない
//
// Rust 2021 では、`t.0` はムーブされているが、
// `t.1` は残っており、ここでドロップされる
}
```

<!--
In most cases, dropping values at different times just affects when memory is freed and is not important. However, some `Drop` impls (aka, destructors) have side-effects, and changing the drop order in those cases can alter the semantics of your program. In such cases, the compiler will suggest inserting a dummy `let` to force the entire variable to be captured.
-->

ほとんどの場合、ドロップのタイミングが変わってもメモリが解放されるタイミングが変わるだけで、さほど問題にはなりません。
しかし、`Drop` の実装に副作用のある（いわゆるデストラクタである）場合、ドロップの順序が変わるとプログラムの意味も変わってしまいます。
その場合は、コンパイラはダミーの `let` を挿入して変数全体がキャプチャされるように提案します。

<!--
### Trait implementations
-->

### トレイト実装

<!--
Closures automatically implement the following traits based on what values they capture:
-->

何がキャプチャされているかによって、クロージャには自動的に以下のトレイトが実装されます:

<!--
- [`Clone`]: if all captured values are [`Clone`].
- [Auto traits] like [`Send`], [`Sync`], and [`UnwindSafe`]: if all captured values implement the given trait.
-->

- [`Clone`]: キャプチャされた値がすべて [`Clone`] を実装していた場合。
- [`Send`], [`Sync`], [`UnwindSafe`] などの[自動トレイト]: キャプチャされた値がすべてそのトレイトを実装していた場合。

<!--
[auto traits]: https://doc.rust-lang.org/nightly/reference/special-types-and-traits.html#auto-traits
[`clone`]: https://doc.rust-lang.org/std/clone/trait.Clone.html
[`send`]: https://doc.rust-lang.org/std/marker/trait.Send.html
[`sync`]: https://doc.rust-lang.org/std/marker/trait.Sync.html
[`unwindsafe`]: https://doc.rust-lang.org/std/marker/trait.UnwindSafe.html
-->

[自動トレイト]: https://doc.rust-lang.org/nightly/reference/special-types-and-traits.html#auto-traits
[`clone`]: https://doc.rust-lang.org/std/clone/trait.Clone.html
[`send`]: https://doc.rust-lang.org/std/marker/trait.Send.html
[`sync`]: https://doc.rust-lang.org/std/marker/trait.Sync.html
[`unwindsafe`]: https://doc.rust-lang.org/std/marker/trait.UnwindSafe.html

<!--
In Rust 2021, since different values are being captured, this can affect what traits a closure will implement. The migration lints test each closure to see whether it would have implemented a given trait before and whether it still implements it now; if they find that a trait used to be implemented but no longer is, then "dummy lets" are inserted.
-->

Rust 2021 では、キャプチャされる値が変わることによって、クロージャが実装するトレイトも変わることがあります。
先ほどの移行リントは、それぞれのクロージャについて、これまで実装されていた自動トレイトが何であるか、そして移行後もそれらが残るかどうかを調べます。
もし今まで実装されていたトレイトが実装されなくなる場合、「ダミーの let」が挿入されます。

<!--
For instance, a common way to allow passing around raw pointers between threads is to wrap them in a struct and then implement `Send`/`Sync` auto trait for the wrapper. The closure that is passed to `thread::spawn` uses the specific fields within the wrapper but the entire wrapper is captured regardless. Since the wrapper is `Send`/`Sync`, the code is considered safe and therefore compiles successfully.
-->

例えば、スレッド間で生ポインタを受け渡しする一般的な方法に、ポインタを構造体でラップし、そのラッパー構造体に自動トレイト `Send`/`Sync` を実装するというものがあります。
`thread::spawn` に渡されるクロージャが使うのは、ラッパー構造体のうち特定の変数だけですが、キャプチャされるのはラッパー構造体全体です。
ラッパー構造体は `Send`/`Sync` なので、コードは安全であるとみなされ、コンパイルは成功します。

<!--
With disjoint captures, only the specific field mentioned in the closure gets captured, which wasn't originally `Send`/`Sync` defeating the purpose of the wrapper.
-->

フィールドごとのキャプチャが導入されると、キャプチャ内で使用されているフィールドだけがキャプチャされますが、フィールドの中身はもともと `Send`/`Sync` でなかったのですから、せっかくラッパーを作っても元の木阿弥です。

<!--
```rust
use std::thread;

struct Ptr(*mut i32);
unsafe impl Send for Ptr {}


let mut x = 5;
let px = Ptr(&mut x as *mut i32);

let c = thread::spawn(move || {
    unsafe {
        *(px.0) += 10;
    }
}); // Closure captured px.0 which is not Send
```
-->

```rust
use std::thread;

struct Ptr(*mut i32);
unsafe impl Send for Ptr {}


let mut x = 5;
let px = Ptr(&mut x as *mut i32);

let c = thread::spawn(move || {
    unsafe {
        *(px.0) += 10;
    }
}); // クロージャは px.0 をキャプチャしたが、これは Send ではない
```
