<!--
# What are Editions?
-->
# エディションとは？

<!--
In May 2015, the [release of Rust 1.0](https://blog.rust-lang.org/2015/05/15/Rust-1.0.html) established "[stability without stagnation](https://blog.rust-lang.org/2014/10/30/Stability.html)" as a core Rust axiom. Since then, Rust has committed to a pivotal rule: once a feature is [released through stable](https://doc.rust-lang.org/book/appendix-07-nightly-rust.html), contributors will continue to support that feature for all future releases.
-->

2015年5月に Rust が[バージョン 1.0](https://blog.rust-lang.org/2015/05/15/Rust-1.0.html)を迎えるにあたり、
[「よどみない安定性」](https://blog.rust-lang.org/2014/10/30/Stability.html)という根本理念が掲げられました。
これにより、安定版にリリースされた機能は、将来の全リリースに渡って開発者がサポートし続けるというルールを厳守しながら、
Rust は 1.0 から今まで開発され続けてきました。

<!--
However, there are times when it's useful to make backwards-incompatible changes to the language. A common example is the introduction of a new keyword. For instance, early versions of Rust didn't feature the `async` and `await` keywords.
-->

一方で、後方互換性を破るような言語仕様変更をすべきときもあります。
例えば、Rust の初期のバージョンには `asnyc` や `await` といったキーワードはありませんでした。

<!--
If Rust had suddenly introduced these new keywords, some code would have broken: `let async = 1;` would no longer work.
-->

後のバージョンになってそれらが突然キーワードになってしまうと、例えば `let async = 1;` のようなコードが壊れてしまいます。

<!--
Rust uses **editions** to solve this problem. When there are backwards-incompatible changes, they are pushed into the next edition. Since editions are opt-in, existing crates won't use the changes unless they explicitly migrate into the new edition. For example, the latest version of Rust doesn't treat `async` as a keyword unless edition 2018 or later is chosen.
-->

このような問題を解決するために、**エディション**という仕組みが使われています。
後方互換性を破るような変更は、次回のエディションで導入されます。
エディションはオプトイン、すなわち導入したい人だけが導入できるので、既存のクレートは明示的に新しいエディションに移行しない限りは影響を受けません。
例えば、2018 以降のエディションを選択しない限り、たとえ最新バージョンの Rust であっても `async` はキーワードとして扱われません。

<!--
Each crate chooses its edition [within its `Cargo.toml` file](https://doc.rust-lang.org/cargo/reference/manifest.html#the-edition-field). When creating a new crate with Cargo, it will automatically select the newest stable edition.
-->

エディションは [`Cargo.toml` で指定](https://doc.rust-lang.org/cargo/reference/manifest.html#the-edition-field)することができ、クレートごとに独立です。
Cargo で新しいクレートを作る場合は、自動的に最新の安定版のエディションが選ばれます。

<!--
## Editions do not split the ecosystem
-->

## エディションはエコシステムを分断しない

<!--
When creating editions, there is one most consequential rule: crates in one edition **must** seamlessly interoperate with those compiled with other editions.
-->

エディションを導入するにあたって最重要なのは、
あるエディションのクレートと別のエディションでコンパイルされたクレートがシームレスに相互運用できることです。

<!--
In other words, each crate can decide when to migrate to a new edition independently. This decision is 'private' - it won't affect other crates in the ecosystem.
-->

すなわち、新しいエディションへ移行するかどうかは、クレートごとに決められます。
移行の決断が、他のクレートに影響を与えない「自分だけの問題」だと言えるのです。

<!--
For Rust, this required compatibility implies some limits on the kinds of changes that can be featured in an edition. As a result, changes found in new Rust editions tend to be 'skin deep'. All Rust code - regardless of edition - will ultimately compile down to the same internal representation within the compiler.
-->

Rust では、クレートの相互運用性を守るために、エディションへの機能追加はある程度制限されます。
一般に、エディションに加えられる変更は「表面上の」ものになりがちです。
エディションに関わらず、すべての Rust のコードは最終的にはコンパイラの中で同じ内部表現に変換されるのです。

<!--
## Edition migration is easy and largely automated
-->

## エディションの移行は簡単で、ほとんど自動化されている

<!--
Rust aims to make upgrading to a new edition an easy process. When a new edition releases, crate authors may use [automatic migration tooling within `cargo`](https://doc.rust-lang.org/cargo/commands/cargo-fix.html) to migrate. Cargo will then make minor changes to the code to make it compatible with the new version.
-->

Rust は、クレートを簡単に新しいエディションにアップグレードできるように目指しています。
新しいエディションがリリースされたら、[自動移行ツール](https://doc.rust-lang.org/cargo/commands/cargo-fix.html)が使用できます。
これを使うと、Cargo はコードが新バージョンに適合するように微修正します。

<!--
For example, when migrating to Rust 2018, anything named `async` will now use the equivalent [raw identifier syntax](https://doc.rust-lang.org/rust-by-example/compatibility/raw_identifiers.html): `r#async`.
-->
例えば、Rust 2018 への移行の際は、`async` と名のつく全てのものを、等価な[生識別子構文](https://doc.rust-lang.org/rust-by-example/compatibility/raw_identifiers.html)である `r#async` へと書き換える、といった具合です。

<!--
Cargo's automatic migrations aren't perfect: there may still be corner cases where manual changes are required. It aims to avoid changes to semantics that could affect the correctness or performance of the code.
-->
Cargo の自動移行は完璧でないので、手作業での変更が必要なコーナーケースもまれにあります。
正常動作やパフォーマンスといった、コードの意味を損ないうる変更はしないよう設計されています。

<!--
## What this guide covers
-->
## このガイドの対象読者

<!--
In addition to tooling, this Rust Edition Guide also covers the changes that are part of each edition. It describes each change and links to additional details, if available. It also covers corner cases or tricky details crate authors should be aware of.
-->
自動移行ツールに加えて、本資料「Rust エディションガイド」はで各エディションにおける変更内容を説明しています。
各変更内容に加え、もっと詳しく知りたい人向けのリンクもあれば紹介していますし、
さらには把握しておくべき詳細や重箱の隅まで網羅しています。

<!--
Crate authors should find:

- An overview of editions
- A migration guide for specific editions
- A quick troubleshooting reference when automated tooling isn't working.
-->
クレート作者の皆様は、

- エディションの概要
- 特定のエディションへの移行ガイド
- 自動移行ツールに何らかの問題が生じたときのトラブルシューティング用の文献

としてご利用ください。
