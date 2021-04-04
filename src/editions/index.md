<!--
# What are Editions?
-->
# エディションとは？

<!--
Rust ships releases on a six-week cycle. This means that users get a constant
stream of new features. This is much faster than updates for other languages,
but this also means that each update is smaller.  After a while, all of those
tiny changes add up. But, from release to release, it can be hard to look back
and say *"Wow, between Rust 1.10 and Rust 1.20, Rust has changed a lot!"*
-->

Rustは6週間ごとにリリースを行います。
これにより、ユーザーは新しい機能を常に手に入れることができます。
これは他の言語よりも速いサイクルですが、アップデートのサイズは小さくなります。
しばらくするとこれらの小変更が積み重なってきますが、いくつかのリリースを振り返って、「おお、バージョン1.10から1.20の間にRustは大きく変わったなぁ」と言うのは難しいかも知れません。

<!--
Every two or three years, we'll be producing a new *edition* of Rust. Each
edition brings together the features that have landed into a clear package, with
fully updated documentation and tooling. New editions ship through the usual
release process.
-->

2,3年に一度、Rustの新しい「エディション」を作成します。
各エディションはそれまでRustに加えられた変更をまとめ上げたもので、最新のドキュメントとツールもそれに含まれます。
新しいエディションは通常のリリースプロセスを経てリリースされます。

<!--
This serves different purposes for different people:

- For active Rust users, it brings together incremental changes into an
  easy-to-understand package.

- For non-users, it signals that some major advancements have landed, which
  might make Rust worth another look.

- For those developing Rust itself, it provides a rallying point for the project as a
  whole.
  -->

エディションは様々な人の異なる要求を満たします。

- Rustのアクティブなユーザーにとっては、6週間ごとににリリースされた機能変更をわかりやすくまとめたパッケージとなります。

- Rustを使っていない人にとっては、新たな機能が追加されたことを知らせる役割を果たし、それによってRustを使ってみようと思うようになるかも知れません。

- Rustの内部開発者にとっては、プロジェクト全体の集結地点になります。


<!--
## Compatibility
-->
## 互換性

<!--
When a new edition becomes available in the compiler, crates must explicitly opt
in to it to take full advantage. This opt in enables editions to contain
incompatible changes, like adding a new keyword that might conflict with
identifiers in code, or turning warnings into errors. A Rust compiler will
support all editions that existed prior to the compiler's release, and can link
crates of any supported editions together.
Edition changes only affect the way the compiler initially parses the code.
Therefore, if you're using Rust 2015, and
one of your dependencies uses Rust 2018, it all works just fine. The opposite
situation works as well.
-->

新しいエディションがコンパイラで利用可能になった際に、その利点を最大限に活かすためには、クレートは明示的にオプトインする必要があります。
このオプトインはエディションに非互換の変更を加えるために必要で、例えば、既存のコードで使われている識別子と競合する新たなキーワードを導入したり、警告だったものをエラーにする、などの変更を加えることができるようになります。
Rustのコンパイラはこれまでの全てのエディションをサポートしていて、どのエディションでもクレートをコンパイルすることができます。
エディションの変更はコンパイラが最初にコードを構文解析する際の動作のみに影響します。
従って、例ばあなたがRust 2015を使っていて、依存するクレートが Rust 2018を使っていても全く問題なく動作します。
その逆の場合も同様です。


<!--
Just to be clear: most features will be available on all editions.
People using any edition of Rust will continue to see improvements as new
stable releases are made.  In some cases however, mainly when new keywords are
added, but sometimes for other reasons, there may be new features that are only
available in later editions.  You only need to upgrade if you want to take
advantage of such features.
-->

念の為はっきりさせておきますが、ほとんどの機能は全てのエディションで利用可能です。
どのエディションを利用していても、新たな安定板リリースが出た際には改善を見ることができます。
時折、例えば新たなキーワードが導入されたりその他の理由で、あるエディション以降でしか利用できない機能追加があります。
そのような機能を利用したい時にエディションのアップデートを検討するのが良いでしょう。
