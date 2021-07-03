<!--
# Alternative Cargo registries
-->
# Cargoレジストリが選択できるように

<!--
Initially added: ![Minimum Rust version: 1.34](https://img.shields.io/badge/Minimum%20Rust%20Version-1.34-brightgreen.svg)
-->
![Minimum Rust version: 1.34](https://img.shields.io/badge/Minimum%20Rust%20Version-1.34-brightgreen.svg)で最初に導入されました。

<!--
For various reasons, you may not want to publish code to crates.io, but you
may want to share it with others. For example, maybe your company writes Rust
code that's not open source, but you'd still like to use these internal
packages.
-->
自分のコードを他の人とシェアしたいけど、何らかの理由でcrates.ioには公開したくないということがあるかも知れません。
例えば、あなたの会社ではオープンソースではないRustのコードを書いていて、でもその内部パッケージを使いたいという場合です。

<!--
Cargo supports alternative registries by settings in `.cargo/config`:
-->
Cargoは`.cargo/config`に設定を入れることで別のレジストリをサポートします。

```toml
[registries]
my-registry = { index = "https://my-intranet:7878/git/index" }
```

<!--
When you want to depend on a package from another registry, you add that
in to your `Cargo.toml`:
-->
そして、別のレジストリのパッケージに依存したい時は、`Cargo.toml`にそれを追加します。

```toml
[dependencies]
other-crate = { version = "1.0", registry = "my-registry" }
```

<!--
To learn more, check out the [registries section of the Cargo
book](https://doc.rust-lang.org/nightly/cargo/reference/registries.html).
-->
詳しく知りたい場合は、[Cargoブックのレジストリのセクション](https://doc.rust-lang.org/nightly/cargo/reference/registries.html)を確認してみてください。
