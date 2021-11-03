<!--
# TryFrom and TryInto
-->
# TryFromとTryInto

<!--
Initially added: ![Minimum Rust version: 1.34](https://img.shields.io/badge/Minimum%20Rust%20Version-1.34-brightgreen.svg)
-->
![Minimum Rust version: 1.34](https://img.shields.io/badge/Minimum%20Rust%20Version-1.34-brightgreen.svg)で最初に導入されました。

<!--
The [`TryFrom`](../../std/convert/trait.TryFrom.html) and
[`TryInto`](../../std/convert/trait.TryInto.html) traits are like the
[`From`](../../std/convert/trait.From.html) and
[`Into`](../../std/convert/trait.Into.html) traits, except that they return a
result, meaning that they may fail.
-->
[`TryFrom`](../../std/convert/trait.TryFrom.html)と[`TryInto`](../../std/convert/trait.TryInto.html)トレイトは[`From`](../../std/convert/trait.From.html)と[`Into`](../../std/convert/trait.Into.html)トレイトと似ていますが、Result型を返すという点で異なっています。
つまり、これらの呼び出しは失敗することがあります。

<!--
For example, the `from_be_bytes` and related methods on integer types take
arrays, but data is often read in via slices. Converting between slices and
arrays is tedious to do manually. With the new traits, it can be done inline
with `.try_into()`:
-->
例えば、整数型の`from_be_bytes`や関連するメソッドは配列を引数に取りますが、そのデータはスライスから読み込まれるということがよくあります。スライスから配列への変換は手動でやると退屈なものです。
このトレイトを使えば、`try_into()`でインラインでできます。

```rust
use std::convert::TryInto;
# fn main() -> Result<(), Box<dyn std::error::Error>> {
# let slice = &[1, 2, 3, 4][..];
let num = u32::from_be_bytes(slice.try_into()?);
# Ok(())
# }
```
