<!--
# Disallow references to static mut
-->

# static mut への参照の禁止

<!--
## Summary
-->

## 概要

<!--
- The [`static_mut_refs`] lint level is now `deny` by default.
  This checks for taking a shared or mutable reference to a `static mut`.
-->

- [`static_mut_refs`] のリントレベルはデフォルトで `deny` (必ずエラー) になりました。
  このリントは、`static mut` への共有参照・可変参照を検出します。

<!--
[`static_mut_refs`]: ../../rustc/lints/listing/warn-by-default.html#static-mut-refs
-->

[`static_mut_refs`]: https://doc.rust-lang.org/rustc/lints/listing/warn-by-default.html#static-mut-refs

<!--
## Details
-->

## 詳細

<!--
The [`static_mut_refs`] lint detects taking a reference to a [`static mut`]. In the 2024 Edition, this lint is now `deny` by default to emphasize that you should avoid making these references.
-->

[`static_mut_refs`] リントは、[`static mut`] への参照を検出します。
この種の参照は避けるべきであると明示するために、2024 エディションではこのリントはデフォルトで `deny` (不許可＝必ずエラー) となります。

<!-- edition2024 -->
```rust
static mut X: i32 = 23;
static mut Y: i32 = 24;

unsafe {
    let y = &X;             // ERROR: shared reference to mutable static
    let ref x = X;          // ERROR: shared reference to mutable static
    let (x, y) = (&X, &Y);  // ERROR: shared reference to mutable static
                            // (訳) エラー: 可変スタティック変数への共有参照
}
```

<!--
Merely taking such a reference in violation of Rust's mutability XOR aliasing requirement has always been *instantaneous* [undefined behavior], **even if the reference is never read from or written to**.  Furthermore, upholding mutability XOR aliasing for a `static mut` requires *reasoning about your code globally*, which can be particularly difficult in the face of reentrancy and/or multithreading.
-->

Rust の可変性に関するエイリアシングルール [^1] を破るような参照を生成するのは、**たとえその参照が実際に読み書きされないとしても**、今も昔も問答無用で[未定義動作]です。
その上、このルールは **コード全体で** 守られてなくてはならず、特に再入可能性やマルチスレッドが関わると困難なタスクです。

[^1]: 一つの値に対して、ただ一つの可変参照か、複数の共有参照の、たかだか一方しか存在することができないというルールのこと。

<!--
Note that there are some cases where implicit references are automatically created without a visible `&` operator. For example, these situations will also trigger the lint:
-->

注意すべきなのは、表向きは `&` が使われていなくても暗黙に参照が生成される場面がある点です。
たとえば、以下のような使い方もリントに検知されます。

<!-- edition2024 -->
```rust
static mut NUMS: &[u8; 3] = &[0, 1, 2];

unsafe {
    println!("{NUMS:?}");   // ERROR: shared reference to mutable static
    let n = NUMS.len();     // ERROR: shared reference to mutable static
                            // (訳) エラー: 可変スタティック変数への共有参照
}
```

<!--
## Alternatives
-->

## 代替案

<!--
Wherever possible, it is **strongly recommended** to use instead an *immutable* `static` of a type that provides *interior mutability* behind some *locally-reasoned abstraction* (which greatly reduces the complexity of ensuring that Rust's mutability XOR aliasing requirement is upheld).
-->

可能な限り、（可変性エイリアシングルールの検証がはるかに楽となるように）**局所的に正当化された抽象化**にもとづく**内部可変性**を提供する型の、`static` で**不変な**変数を使うことが**強く推奨**されます。

<!--
In situations where no locally-reasoned abstraction is possible and you are therefore compelled still to reason globally about accesses to your `static` variable, you must now use raw pointers such as can be obtained via the [`&raw const` or `&raw mut` operators][raw].  By first obtaining a raw pointer rather than directly taking a reference, (the safety requirements of) accesses through that pointer will be more familiar to `unsafe` developers and can be deferred until/limited to smaller regions of code.
-->

局所的正当化による抽象化が不可能で、どうしても `static` 変数へのグローバルアクセスが必要なときは、[`&raw const` か `&raw mut` 演算子][raw]を用いるなどして生ポインタを使う必要があります。
直接参照を取るのでなく先に生ポインタを作っておけば、そのポインタへのアクセス（に必要な安全性要件）は `unsafe` コードを書く人にとっても慣れたものですし、より局所的なコードに限定（先送り）できます。

<!--
[Undefined Behavior]: ../../reference/behavior-considered-undefined.html
[`static mut`]: ../../reference/items/static-items.html#mutable-statics
[`addr_of_mut!`]: https://docs.rust-lang.org/core/ptr/macro.addr_of_mut.html
[raw]: ../../reference/expressions/operator-expr.html#raw-borrow-operators
-->

[未定義動作]: https://doc.rust-lang.org/reference/behavior-considered-undefined.html
[`static mut`]: https://doc.rust-lang.org/reference/items/static-items.html#mutable-statics
[`addr_of_mut!`]: https://docs.rust-lang.org/core/ptr/macro.addr_of_mut.html
[raw]: https://doc.rust-lang.org/reference/expressions/operator-expr.html#raw-borrow-operators

<!--
Note that the following examples are just illustrations and are not intended as full-fledged implementations. Do not copy these as-is. There are details for your specific situation that may require alterations to fit your needs. These are intended to help you see different ways to approach your problem.
-->

なお、以下に示すのはあくまで例示で、厳密な実装というわけではありません。
そのまま丸写ししないでください。
一見使えそうな例があっても、用途に合わせて書き換えるべき点もあるでしょう。
ここでは、様々な問題に対処するための選択肢を示すためにいくつかの例を列挙しています。

<!--
It is recommended to read the documentation for the specific types in the standard library, the reference on [undefined behavior], the [Rustonomicon], and if you are having questions to reach out on one of the Rust forums such as the [Users Forum].
-->

[未定義動作]のリファレンスや [Rust 裏本]、標準ライブラリで提供される型を使用する場合はそのドキュメントなどをよくお読みください。
疑問点は[ユーザーズフォーラム]などの Rust フォーラムで質問するとよいでしょう。

<!--
[undefined behavior]: ../../reference/behavior-considered-undefined.html
[Rustonomicon]: ../../nomicon/index.html
[Users Forum]: https://users.rust-lang.org/
-->

[未定義動作]: https://doc.rust-lang.org/reference/behavior-considered-undefined.html
[Rustonomicon]: https://doc.rust-jp.rs/rust-nomicon-ja/index.html
[Users Forum]: https://users.rust-lang.org/

> **訳注**: 日本語話者向けの Rust コミュニティとして [Zulip](https://rust-lang-jp.zulipchat.com/) がありますので、併せてご活用ください。

<!--
### Don't use globals
-->

### グローバル変数を使わない

<!--
This is probably something you already know, but if possible it is best to avoid mutable global state. Of course this can be a little more awkward or difficult at times, particularly if you need to pass a mutable reference around between many functions.
-->

ご存知かもしれませんが、グローバルな状態を書き換えなくて済むならそれが一番です。
もちろんこれは場合によっては、特に可変参照をたくさんの関数に引き回す必要のあるときは、若干の困難や面倒の種です。

<!--
### Atomics
-->

### アトミック型

<!--
The [atomic types][atomics] provide integers, pointers, and booleans that can be used in a `static` (without `mut`).
-->

[アトミック型][atomics]を使うと、`static` 文脈で（`mut` なしに）使える整数・ポインタ・論理型を定義できます。

```rust,edition2024
# use std::sync::atomic::Ordering;
# use std::sync::atomic::AtomicU64;

// Change from this:
//   static mut COUNTER: u64 = 0;
// to this:
static COUNTER: AtomicU64 = AtomicU64::new(0);

fn main() {
    // Be sure to analyze your use case to determine the correct Ordering to use.
    COUNTER.fetch_add(1, Ordering::Relaxed);
}
```

```rust,edition2024
# use std::sync::atomic::Ordering;
# use std::sync::atomic::AtomicU64;

//   static mut COUNTER: u64 = 0;
// 上記の変数は以下のように置き換えられる
static COUNTER: AtomicU64 = AtomicU64::new(0);

fn main() {
    // コードに応じて適切な `Ordering` を指定する
    COUNTER.fetch_add(1, Ordering::Relaxed);
}
```

<!--
[atomics]: ../../std/sync/atomic/index.html
-->

[atomics]: https://doc.rust-lang.org/std/sync/atomic/index.html

<!--
### Mutex or RwLock
-->

### Mutex と RwLock

<!--
When your type is more complex than an atomic, consider using a [`Mutex`] or [`RwLock`] to ensure proper access to the global value.
-->

アトミック型より複雑な型を使いたい場合は、グローバル変数への適切なアクセスの確保のために [`Mutex`] や [`RwLock`] が使えます。

<!--
```rust,edition2024
# use std::sync::Mutex;
# use std::collections::VecDeque;

// Change from this:
//     static mut QUEUE: VecDeque<String> = VecDeque::new();
// to this:
static QUEUE: Mutex<VecDeque<String>> = Mutex::new(VecDeque::new());

fn main() {
    QUEUE.lock().unwrap().push_back(String::from("abc"));
    let first = QUEUE.lock().unwrap().pop_front();
}
```
-->

```rust,edition2024
# use std::sync::Mutex;
# use std::collections::VecDeque;

//     static mut QUEUE: VecDeque<String> = VecDeque::new();
// 上記の変数は以下のように置き換えられる
static QUEUE: Mutex<VecDeque<String>> = Mutex::new(VecDeque::new());

fn main() {
    QUEUE.lock().unwrap().push_back(String::from("abc"));
    let first = QUEUE.lock().unwrap().pop_front();
}
```

<!--
[`Mutex`]: ../../std/sync/struct.Mutex.html
[`RwLock`]: ../../std/sync/struct.RwLock.html
-->

[`Mutex`]: https://doc.rust-lang.org/std/sync/struct.Mutex.html
[`RwLock`]: https://doc.rust-lang.org/std/sync/struct.RwLock.html

<!--
### OnceLock or LazyLock
-->

### OnceLock と LazyLock

<!--
If you are using a `static mut` because you need to do some one-time initialization that can't be `const`, you can instead reach for [`OnceLock`] or [`LazyLock`] instead.
-->

`const` にできない一度限りの初期化をするために `static mut` を使っている場合、[`OnceLock`] や [`LazyLock`] に置き換えられます。

<!--
```rust,edition2024
# use std::sync::LazyLock;
#
# struct GlobalState;
#
# impl GlobalState {
#     fn new() -> GlobalState {
#         GlobalState
#     }
#     fn example(&self) {}
# }

// Instead of some temporary or uninitialized type like:
//     static mut STATE: Option<GlobalState> = None;
// use this instead:
static STATE: LazyLock<GlobalState> = LazyLock::new(|| {
    GlobalState::new()
});

fn main() {
    STATE.example();
}
```
-->

```rust,edition2024
# use std::sync::LazyLock;
#
# struct GlobalState;
#
# impl GlobalState {
#     fn new() -> GlobalState {
#         GlobalState
#     }
#     fn example(&self) {}
# }

//     static mut STATE: Option<GlobalState> = None;
// 上記のように一時的・未初期化な値を使用する代わりに、以下のように書ける
static STATE: LazyLock<GlobalState> = LazyLock::new(|| {
    GlobalState::new()
});

fn main() {
    STATE.example();
}
```

<!--
[`OnceLock`] is similar to [`LazyLock`], but can be used if you need to pass information into the constructor, which can work well with single initialization points (like `main`), or if the inputs are available wherever you access the global.
-->

[`OnceLock`] と [`LazyLock`] は似ていますが、[`OnceLock`] は初期化時に情報を渡す必要がある場合に使え、特に初期化箇所が単一のときや（`main` など）、アクセス時にいつでもその情報を持っているときなどに便利です。

```rust,edition2024
# use std::sync::OnceLock;
#
# struct GlobalState;
#
# impl GlobalState {
#     fn new(verbose: bool) -> GlobalState {
#         GlobalState
#     }
#     fn example(&self) {}
# }
#
# struct Args {
#     verbose: bool
# }
# fn parse_arguments() -> Args {
#     Args { verbose: true }
# }

static STATE: OnceLock<GlobalState> = OnceLock::new();

fn main() {
    let args = parse_arguments();
    let state = GlobalState::new(args.verbose);
    let _ = STATE.set(state);
    // ...
    STATE.get().unwrap().example();
}
```

<!--
[`OnceLock`]: ../../std/sync/struct.OnceLock.html
[`LazyLock`]: ../../std/sync/struct.LazyLock.html
-->

[`OnceLock`]: https://doc.rust-lang.org/std/sync/struct.OnceLock.html
[`LazyLock`]: https://doc.rust-lang.org/std/sync/struct.LazyLock.html

<!--
### `no_std` one-time initialization
-->

### `no_std` な一度限りの初期化

<!--
This example is similar to [`OnceLock`] in that it provides one-time initialization of a global, but it does not require `std` which is useful in a `no_std` context. Assuming your target supports atomics, then you can use an atomic to check for the initialization of the global. The pattern might look something like this:
-->

以下の例はグローバル変数を一度だけ初期化するという点で [`OnceLock`] と似ていますが、`std` が不要なので `no_std` 文脈で便利です。
ターゲット環境がアトミック型をサポートするなら、グローバル変数の初期化が完了しているかどうかをアトミック型で管理できます。
例えば、以下のように書けるでしょう。

<!--
```rust,edition2024
# use core::sync::atomic::AtomicUsize;
# use core::sync::atomic::Ordering;
#
# struct Args {
#     verbose: bool,
# }
# fn parse_arguments() -> Args {
#     Args { verbose: true }
# }
#
# struct GlobalState {
#     verbose: bool,
# }
#
# impl GlobalState {
#     const fn default() -> GlobalState {
#         GlobalState { verbose: false }
#     }
#     fn new(verbose: bool) -> GlobalState {
#         GlobalState { verbose }
#     }
#     fn example(&self) {}
# }

const UNINITIALIZED: usize = 0;
const INITIALIZING: usize = 1;
const INITIALIZED: usize = 2;

static STATE_INITIALIZED: AtomicUsize = AtomicUsize::new(UNINITIALIZED);
static mut STATE: GlobalState = GlobalState::default();

fn set_global_state(state: GlobalState) {
    if STATE_INITIALIZED
        .compare_exchange(
            UNINITIALIZED,
            INITIALIZING,
            Ordering::SeqCst,
            Ordering::SeqCst,
        )
        .is_ok()
    {
        // SAFETY: The reads and writes to STATE are guarded with the INITIALIZED guard.
        unsafe {
            STATE = state;
        }
        STATE_INITIALIZED.store(INITIALIZED, Ordering::SeqCst);
    } else {
        panic!("already initialized, or concurrent initialization");
    }
}

fn get_state() -> &'static GlobalState {
    if STATE_INITIALIZED.load(Ordering::Acquire) != INITIALIZED {
        panic!("not initialized");
    } else {
        // SAFETY: Mutable access is not possible after state has been initialized.
        unsafe { &*&raw const STATE }
    }
}

fn main() {
    let args = parse_arguments();
    let state = GlobalState::new(args.verbose);
    set_global_state(state);
    // ...
    let state = get_state();
    state.example();
}
```
-->

```rust,edition2024
# use core::sync::atomic::AtomicUsize;
# use core::sync::atomic::Ordering;
#
# struct Args {
#     verbose: bool,
# }
# fn parse_arguments() -> Args {
#     Args { verbose: true }
# }
#
# struct GlobalState {
#     verbose: bool,
# }
#
# impl GlobalState {
#     const fn default() -> GlobalState {
#         GlobalState { verbose: false }
#     }
#     fn new(verbose: bool) -> GlobalState {
#         GlobalState { verbose }
#     }
#     fn example(&self) {}
# }

const UNINITIALIZED: usize = 0;
const INITIALIZING: usize = 1;
const INITIALIZED: usize = 2;

static STATE_INITIALIZED: AtomicUsize = AtomicUsize::new(UNINITIALIZED);
static mut STATE: GlobalState = GlobalState::default();

fn set_global_state(state: GlobalState) {
    if STATE_INITIALIZED
        .compare_exchange(
            UNINITIALIZED,
            INITIALIZING,
            Ordering::SeqCst,
            Ordering::SeqCst,
        )
        .is_ok()
    {
        // SAFETY: STATE への読み書きは INITIALIZED で保護されている
        unsafe {
            STATE = state;
        }
        STATE_INITIALIZED.store(INITIALIZED, Ordering::SeqCst);
    } else {
        panic!("already initialized, or concurrent initialization");
    }
}

fn get_state() -> &'static GlobalState {
    if STATE_INITIALIZED.load(Ordering::Acquire) != INITIALIZED {
        panic!("not initialized");
    } else {
        // SAFETY: state が初期化されると、グローバルなアクセスは不可能になる
        unsafe { &*&raw const STATE }
    }
}

fn main() {
    let args = parse_arguments();
    let state = GlobalState::new(args.verbose);
    set_global_state(state);
    // ...
    let state = get_state();
    state.example();
}
```

<!--
This example assumes you can put some default value in the static before it is initialized (the const `default` constructor in this example). If that is not possible, consider using either [`MaybeUninit`], or dynamic trait dispatch (with a dummy type that implements a trait), or some other approach to have a default placeholder.
-->

この例では、スタティック変数は最初何らかのデフォルト値（上記の例では const な `default` コンストラクタ）で埋められています。
それが不可能なときは、他のデフォルト値の埋め方として、[`MaybeUninit`] を使ったり、動的トレイトディスパッチを使って、初期値はそのトレイトを実装するダミー型にしておいたりするとよいでしょう。

<!--
There are community-provided crates that can provide similar one-time initialization, such as the [`static-cell`] crate (which supports targets that do not have atomics by using [`portable-atomic`]).
-->

[`static-cell`] など、一度限りの初期化機能を提供するようなサードパーティークレートを使うこともできます（このクレートは [`portable-atomic`] を利用しており、アトミック型が使えないターゲット環境でも利用可能です）。

<!--
[`MaybeUninit`]: ../../core/mem/union.MaybeUninit.html
[`static-cell`]: https://crates.io/crates/static_cell
[`portable-atomic`]: https://crates.io/crates/portable-atomic
-->

[`MaybeUninit`]: https://doc.rust-lang.org/core/mem/union.MaybeUninit.html
[`static-cell`]: https://crates.io/crates/static_cell
[`portable-atomic`]: https://crates.io/crates/portable-atomic

<!--
### Raw pointers
-->

### 生ポインタ

<!--
In some cases you can continue to use `static mut`, but avoid creating references. For example, if you just need to pass [raw pointers] into a C library, don't create an intermediate reference. Instead you can use [raw borrow operators], like in the following example:
-->

`static mut` を使い続けつつ、参照の生成を避ける方法もあります。
例えば、C のライブラリに[生ポインタ]を渡す必要があるだけだったら、参照を経由する代わりに以下のように[生参照演算子]を使うとよいでしょう。

<!--
```rust,edition2024,no_run
# #[repr(C)]
# struct GlobalState {
#     value: i32
# }
#
# impl GlobalState {
#     const fn new() -> GlobalState {
#         GlobalState { value: 0 }
#     }
# }

static mut STATE: GlobalState = GlobalState::new();

unsafe extern "C" {
    fn example_ffi(state: *mut GlobalState);
}

fn main() {
    unsafe {
        // Change from this:
        //     example_ffi(&mut STATE as *mut GlobalState);
        // to this:
        example_ffi(&raw mut STATE);
    }
}
```
-->

```rust,edition2024,no_run
# #[repr(C)]
# struct GlobalState {
#     value: i32
# }
#
# impl GlobalState {
#     const fn new() -> GlobalState {
#         GlobalState { value: 0 }
#     }
# }

static mut STATE: GlobalState = GlobalState::new();

unsafe extern "C" {
    fn example_ffi(state: *mut GlobalState);
}

fn main() {
    unsafe {
        //     example_ffi(&mut STATE as *mut GlobalState);
        // 上記は以下のように置き換えられます。
        example_ffi(&raw mut STATE);
    }
}
```

<!--
Just beware that you still need to uphold the aliasing constraints around mutable pointers. This may require some internal or external synchronization or proofs about how it is used across threads, interrupt handlers, and reentrancy.
-->

ただし、可変ポインタのエイリアシング規則は変わらず遵守する必要があります。
内外における同期や、マルチスレッド・割り込み・再入可能性などの観点からの正常な利用を保証してください。

<!--
[raw borrow operators]: ../../reference/expressions/operator-expr.html#raw-borrow-operators
[raw pointers]: ../../reference/types/pointer.html#raw-pointers-const-and-mut
-->

[生参照演算子]: https://doc.rust-lang.org/reference/expressions/operator-expr.html#raw-borrow-operators
[生ポインタ]: https://doc.rust-lang.org/reference/types/pointer.html#raw-pointers-const-and-mut

<!--
### `UnsafeCell` with `Sync`
-->

### `Sync` な `UnsafeCell`

<!--
[`UnsafeCell`] does not impl `Sync`, so it cannot be used in a `static`. You can create your own wrapper around [`UnsafeCell`] to add a `Sync` impl so that it can be used in a `static` to implement interior mutability. This approach can be useful if you have external locks or other guarantees that uphold the safety invariants required for mutable pointers.
-->

[`UnsafeCell`] は `Sync` を impl しないので、`static` 文脈で使えません。
`static` 文脈で内部可変性が利用できるように、[`UnsafeCell`] の独自の `Sync` なラッパーを作ることができます。
特に、可変ポインタが要求する安全性不変条件を守るための機構（外部ロックなど）がある場合には、この方法が有効です。

<!--
Note that this is largely the same as the [raw pointers](#raw-pointers) example. The wrapper helps to emphasize how you are using the type, and focus on which safety requirements you should be careful of. But otherwise they are roughly the same.
-->

なおこの例は、[生ポインタ](#生ポインタ)の例とほぼ同等です。
このようなラッパーを使うことで、型の使用方法を強調して注意すべき安全性要件に着目させることができますが、他の点は大方同じです。

<!--
```rust,edition2024
# use std::cell::UnsafeCell;
#
# fn with_interrupts_disabled<T: Fn()>(f: T) {
#     // A real example would disable interrupts.
#     f();
# }
#
# #[repr(C)]
# struct GlobalState {
#     value: i32,
# }
#
# impl GlobalState {
#     const fn new() -> GlobalState {
#         GlobalState { value: 0 }
#     }
# }

#[repr(transparent)]
pub struct SyncUnsafeCell<T>(UnsafeCell<T>);

unsafe impl<T: Sync> Sync for SyncUnsafeCell<T> {}

static STATE: SyncUnsafeCell<GlobalState> = SyncUnsafeCell(UnsafeCell::new(GlobalState::new()));

fn set_value(value: i32) {
    with_interrupts_disabled(|| {
        let state = STATE.0.get();
        unsafe {
            // SAFETY: This value is only ever read in our interrupt handler,
            // and interrupts are disabled, and we only use this in one thread.
            (*state).value = value;
        }
    });
}
```
-->

```rust,edition2024
# use std::cell::UnsafeCell;
#
# fn with_interrupts_disabled<T: Fn()>(f: T) {
#     // 本当ならここで割り込みを無効化する
#     f();
# }
#
# #[repr(C)]
# struct GlobalState {
#     value: i32,
# }
#
# impl GlobalState {
#     const fn new() -> GlobalState {
#         GlobalState { value: 0 }
#     }
# }

#[repr(transparent)]
pub struct SyncUnsafeCell<T>(UnsafeCell<T>);

unsafe impl<T: Sync> Sync for SyncUnsafeCell<T> {}

static STATE: SyncUnsafeCell<GlobalState> = SyncUnsafeCell(UnsafeCell::new(GlobalState::new()));

fn set_value(value: i32) {
    // (訳注) 割り込みを無効化した状態で処理を実行する関数
    with_interrupts_disabled(|| {
        let state = STATE.0.get();
        unsafe {
            // SAFETY: この値は割り込みハンドラでしか読まれず、
            // 今は割り込みが無効化されており、かつこれは一つのスレッドからしか実行されない
            (*state).value = value;
        }
    });
}
```

<!--
The standard library has a nightly-only (unstable) variant of [`UnsafeCell`] called [`SyncUnsafeCell`]. This example above shows a very simplified version of the standard library type, but would be used roughly the same way. It can provide even better isolation, so do check out its implementation for more details.
-->

標準ライブラリには [`UnsafeCell`] の変種として [`SyncUnsafeCell`] がありますが、nightly 限定です（安定化されていません）。
上記のサンプルコードは [`SyncUnsafeCell`] の超簡略版ですが、使用方法はほぼ同じです。
[`SyncUnsafeCell`] ではより厳密な保護策がとられているので、詳細に関しては実装をご覧ください。

<!--
This example includes a fictional `with_interrupts_disabled` function which is the type of thing you might see in an embedded environment. For example, the [`critical-section`] crate provides a similar kind of functionality that could be used for an embedded environment.
-->

本例では仮想的な `with_interrupts_disabled` 関数を使っています。
組み込み環境ではこの手のものが実際に見られるでしょう。
例えば、[`critical-section`] クレートを使うと、組み込み環境で似たような機能が使えます。

<!--
[`critical-section`]: https://crates.io/crates/critical-section
[`UnsafeCell`]: ../../std/cell/struct.UnsafeCell.html
[`SyncUnsafeCell`]: ../../std/cell/struct.SyncUnsafeCell.html
-->

[`critical-section`]: https://crates.io/crates/critical-section
[`UnsafeCell`]: https://doc.rust-lang.org/std/cell/struct.UnsafeCell.html
[`SyncUnsafeCell`]: https://doc.rust-lang.org/std/cell/struct.SyncUnsafeCell.html

### Safe references

<!--
In some cases it may be safe to create a reference of a `static mut`. The whole point of the [`static_mut_refs`] lint is that this is very hard to do correctly! However, that's not to say it is *impossible*. If you have a situation where you can guarantee that the aliasing requirements are upheld, such as guaranteeing the static is narrowly scoped (only used in a small module or function), has some internal or external synchronization, accounts for interrupt handlers and reentrancy, panic safety, drop handlers, etc., then taking a reference may be fine.
-->

`static mut` の参照を作っても安全な場面も存在するにはします。
[`static_mut_refs`] リントが言いたいのは正しい使用が非常に困難だということで、不可能ではないのです。
エイリアシング規則が遵守できていると保証できるなら、参照を作っても問題ないでしょう。
例えば、スタティック変数のスコープが十分小さかったり（小さいモジュールや関数など）、内部や外部に同期機構があったり、割り込みハンドラや再入可能性、パニック時の安全性、ドロップハンドラなども十分に考慮できる場合です。

<!--
There are two approaches you can take for this. You can either allow the [`static_mut_refs`] lint (preferably as narrowly as you can), or convert raw pointers to a reference, as with `&mut *&raw mut MY_STATIC`.
-->

この場合、2 つの方法があります。
[`static_mut_refs`] リントを（できるだけ狭い範囲で）有効化するか、生ポインタを `&mut *&raw mut MY_STATIC` のように参照に変換するかです。

<!-- TODO: Should we prefer one or the other here? -->

<!--
#### Short-lived references
-->

#### 参照を短命にする

<!--
If you must create a reference to a `static mut`, then it is recommended to minimize the scope of how long that reference exists. Avoid squirreling the reference away somewhere, or keeping it alive through a large section of code. Keeping it short-lived helps with auditing, and verifying that exclusive access is maintained for the duration. Using pointers should be your default unit, and only convert the pointer to a reference on demand when absolutely required.
-->

`static mut` への参照を作る必要があるときは、参照が存在するスコープをできるだけ小さくすることが肝要です。
作った参照を後で使うために大事に取っておいたり、コード中で長い間捨てずに保持し続けたりしないでください。
参照を短命にしておくことで、その間だけ排他的アクセスが保持されていることが検証しやすくなります。
基本的にポインタを使っておき、どうしても必要なときだけ参照に変換するとよいでしょう。

<!--
## Migration
-->

## 移行

<!--
There is no automatic migration to fix these references to `static mut`. To avoid undefined behavior you must rewrite your code to use a different approach as recommended in the [Alternatives](#alternatives) section.
-->

`static mut` への参照に関する自動移行はありません。
未定義動作を避けるために、[代替案](#代替案)の節で示したような方法での修正が必要です。
