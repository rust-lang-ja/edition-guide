<!--
# C-string literals
-->

# C 文字列リテラル

<!--
## Summary
-->

## 概要

<!--
- Literals of the form `c"foo"` or `cr"foo"` represent a string of type [`&core::ffi::CStr`][CStr].
-->
リテラル `c"foo"` や `cr"foo"` で、[`&core::ffi::CStr`][CStr] 型の文字列を表現できます。

<!--
[CStr]: ../../core/ffi/struct.CStr.html
-->

[CStr]: https://doc.rust-lang.org/core/ffi/struct.CStr.html

<!--
## Details
-->

## 詳細

<!--
Starting with Rust 1.77, C-strings can be written using C-string literal syntax with the `c` or `cr` prefix.
-->

Rust 1.77 から、`c` や `cr` から始まる C 文字列リテラル構文を用いて C 文字列を表現できるようになりました。

<!--
Previously, it was challenging to properly produce a valid string literal that could interoperate with C APIs which terminate with a NUL byte.
The [`cstr`] crate was a popular solution, but that required compiling a proc-macro which was quite expensive.
Now, C-strings can be written directly using literal syntax notation, which will generate a value of type [`&core::ffi::CStr`][CStr] which is automatically terminated with a NUL byte.
-->

以前は、C 言語の API と相互運用できるようにヌルバイトで終端された文字列リテラルを正しく作成することは困難でした。
それまでは [`cstr`] クレートが広く使われていましたが、重い処理である proc macro のコンパイルが必要でした。
今回導入されたリテラル構文により、暗黙にヌルバイトで終端された [`&core::ffi::CStr`][CStr] 型の C 文字を直接記述できるようになりました。

```rust,edition2021
# use core::ffi::CStr;

assert_eq!(c"hello", CStr::from_bytes_with_nul(b"hello\0").unwrap());
// ↓ 訳: バイトエスケープ \xff も使えます
assert_eq!(
    c"byte escapes \xff work",
    CStr::from_bytes_with_nul(b"byte escapes \xff work\0").unwrap()
);
// ↓ 訳: ユニコードエスケープ \u{00E6} も使えます
assert_eq!(
    c"unicode escapes \u{00E6} work",
    CStr::from_bytes_with_nul(b"unicode escapes \xc3\xa6 work\0").unwrap()
);
// ↓ 訳: ユニコード文字 αβγ は UTF-8 で表現されます
assert_eq!(
    c"unicode characters αβγ encoded as UTF-8",
    CStr::from_bytes_with_nul(
        b"unicode characters \xce\xb1\xce\xb2\xce\xb3 encoded as UTF-8\0"
    )
    .unwrap()
);
// ↓ 訳: 文字列は複数行にわたることができます
assert_eq!(
    c"strings can continue \
        on multiple lines",
    CStr::from_bytes_with_nul(b"strings can continue on multiple lines\0").unwrap()
);
```

<!--
C-strings do not allow interior NUL bytes (such as with a `\0` escape).
-->

C 文字列は内部にヌル文字（エスケープ `\0` など）を含めることはでいません。

<!--
Similar to regular strings, C-strings also support "raw" syntax with the `cr` prefix.
These raw C-strings do not process backslash escapes which can make it easier to write strings that contain backslashes.
Double-quotes can be included by surrounding the quotes with the `#` character.
Multiple `#` characters can be used to avoid ambiguity with internal `"#` sequences.
-->

C 文字列では通常の文字列と同様、`cr` で「生」構文を使うこともできます。
生 C 文字列はバックスラッシュ `\` によるエスケープが効かないので、バックスラッシュを含む文字列を書きたいときに有効です。
`#` で囲むことで、二重引用符 `"` も含めることもできます。
`#` を複数使えば、`"#` が内部に含まれる文字列も表現可能です。

<!--
```rust,edition2021
assert_eq!(cr"foo", c"foo");
// Number signs can be used to embed interior double quotes.
assert_eq!(cr#""foo""#, c"\"foo\"");
// This requires two #.
assert_eq!(cr##""foo"#"##, c"\"foo\"#");
// Escapes are not processed.
assert_eq!(cr"C:\foo", c"C:\\foo");
```
-->

```rust,edition2021
assert_eq!(cr"foo", c"foo");
// 記号 # を使えば、 " を含められます。
assert_eq!(cr#""foo""#, c"\"foo\"");
// この文字列の場合、 # が2つ必要です。
assert_eq!(cr##""foo"#"##, c"\"foo\"#");
// エスケープは無効です。
assert_eq!(cr"C:\foo", c"C:\\foo");
```

<!--
See [The Reference] for more details.
-->

詳しくは[言語リファレンス]を参照してください。

<!--
[`cstr`]: https://crates.io/crates/cstr
[The Reference]: ../../reference/tokens.html#c-string-and-raw-c-string-literals
-->

[`cstr`]: https://crates.io/crates/cstr
[言語リファレンス]: https://doc.rust-lang.org/reference/tokens.html#c-string-and-raw-c-string-literals

<!--
## Migration
-->

## 移行

<!--
Migration is only necessary for macros which may have been assuming a sequence of tokens that looks similar to `c"…"` or `cr"…"`, which previous to the 2021 edition would tokenize as two separate tokens, but in 2021 appears as a single token.
-->

移行が必要なケースはまれで、`c"…"` や `cr"…"` のようなトークン列を含むマクロ呼び出しだけ変更の必要があります。
2021 エディションより前ではこれらは2トークンに分割されましたが、2021 では単一トークンと解釈されるからです。

<!--
As part of the [syntax reservation] for the 2021 edition, any macro input which may run into this issue should issue a warning from the `rust_2021_prefixes_incompatible_syntax` migration lint.
See that chapter for more detail.
-->

2021 エディションで[構文が予約]されるにあたり、問題となりうるマクロ呼び出しは、移行リント`rust_2021_prefixes_incompatible_syntax` で警告が出るようになっています。
詳しくは当該の節を参照してください。

<!--
[syntax reservation]: reserved-syntax.md
-->

[構文が予約]: reserved-syntax.md
