<!--
# Cargo changes
-->

# Cargo への変更

<!--
## Summary
-->

## 概要

<!--
- If there is a target definition in a `Cargo.toml` manifest, it no longer
  automatically disables automatic discovery of other targets.
- Target paths of the form `src/{target_name}.rs` are no longer inferred for
  targets where the `path` field is not set.
- `cargo install` for the current directory is no longer allowed, you must
  specify `cargo install --path .` to install the current package.
-->

- `Cargo.toml` にターゲットの指定がある場合であっても、他のターゲットの自動探索がされなくなるということはなくなりました[^1]。
- `path` フィールドが指定されていないターゲットに対して、`src/{target_name}.rs` の形のターゲットパスは自動設定されなくなりました[^2]。
- 現在のディレクトリに対して `cargo install` できなくなりました。現在のパッケージをインストールしたい場合は `cargo install --path .` と指定する必要があります[^3]。

[^1] *訳注*:
  Cargo は、`Cargo.toml` 内に明示的に指定されていなくても、[フォルダ構成の慣習][package-layout]に従っているファイルに関しては、自動でターゲットに追加します。
  例えば、`src/bin/my_application.rs` というファイルがプロジェクト内に存在したら、`Cargo.toml` に `[[bin]]` セクションで `my_application` の存在が宣言されていなくても、自動的にこのファイルがバイナリターゲット（つまり、`cargo run --bin my_application` として実行できるもの）としてビルドされるようになっています。
  これは、[*ターゲットの自動探索*][target-auto-discovery]と呼ばれています。<br>
  Rust 2015 と Rust 2018 の違いは、`Cargo.toml` に明示的にターゲットが1つ以上宣言されている場合（つまり、`[lib]`, `[[bin]]`, `[[test]]`, `[[bench]]`, `[[example]]` のどれかが1つ以上ある場合）に生じます。
  Rust 2015 では、これらのうちどれか1つが指定されていた場合、ターゲットの自動探索は無効化されました。
  Rust 2018 では、その場合であっても、ターゲットの自動探索は無効化されません。<br>
  詳細は、[Cargo Book の "Target auto-discovery"（英語）][target-auto-discovery]もご覧ください。

[^2] *訳注*:
  [WIP] 詳細は不明です。

[^3] *訳注*:
  [`cargo install`][cargo-install] コマンドは、指定したクレートのバイナリターゲットをインストールする、つまりビルドして実行ファイルを所定の場所に配置するためのコマンドです。
  Rust 2015 では、カレントディレクトリが Cargo プロジェクト下であったときに `cargo install` とだけ実行すると、そのプロジェクトに含まれるクレートを対象としてインストールが実行されました。
  Rust 2018 ではこの挙動は廃止され、`cargo install --path .` と明示的にカレントディレクトリのクレートをインストールの対象にすると宣言しなければならなくなりました。

[package-layout]: https://doc.rust-lang.org/cargo/guide/project-layout.html
[target-auto-discovery]: https://doc.rust-lang.org/cargo/reference/cargo-targets.html#the-path-field
[cargo-install]: https://doc.rust-lang.org/cargo/commands/cargo-install.html
