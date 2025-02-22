<!--
# Default Cargo feature resolver
-->

# デフォルトの Cargo のフィーチャリゾルバ

<!--
## Summary
-->

## 概要

<!--
- `edition = "2021"` implies `resolver = "2"` in `Cargo.toml`.
-->

- `edition = "2021"` では `Cargo.toml` で `resolver = "2"` が設定されているとみなされます。

<!--
## Details
-->

## 詳細

<!--
Since Rust 1.51.0, Cargo has opt-in support for a [new feature resolver][4]
which can be activated with `resolver = "2"` in `Cargo.toml`.
-->

Rust 1.51.0 から、Cargo には[新しいフィーチャリゾルバ][4]がオプトインできるようになっています。
これは、`Cargo.toml` で `resolver = "2"` と書くことで有効化できます。

<!--
Starting in Rust 2021, this will be the default.
That is, writing `edition = "2021"` in `Cargo.toml` will imply `resolver = "2"`.
-->

Rust 2021 から、これがデフォルトになりました。
つまり、 `Cargo.toml` に `edition = "2021"` と書けば、暗黙に `resolver = "2"` も設定されているとみなされます。

<!--
The resolver is a global setting for a [workspace], and the setting is ignored in dependencies.
The setting is only honored for the top-level package of the workspace.
If you are using a [virtual workspace], you will still need to explicitly set the [`resolver` field]
in the `[workspace]` definition if you want to opt-in to the new resolver.
-->

このリゾルバは[ワークスペース]全体に設定され、依存先では無視されます。
また、この設定はワークスペースの最上位のパッケージでしか効きません。
[仮想ワークスペース]において新しいリゾルバをオプトインしたい場合は、
以前と同様に [`resolver` フィールド]を明示的に設定する必要があります。　

<!--
The new feature resolver no longer merges all requested features for
crates that are depended on in multiple ways.
See [the announcement of Rust 1.51][5] for details.
-->

新しいフィーチャリゾルバは、クレートへの依存に異なるフィーチャが設定されていてもそれらをマージしないようになりました。
詳細は [the announcement of Rust 1.51][5] に記載されています。

<!--
[4]: ../../cargo/reference/resolver.html#feature-resolver-version-2
[5]: https://blog.rust-lang.org/2021/03/25/Rust-1.51.0.html#cargos-new-feature-resolver
[workspace]: ../../cargo/reference/workspaces.html
[virtual workspace]: ../../cargo/reference/workspaces.html#virtual-workspace
[`resolver` field]: ../../cargo/reference/resolver.html#resolver-versions
-->

[4]: https://doc.rust-lang.org/cargo/reference/resolver.html#feature-resolver-version-2
[5]: https://blog.rust-lang.org/2021/03/25/Rust-1.51.0.html#cargos-new-feature-resolver
[ワークスペース]: https://doc.rust-lang.org/cargo/reference/workspaces.html
[仮想ワークスペース]: https://doc.rust-lang.org/cargo/reference/workspaces.html#virtual-workspace
[`resolver` フィールド]: https://doc.rust-lang.org/cargo/reference/resolver.html#resolver-versions

<!--
## Migration
-->

## 移行

<!--
There are no automated migration tools for updating for the new resolver.
For most projects, there are usually few or no changes as a result of updating.
-->

新しいリゾルバに適合させるための自動化された移行ツールはありません。
ほとんどのプロジェクトでは、更新後に必要な変更はあっても微々たるものでしょう。

<!--
When updating with `cargo fix --edition`, Cargo will display a report if the new resolver will build dependencies with different features.
It may look something like this:
-->

`cargo fix --edition` でのアップデート時に、Cargo は新しいリゾルバで依存先のフィーチャに変更があるかどうかを表示します。
たとえば、このように表示されます:

> note: Switching to Edition 2021 will enable the use of the version 2 feature resolver in Cargo.
> This may cause some dependencies to be built with fewer features enabled than previously.
> More information about the resolver changes may be found at <https://doc.rust-lang.org/nightly/edition-guide/rust-2021/default-cargo-resolver.html><br>
> When building the following dependencies, the given features will no longer be used:
>
> _(訳)_ 2021 エディションに切り替えると、Cargoのフィーチャリゾルバがバージョン 2 に切り替わります。
> 切り替え後、いくつかの依存先では有効化されるフィーチャが減少することがあります。
> リゾルバの変更点については、 <https://doc.rust-lang.org/nightly/edition-guide/rust-2021/default-cargo-resolver.html> もご覧ください。<br>
> 以下の依存先をビルドするときに、以下のフィーチャが使われなくなります:
>
> ```text
>   bstr v0.2.16: default, lazy_static, regex-automata, unicode
>   libz-sys v1.1.3 (as host dependency): libc
> ```

<!--
This lets you know that certain dependencies will no longer be built with the given features.
-->

これにより、記載されたフィーチャがその依存先で使われずにビルドされるようになることがわかります。

<!--
### Build failures
-->

### ビルドの失敗

<!--
There may be some circumstances where your project may not build correctly after the change.
If a dependency declaration in one package assumes that certain features are enabled in another, and those features are now disabled, it may fail to compile.
-->

状況によっては、変更後にプロジェクトが正しくビルドされなくなることもあります。
あるパッケージの依存関係が、別のパッケージにおいて特定のフィーチャが有効されることを前提にしている場合、そのフィーチャが使われなくなることでコンパイルに失敗するかもしれません。

<!--
For example, let's say we have a dependency like this:
-->

たとえば、我々のパッケージにこんな依存関係があったとしましょう：

```toml
# Cargo.toml

[dependencies]
bstr = { version = "0.2.16", default-features = false }
# ...
```

<!--
And somewhere in our dependency tree, another package has this:
-->

そして依存関係の中にはこんなパッケージもあるとしましょう:

```toml
# Another package's Cargo.toml
# 別のパッケージの Cargo.toml

[build-dependencies]
bstr = "0.2.16"
```

<!--
In our package, we've been using the [`words_with_breaks`](https://docs.rs/bstr/0.2.16/bstr/trait.ByteSlice.html#method.words_with_breaks) method from `bstr`, which requires `bstr`'s  "unicode" feature to be enabled.
This has historically worked because Cargo unified the features of `bstr` between the two packages.
However, after updating to Rust 2021, the new resolver will build `bstr` twice, once with the default features (as a build dependency), and once with no features (as our normal dependency).
Since `bstr` is now being built without the "unicode" feature, the `words_with_breaks` method doesn't exist, and the build will fail with an error that the method is missing.
-->

我々のパッケージでは、今までは `bstr` の [`words_with_breaks`](https://docs.rs/bstr/0.2.16/bstr/trait.ByteSlice.html#method.words_with_breaks) 関数を使用していたとします。この関数は(本来<!--訳注: わかりにくかったので勝手に入れました-->) `bstr` の "unicode" フィーチャを有効化しないと使えないものです。
歴史的事情から、今まではこれでもうまくいきました。というのも、Cargo は2つのパッケージで使われている `bstr` のフィーチャを共通化していたからです。
しかしながら、Rust 2021 へのアップデート後、 `bstr` は1回目(ビルド依存関係として)はデフォルトのフィーチャで、2回目(我々のパッケージの通常の依存先として)はフィーチャなしで、合計2回ビルドされます。
今や `bstr` は "unicode" フィーチャなしでビルドされるので、 `words_with_breaks` メソッドは存在せず、メソッドがないというエラーが発生してビルドは失敗します。

<!--
The solution here is to ensure that the dependency is declared with the features you are actually using.
For example:
-->

ここでの解決策は、依存関係の宣言に我々が実際に使っているフィーチャを書くようにすることです。

```toml
[dependencies]
bstr = { version = "0.2.16", default-features = false, features = ["unicode"] }
```

<!--
In some cases, this may be a problem with a third-party dependency that you don't have direct control over.
You can consider submitting a patch to that project to try to declare the correct set of features for the problematic dependency.
Alternatively, you can add features to any dependency from within your own `Cargo.toml` file.
For example, if the `bstr` example given above was declared in some third-party dependency, you can just copy the correct dependency declaration into your own project.
The features will be unified, as long as they match the unification rules of the new resolver. Those are:
-->

ときには、あなたが直接いじることのできないサードパーティな依存先で問題が発生することもあります。
その場合は、問題が起こっている依存関係について、正しくフィーチャを指定するように、そのプロジェクトにパッチを送るのもよいでしょう。
あるいは、自身の `Cargo.toml` に記載する依存関係にフィーチャを追加することもできます。
新しいリゾルバには以下のような併合ルールがあり、その下でフィーチャは併合されます。すなわち、

<!--
* Features enabled on platform-specific dependencies for targets not currently being built are ignored.
* Build-dependencies and proc-macros do not share features with normal dependencies.
* Dev-dependencies do not activate features unless building a target that needs them (like tests or examples).
-->

* 現在ビルドされていないターゲットに対するプラットフォーム特有の依存関係で有効化されているフィーチャは、無視されます。
* build-dependencies と proc-macro では、通常の依存関係とは独立したフィーチャが使用されます。
* dev-dependencies では、(tests や examples などの) ターゲットをビルドするときに必要でない限り、フィーチャは有効化されません。

<!--
A real-world example is using [`diesel`](https://crates.io/crates/diesel) and [`diesel_migrations`](https://crates.io/crates/diesel_migrations).
These packages provide database support, and the database is selected using a feature, like this:
-->

実際の例としては、[`diesel`](https://crates.io/crates/diesel) と [`diesel_migrations`](https://crates.io/crates/diesel_migrations) を使用する場合が挙げられます。
これらのパッケージはデータベースへのサポートを提供しますが、データベースはフィーチャを用いて選択されます。たとえば、こんな感じです:

```toml
[dependencies]
diesel = { version = "1.4.7", features = ["postgres"] }
diesel_migrations = "1.4.0"
```

<!--
The problem is that `diesel_migrations` has an internal proc-macro which itself depends on `diesel`, and the proc-macro assumes its own copy of `diesel` has the same features enabled as the rest of the dependency graph.
After updating to the new resolver, it fails to build because now there are two copies of `diesel`, and the one built for the proc-macro is missing the "postgres" feature.
-->

ここで問題なのは、 `diesel_migrations` は内部に `diesel` に依存する手続き的マクロをもちます。
この手続き的マクロは、自身が使用する `diesel` で有効化されているフィーチャが、依存関係木の他の場所で有効化されているものと同じであると仮定します。
ところが、新しいリゾルバが使用されると、2つの `diesel` が使用され、そのうち手続き的マクロ用のものは "postgres" フィーチャなしでビルドされるために、ビルドに失敗します。

<!--
A solution here is to add `diesel` as a build-dependency with the required features, for example:
-->

ここでの解決策は、`diesel` をビルド時の依存として追加し、そこに必要なフィーチャを指定することです。例えば以下のようになります。

```toml
[build-dependencies]
diesel = { version = "1.4.7", features = ["postgres"] }
```

<!--
This causes Cargo to add "postgres" as a feature for host dependencies (proc-macros and build-dependencies).
Now, the `diesel_migrations` proc-macro will get the "postgres" feature enabled, and it will build correctly.
-->

これにより、 Cargo はホスト依存関係（proc-macro と build-dependencies）のフィーチャとして "postgres" を追加します。

> 訳注：ホスト依存関係とは、コンパイラホスト（コンパイラを実行しているプラットフォーム）向けにビルド・実行される依存を指し、proc-macro クレートや build-dependencies 配下の依存クレートが該当します。
> 一方、通常の依存関係はコンパイルターゲットのプラットフォーム向けにビルドされます。

これで、 `diesel_migrations` の手続き的マクロは "postgres" フィーチャが有効化された状態で走り、正しくビルドされます。

<!--
The 2.0 release of `diesel` (currently in development) does not have this problem as it has been restructured to not have this dependency requirement.
-->

(現在開発中の) `diesel` のリリース 2.0 では、このような仮定なしに動くよう再設計されているため、このような問題は発生しません。

<!--
### Exploring features
-->

### フィーチャを探索する

<!--
The [`cargo tree`] command has had substantial improvements to help with the migration to the new resolver.
`cargo tree` can be used to explore the dependency graph, and to see which features are being enabled, and importantly *why* they are being enabled.
-->

[`cargo tree`] コマンドには、新しいリゾルバへの移行を補助する、素晴らしい新機能が含まれています。
`cargo tree` を使えば、依存関係木を探索して、どのフィーチャが有効化されているか、そしてなにより*なぜ*それが有効化されているのかが分かります。

<!--
One option is to use the `--duplicates` flag (`-d` for short), which will tell you when a package is being built multiple times.
Taking the `bstr` example from earlier, we might see:
-->

例えば、`--duplicates` (短縮形: `-d`) フラグを使用すると、同じパッケージが複数回ビルドされている場所がわかります。
さきほどの `bstr` を例に取れば、このような表示になるでしょう:

```console
> cargo tree -d
bstr v0.2.16
└── foo v0.1.0 (/MyProjects/foo)

bstr v0.2.16
[build-dependencies]
└── bar v0.1.0
    └── foo v0.1.0 (/MyProjects/foo)

```

<!--
This output tells us that `bstr` is built twice, and shows the chain of dependencies that led to its inclusion in both cases.
-->

この出力から、`bstr` が複数回ビルドされていることと、どの依存関係をたどると双方が現れるかが分かります。

<!--
You can print which features each package is using with the `-f` flag, like this:
-->

`-f` フラグを使えば、それぞれのパッケージがどのフィーチャを使用しているかがわかります。こんな感じです:

```console
cargo tree -f '{p} {f}'
```

<!--
This tells Cargo to change the "format" of the output, where it will print both the package and the enabled features.
-->

こうすると、Cargo は出力の「フォーマット」を変更して、パッケージと有効化されているフィーチャの双方を表示するようになります。

<!--
You can also use the `-e` flag to tell it which "edges" to display.
For example, `cargo tree -e features` will show in-between each dependency which features are being added by each dependency.
This option becomes more useful with the `-i` flag which can be used to "invert" the tree.
This allows you to see how features *flow* into a given dependency.
For example, let's say the dependency graph is large, and we're not quite sure who is depending on `bstr`, the following command will show that:
-->

さらに、`-e` フラグを使用してどの「辺」を表示してほしいか指定することもできます。
例えば、`cargo tree -e features` とすれば、各依存関係の間に、各依存関係がどのフィーチャを追加しているのかが表示されます。
`-i` フラグを使って木を「反転」させると、このオプションはより便利になります。
例えば、依存関係木があまりにも大きくて、何が `bstr` に依存してるのかよくわからなくても、次のコマンドを実行すればいいです:

```console
> cargo tree -e features -i bstr
bstr v0.2.16
├── bstr feature "default"
│   [build-dependencies]
│   └── bar v0.1.0
│       └── bar feature "default"
│           └── foo v0.1.0 (/MyProjects/foo)
├── bstr feature "lazy_static"
│   └── bstr feature "unicode"
│       └── bstr feature "default" (*)
├── bstr feature "regex-automata"
│   └── bstr feature "unicode" (*)
├── bstr feature "std"
│   └── bstr feature "default" (*)
└── bstr feature "unicode" (*)
```

<!--
This snippet of output shows that the project `foo` depends on `bar` with the "default" feature.
Then, `bar` depends on `bstr` as a build-dependency with the "default" feature.
We can further see that `bstr`'s  "default" feature enables "unicode" (among other features).
-->

この出力例からは、`foo` が `bar` に "default" フィーチャ付きで依存していることがわかり、
`bar` はビルド時の依存として `bstr` に "default" フィーチャ付きで依存していることもわかります。
さらに、`bstr` の "default" フィーチャによって "unicode" フィーチャ（と、他のフィーチャも）が有効になっていることもわかります。

<!--
[`cargo tree`]: ../../cargo/commands/cargo-tree.html
-->
[`cargo tree`]: https://doc.rust-lang.org/cargo/commands/cargo-tree.html
