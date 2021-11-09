<!--
# What are Editions?
-->
# エディションとは？

<!--
The release of Rust 1.0 established
["stability without stagnation"](https://blog.rust-lang.org/2014/10/30/Stability.html)
as a core Rust deliverable.
Ever since the 1.0 release,
the rule for Rust has been that once a feature has been released on stable,
we are committed to supporting that feature for all future releases.
-->

Rust 1.0 のリリースでは、Rust のコア機能として[「よどみない安定性」](https://blog.rust-lang.org/2014/10/30/Stability.html)が提供されるようになりました。
Rust は、1.0 のリリース以来、いちど安定版にリリースされた機能は、将来の全リリースに渡ってサポートし続ける、というルールの下で開発されてきました。

<!--
There are times, however, when it is useful to be able to make small changes
to the language that are not backwards compatible.
The most obvious example is introducing a new keyword,
which would invalidate variables with the same name.
For example, the first version of Rust did not have the `async` and `await` keywords.
Suddenly changing those words to keywords in a later version would've broken code like `let async = 1;`.
-->

一方で、後方互換でないような小さい変更を言語に加えることも、ときには便利です。
最もわかりやすいのは新しいキーワードの導入で、これは同名の変数を使えなくします。
例えば、Rust の最初のバージョンには `async` や `await` といったキーワードはありませんでした。
後のバージョンになってこれらを突然キーワードに変えてしまうと、例えば `let async = 1;` のようなコードが壊れてしまいます。

<!--
**Editions** are the mechanism we use to solve this problem.
When we want to release a feature that would otherwise be backwards incompatible,
we do so as part of a new Rust *edition*.
Editions are opt-in, and so existing crates do
not see these changes until they explicitly migrate over to the new edition.
This means that even the latest version of Rust will still *not* treat `async` as a keyword,
unless edition 2018 or later is chosen.
This choice is made *per crate* [as part of its `Cargo.toml`](https://doc.rust-lang.org/cargo/reference/manifest.html#the-edition-field).
New crates created by `cargo new` are always configured to use the latest stable edition.
-->

このような問題を解決するために、**エディション**という仕組みが使われています。
後方互換性を失わせるような機能をリリースしたいとき、我々はこれを新しい*エディション*の一部として提供します。
エディションはオプトイン、すなわち導入したい人だけが導入できるので、既存のクレートは明示的に新しいエディションに移行しない限りは変化を受けません。
すなわち、2018 以降のエディションを選択しない限り、たとえ最新バージョンの Rust であっても `async` はキーワードとして*扱われません*。
導入の可否は*クレートごとに*決めることができ、[`Cargo.toml` への記載内容](https://doc.rust-lang.org/cargo/reference/manifest.html#the-edition-field)により決定されます。
`cargo new` コマンドで作成される新しいクレートは、常に最新の安定版のエディションでセットアップされます。

<!--
### Editions do not split the ecosystem
-->

### エディションはエコシステムを分断しない

<!--
The most important rule for editions is that crates in one edition can
interoperate seamlessly with crates compiled in other editions. This ensures
that the decision to migrate to a newer edition is a "private one" that the
crate can make without affecting others.
-->

エディションで最も重要な規則は、あるエディションのクレートと別のエディションでコンパイルされたクレートがシームレスに相互運用できるようになっているということです。
これにより、新しいエディションへ移行するかどうかは、他のクレートに影響を与えない「自分だけの問題」だと言えるのです。

<!--
The requirement for crate interoperability implies some limits on the kinds of
changes that we can make in an edition.
In general, changes that occur in an edition tend to be "skin deep".
All Rust code, regardless of edition,
is ultimately compiled to the same internal representation within the compiler.
-->

クレートの相互運用性を守るために、我々がエディションに加えられる変更にはある種の制限がかかります。
一般に、エディションに加えられる変更は「表面上の」ものになりがちです。
エディションに関わらず、すべての Rust のコードは最終的にはコンパイラの中で同じ内部表現に変換されるのです。

<!--
### Edition migration is easy and largely automated
-->

### エディションの移行は簡単で、ほとんど自動化されている

<!--
Our goal is to make it easy for crates to upgrade to a new edition.
When we release a new edition,
we also provide [tooling to automate the migration](https://doc.rust-lang.org/cargo/commands/cargo-fix.html).
It makes minor changes to your code necessary to make it compatible with the new edition.
For example, when migrating to Rust 2018, it changes anything named `async` to use the equivalent
[raw identifier syntax](https://doc.rust-lang.org/rust-by-example/compatibility/raw_identifiers.html): `r#async`.
-->

我々は、クレートを新しいエディションにアップグレードするのが簡単になるよう目指しています。
新しいエディションがリリースされるとき、我々は[移行を自動化するツール](https://doc.rust-lang.org/cargo/commands/cargo-fix.html)も提供します。
このツールは、新しいエディションに適合させるために必要な小さな変更をコードに施します。
例えば、Rust 2018 への移行の際は、`async` と名のつく全てのものを、等価な[生識別子構文](https://doc.rust-lang.org/rust-by-example/compatibility/raw_identifiers.html)である `r#async` へと書き換える、といった具合です。

<!--
The automated migrations are not necessarily perfect:
there might be some corner cases where manual changes are still required.
The tooling tries hard to avoid changes
to semantics that could affect the correctness or performance of the code.
-->

この自動移行は必ずしも完璧とは限らず、手作業での変更が必要なコーナーケースもないとは言えません。
このツールは、コードの正しさやパフォーマンスに影響を与えうるような、プログラムの意味に関わる変更は避けるために全力を尽くします。

<!--
In addition to tooling, we also maintain this Edition Migration Guide that covers
the changes that are part of an edition.
This guide describes each change and gives pointers to where you can learn more about it.
It also covers any corner cases or details you should be aware of.
This guide serves both as an overview of the edition
and as a quick troubleshooting reference
if you encounter problems with the automated tooling.
-->

我々はこのツールの他に、エディションを構成する変更を取り扱っている、本エディション移行ガイドも管理しています。
このガイドでは、それぞれの変更内容と、もっと詳しく知りたい人向けのリンク、さらには知っておくべき詳細や重箱の隅まで網羅しています。
このガイドはエディションの概要を示すと同時に、自動化ツールに何らかの問題が生じたときのトラブルシューティング用の文献にもなります。
