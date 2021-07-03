# `const fn`

<!--
Initially added: ![Minimum Rust version: 1.31](https://img.shields.io/badge/Minimum%20Rust%20Version-1.31-brightgreen.svg)
-->
![Minimum Rust version: 1.31](https://img.shields.io/badge/Minimum%20Rust%20Version-1.31-brightgreen.svg)で最初に導入されました。

<!--
Expanded in many releases, see each aspect below for more details.
-->
そしてその後のリリースで何度か拡張されています。詳細は以下を確認してください。

<!--
A `const fn` allows you to execute code in a "const context." For example:
-->
`const fn`によりコードを「constコンテキスト」で実行することができます(訳注：つまり、コンパイル時に評価されます)。例を示します。

```rust
const fn five() -> i32 {
    5
}

const FIVE: i32 = five();
```

<!--
You cannot execute arbitrary code; the reasons why boil down to "you can
destroy the type system." The details are a bit too much to put here, but the
core idea is that `const fn` started off allowing the absolutely minimal
subset of the language, and has slowly added more abilities over time.
Therefore, while you can create a `const fn` in Rust 1.31, you cannot do much
with it. This is why we didn't add `const fn` to the Rust 2018 section; it
truly didn't become useful until after the release of the 2018 edition. This
means that if you read this document top to bottom, the earlier versions may
describe restrictions that are relaxed in later versions.

-->
const fnでは任意のコードを実行することは出来ません。
というのは、簡単に言うと型システムを壊してしまうことになるからです。
詳細はここでは述べませんが、`const fn`では最小限の言語サブセットのみを使えるところからスタートし、徐々に出来ることを増やしていく、というのが基本的な考え方です。
したがって、導入当初のRust 1.31では `const fn`関数を作ることは出来ましたが、実際に出来ることはほとんどありませんでした。
これが、`const fn`をRust 2018のセクションに含めなかった理由なのですが、実際、Rust 2018エディションのリリース時にはあまり使い物になりませんでした。
もしあなたがこのドキュメントを最初から最後まで読んだとすると、以前のバージョンで制約とされていた部分がその後のバージョンで緩和されていることがわかるでしょう。

<!--
Additionally, this has allowed more and more of the standard library to be
made `const`, we won't put all of those changes here, but you should know
that it is becoming more `const` over time.
-->
さらに付け加えると、`const fn`で利用できる表記方が増えるに従って、標準ライブラリの機能が`const`化されていっています。
ここではその変更の全てを述べることはしませんが、時とともに `const`化される部分が増えていくことは知っておいても良いでしょう。

<!--
## Arithmetic and comparison operators on integers
-->
## 整数の算術演算および比較演算

![Minimum Rust version: 1.31](https://img.shields.io/badge/Minimum%20Rust%20Version-1.31-brightgreen.svg)

<!--
You can do arithmetic on integer literals:
-->
整数リテラルに対して算術演算をすることができます。

```rust
const fn foo() -> i32 {
    5 + 6
}
```

<!--
## Many boolean operators
-->
## 多くのブーリアン演算

![Minimum Rust version: 1.31](https://img.shields.io/badge/Minimum%20Rust%20Version-1.31-brightgreen.svg)

<!--
You can use boolean operators other than `&&` and `||`, because they short-circut evaluation:
-->
ブーリアン演算のほとんどが使えます。
例外は`&&`と`||`で、これらは短絡評価を行うので使えなくなっています。

```rust
const fn mask(val: u8) -> u8 {
    let mask = 0x0f;

    mask & val
}
```

<!--
## Constructing arrays, structs, enums, and tuples
-->
## 配列、構造体、列挙型、タプルの生成

![Minimum Rust version: 1.31](https://img.shields.io/badge/Minimum%20Rust%20Version-1.31-brightgreen.svg)

<!--
You can create arrays, structs, enums, and tuples:
-->
配列、構造体、列挙型、タプルを生成することができます。

```rust
struct Point {
    x: i32,
    y: i32,
}

enum Error {
    Incorrect,
    FileNotFound,
}

const fn foo() {
    let array = [1, 2, 3];

    let point = Point {
        x: 5,
        y: 10,
    };

    let error = Error::FileNotFound;

    let tuple = (1, 2, 3);
}
```

<!--
## Calls to other const fns
-->
## 他のconst関数の呼び出し

![Minimum Rust version: 1.31](https://img.shields.io/badge/Minimum%20Rust%20Version-1.31-brightgreen.svg)

<!--
You can call `const fn` from a `const fn`:
-->
`const fn`から`const fn`を呼び出すことができます。


```rust
const fn foo() -> i32 {
    5
}

const fn bar() -> i32 {
    foo()
}
```

<!--
## Index expressions on arrays and slices
-->
## 配列やスライスの添字式

![Minimum Rust version: 1.31](https://img.shields.io/badge/Minimum%20Rust%20Version-1.31-brightgreen.svg)

<!--
You can index into an array or slice:
-->
配列やスライスに添え字アクセスできます。

```rust
const fn foo() -> i32 {
    let array = [1, 2, 3];

    array[1]
}
```

<!--
## Field accesses on structs and tuples
-->
## 構造体やタプルのフィールドアクセス

![Minimum Rust version: 1.31](https://img.shields.io/badge/Minimum%20Rust%20Version-1.31-brightgreen.svg)

<!--
You can access parts of a struct or tuple:
-->
構造体やタプルの構成要素にアクセスできます。

```rust
struct Point {
    x: i32,
    y: i32,
}

const fn foo() {
    let point = Point {
        x: 5,
        y: 10,
    };

    let tuple = (1, 2, 3);

    point.x;
    tuple.0;
}
```

<!--
## Reading from constants
-->
## 定数読み出し

![Minimum Rust version: 1.31](https://img.shields.io/badge/Minimum%20Rust%20Version-1.31-brightgreen.svg)

<!--
You can read from a constant:
-->
定数の値を読み出せます。

```rust
const FOO: i32 = 5;

const fn foo() -> i32 {
    FOO
}
```

<!--
Note that this is *only* `const`, not `static`.
-->
読み出せるのは `const`指定された定数*のみ*で、`static`指定された変数は読み出せません。

<!--
## & and * of references
-->
## 参照の & と *

![Minimum Rust version: 1.31](https://img.shields.io/badge/Minimum%20Rust%20Version-1.31-brightgreen.svg)

<!--
You can create and de-reference references:
-->
参照を作ったり、参照外しをすることができます。

```rust
const fn foo(r: &i32) {
    *r;

    &5;
}
```

<!--
## Casts, except for raw pointer to integer casts
-->
## キャスト。ただし生ポインタから整数へのキャストは除く

![Minimum Rust version: 1.31](https://img.shields.io/badge/Minimum%20Rust%20Version-1.31-brightgreen.svg)

<!--
You may cast things, except for raw pointers may not be casted to an integer:
-->
キャストができます。
例外は生ポインタから整数へのキャスト。

```rust
const fn foo() {
    let x: usize = 5;

    x as i32;
}
```

<!--
## Irrefutable destructuring patterns
-->
## 論駁不可能な分配パターン

![Minimum Rust version: 1.33](https://img.shields.io/badge/Minimum%20Rust%20Version-1.33-brightgreen.svg)

<!--
You can use irrefutable patterns that destructure values. For example:
-->
論駁不可能なパターンで値を分配することができます。

```rust
const fn foo((x, y): (u8, u8)) {
    // ...
}
```

<!--
Here, `foo` destructures the tuple into `x` and `y`. `if let` is another
place that uses irrefutable patterns.
-->
ここで、`foo`はタプルの値を`x`と`y`に分配します。
`if let`でも同様に論駁不可能なパターンを使います。

<!--
## `let` bindings
-->
## `let` 束縛

![Minimum Rust version: 1.33](https://img.shields.io/badge/Minimum%20Rust%20Version-1.33-brightgreen.svg)

<!--
You can use both mutable and immutable `let` bindings:
-->
ミュータブルとイミュータブルの双方の`let`束縛を使うことができます。

```rust
const fn foo() {
    let x = 5;
    let mut y = 10;
}
```

<!--
## Assignment
-->
## 代入

![Minimum Rust version: 1.33](https://img.shields.io/badge/Minimum%20Rust%20Version-1.33-brightgreen.svg)

<!--
You can use assignment and assignment operators:
-->
代入と代入演算子を使うことができます。

```rust
const fn foo() {
    let mut x = 5;
    x = 10;
}
```

<!--
## Calling `unsafe fn`
-->
## `unsafe fn` の呼び出し

![Minimum Rust version: 1.33](https://img.shields.io/badge/Minimum%20Rust%20Version-1.33-brightgreen.svg)

<!--
You can call an `unsafe fn` inside a `const fn`:
-->
`const fn`の中で`unsafe fn`を呼び出すことができます。

```rust
const unsafe fn foo() -> i32 { 5 }

const fn bar() -> i32 {
    unsafe { foo() }
}
```
