<!--
# The Edition Guide
-->
# エディションガイド

<!--
[Introduction](introduction.md)
-->
[序文](introduction.md)

<!--
## What are editions?
-->
## エディションとは？

<!--
- [What are editions?](editions/index.md)
  - [Creating a new project](editions/creating-a-new-project.md)
  - [Transitioning an existing project to a new edition](editions/transitioning-an-existing-project-to-a-new-edition.md)
  - [Advanced migrations](editions/advanced-migrations.md)
-->

- [エディションとは？](editions/index.md)
  - [新しいプロジェクトを作成する](editions/creating-a-new-project.md)
  - [既存のプロジェクトのエディションを移行する](editions/transitioning-an-existing-project-to-a-new-edition.md)
  - [Advanced migrations](editions/advanced-migrations.md)

## Rust 2015

- [Rust 2015](rust-2015/index.md)

## Rust 2018

- [Rust 2018](rust-2018/index.md)
  - [Path and module system changes](rust-2018/path-changes.md)
  - [Anonymous trait function parameters deprecated](rust-2018/trait-fn-parameters.md)
  - [New keywords](rust-2018/new-keywords.md)
  - [Method dispatch for raw pointers to inference variables](rust-2018/tyvar-behind-raw-pointer.md)
  - [Cargo changes](rust-2018/cargo.md)

## Post Rust 2018

- [Rust 2018以降の変更（期間限定公開）](rust-post-2018/index.md)
    - [dbg! マクロ](rust-post-2018/dbg-macro.md)
    - [デフォルトでjemallocを使わない](rust-post-2018/no-jemalloc.md)
    - [統一的なパス](rust-post-2018/uniform-paths.md)
    - [リテラルマクロマッチャ](rust-post-2018/literal-macro-matcher.md)
    - [マクロ内の`?`演算子](rust-post-2018/qustion-mark-operator-in-macros.md)
    - [const fn](rust-post-2018/const-fn.md)
    - [ピン留め](rust-post-2018/pin.md)
    - [FnBoxは不要に](rust-post-2018/no-more-fnbox.md)
    - [Cargoレジストリが選択できるように](rust-post-2018/alternative-cargo-registries.md)
    - [TryFromとTryInto](rust-post-2018/tryfrom-and-tryinto.md)
    - [Futureトレイト](rust-post-2018/future.md)
    - [allocクレート](rust-post-2018/alloc.md)
    - [MaybeUninit<T>](rust-post-2018/maybe-uninit.md)
    - [cargo vendor](rust-post-2018/cargo-vendor.md)

## Rust 2021

<!--
- [Rust 2021](rust-2021/index.md)
  - [Additions to the prelude](rust-2021/prelude.md)
  - [Default Cargo feature resolver](rust-2021/default-cargo-resolver.md)
  - [IntoIterator for arrays](rust-2021/IntoIterator-for-arrays.md)
  - [Disjoint capture in closures](rust-2021/disjoint-capture-in-closures.md)
  - [Panic macro consistency](rust-2021/panic-macro-consistency.md)
  - [Reserving syntax](rust-2021/reserving-syntax.md)
  - [Warnings promoted to errors](rust-2021/warnings-promoted-to-error.md)
  - [Or patterns in macro-rules](rust-2021/or-patterns-macro-rules.md)
-->

- [Rust 2021](rust-2021/index.md)
  - [Additions to the prelude](rust-2021/prelude.md)
  - [デフォルトの Cargo のフィーチャリゾルバ](rust-2021/default-cargo-resolver.md)
  - [配列に対する IntoIterator](rust-2021/IntoIterator-for-arrays.md)
  - [クロージャはフィールドごとにキャプチャする](rust-2021/disjoint-capture-in-closures.md)
  - [Panic macro consistency](rust-2021/panic-macro-consistency.md)
  - [Reserving syntax](rust-2021/reserving-syntax.md)
  - [警告からエラーへの格上げ](rust-2021/warnings-promoted-to-error.md)
  - [Or patterns in macro-rules](rust-2021/or-patterns-macro-rules.md)
