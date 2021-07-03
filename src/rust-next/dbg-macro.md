<!--
# The dbg! macro
-->
# dbg! マクロ

![Minimum Rust version: 1.32](https://img.shields.io/badge/Minimum%20Rust%20Version-1.32-brightgreen.svg)

<!--
The `dbg!` macro provides a nicer experience for debugging than `println!`:
-->
`dbg!`マクロは `println!`よりも、より良いデバッグ体験を提供します。


```rust
fn main() {
    let x = 5;

    dbg!(x);
}
```

<!--
If you run this program, you'll see:
-->
このプログラムを実行すると、次の出力が表示されます。

```text
[src/main.rs:4] x = 5
```

<!--
You get the file and line number of where this was invoked, as well as the
name and value. Additionally, `println!` prints to the standard output, so you
really should be using `eprintln!` to print to standard error. `dbg!` does the
right thing and goes to stderr.
-->
これを見ると、このマクロが呼ばれたソースコードファイル名、行番号とともに、変数名とその値が表示されているのがわかります。
さらに、`println!`は標準出力に出力するので、標準エラーに出力するためには`eprintln!`を使う必要がありますが、`dbg!`は正しく標準エラーに出力を行います。

<!--
It even works in more complex circumstances. Consider this factorial example:
-->
`dbg!`マクロは、より複雑な状況でも動作します。以下の階乗の例を考えてみましょう。


```rust
fn factorial(n: u32) -> u32 {
    if n <= 1 {
        n
    } else {
        n * factorial(n - 1)
    }
}
```

<!--
If we wanted to debug this, we might write it like this with `eprintln!`:
-->
これをデバッグしたい場合、`eprintln!`を使ってこのように書くかも知れません。

```rust
fn factorial(n: u32) -> u32 {
    eprintln!("n: {}", n);

    if n <= 1 {
        eprintln!("n <= 1");

        n
    } else {
        let n = n * factorial(n - 1);

        eprintln!("n: {}", n);

        n
    }
}
```

<!--
We want to log `n` on each iteration, as well as have some kind of context
for each of the branches. We see this output for `factorial(4)`:
 -->
ここでは、各反復で`n`の値と、場合分けのコンテキストを記録したいのですが、 `factorial(4)`を実行すると以下の出力が得られます。

```text
n: 4
n: 3
n: 2
n: 1
n <= 1
n: 2
n: 6
n: 24
```

<!--
This is servicable, but not particularly great. Maybe we could work on how we
print out the context to make it more clear, but now we're not debugging our
code, we're figuring out how to make our debugging code better.
-->
これは使えなくはないですが、特別に良いということでもありません。
コンテキストをより明確に表示するのに工夫することはできるでしょうが、ここではコードのデバッグをしているのではなく、デバッグ用のコードをより良くする方法を探そうとしています。

<!--
Consider this version using `dbg!`:
-->
`dbg!`を使ったこのバージョンを考えてみましょう。

```rust
fn factorial(n: u32) -> u32 {
    if dbg!(n <= 1) {
        dbg!(1)
    } else {
        dbg!(n * factorial(n - 1))
    }
}
```

<!--
We simply wrap each of the various expressions we want to print with the macro. We get this output instead:
 -->
ここでは、表示したい幾つかの式を単純にマクロで囲っています。
これを実行するとこのような出力が得られます。

```text
[src/main.rs:3] n <= 1 = false
[src/main.rs:3] n <= 1 = false
[src/main.rs:3] n <= 1 = false
[src/main.rs:3] n <= 1 = true
[src/main.rs:4] 1 = 1
[src/main.rs:5] n * factorial(n - 1) = 2
[src/main.rs:5] n * factorial(n - 1) = 6
[src/main.rs:5] n * factorial(n - 1) = 24
[src/main.rs:11] factorial(4) = 24
```

<!--
Because the `dbg!` macro returns the value of what it's debugging, instead of
`eprintln!` which returns `()`, we need to make no changes to the structure of
our code. Additionally, we have vastly more useful output.
-->
`eprintln!`は常に`()`を返しますが、`dbg!`マクロは与えられた式の値を返すので、コードの構成を変える必要がありません。
加えて、表示結果にとても役に立つ情報が含まれています。
