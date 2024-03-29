# 原則

* カタカナ語のままで違和感のない用語はカタカナ語のまま使う
  + 気持としては「無理矢理和訳」を避けたい。そのための基準。
  + カタカナ語の方が用語として認識しやすい
* 3音以上のカタカナ語の末尾の長音記号「ー」は省く（[JIS Z 8301:2011](http://kikakurui.com/z8/Z8301-2011-01.html) G.6.2.2 b 表3を参照）
* 構文キーワードなどはそのままアルファベットを使う

# 対訳表

| English                        | 日本語
|:-------------------------------|:-------------
| abort                          | アボート
| (lockの) acquire               | 獲得
| aggregate type                 | 合成型
| alignment                      | アラインメント
| allocate                       | アロケートする
| allocation                     | アロケーション
| allocator                      | アロケータ
| anonymous parameter            | 無名パラメータ
| antipattern                    | アンチパターン
| application                    | アプリケーション
| arity                          | アリティ
| (マッチの)arm                  | 腕
| array                          | 配列
| assignment                     | 代入
| associated -                   | 関連-
| atomic                         | アトミック
| attribute                      | アトリビュート
| binary                         | バイナリ
| binding                        | 束縛
| block                          | ブロック
| body (関数の)                  | 本体
| borrowing                      | 借用
| bounds                         | 境界
| boxed                          | ボックス化された
| bug                            | バグ
| build-dependencies             | ビルド時の依存 (Cargo.toml のセクション名としては build-dependencies をそのまま採用)
| byte string                    | バイト列
| call                           | 呼び出し
| capture                        | キャプチャ
| cargo                          | Cargo
| case analysis                  | 場合分け
| channel                        | チャネル
| closure                        | クロージャ
| coercion                       | 型強制
| code bloat                     | コードの膨張
| combinator                     | コンビネータ
| comma                          | カンマ
| command line                   | コマンドライン
| compile-time error             | コンパイル時エラー
| compiler                       | コンパイラ
| compound data type             | 複合データ型
| composable                     | 合成可能
| computer science               | コンピュータサイエンス
| concurrency                    | 並行性
| conditional compilation        | 条件付きコンパイル
| constant                       | 定数
| constructor                    | コンストラクタ
| continuous integration         | 継続的インテグレーション
| crate                          | クレート
| dangling                       | ダングリング
| data race                      | データ競合
| deadlock                       | デッドロック
| deallocate                     | デアロケートする
| declaration statement          | 宣言文
| dependency                     | 依存関係、依存先
| dependency tree                | 依存関係木
| dereferencing                  | 参照外し
| destructor                     | デストラクタ
| destructuring                  | 分配
| dev-dependencies               | 開発用依存関係 (Cargo.toml のセクション名としては dev-dependencies をそのまま採用)
| directive                      | ディレクティブ
| directory                      | ディレクトリ
| discriminant                   | 判別子
| distribution                   | 配布物
| diverge                        | 発散する
| diverging                      | 発散する〜（上の diverge を修飾語として使った場合）
| documentation comment          | ドキュメンテーションコメント
| documentation test             | ドキュメンテーションテスト
| drop                           | ドロップ
| dynamic dispatch               | 動的ディスパッチ
| early return                   | 早期リターン
| edition                        | エディション
| empty tuple                    | 空タプル
| encode                         | エンコード
| entry point                    | エントリポイント
| enum                           | 列挙型
| equality                       | 等値性
| ergonomic                      | エルゴノミック（人間にとって扱いやすいもの）
| error                          | エラー
| error handling                 | エラーハンドリング
| examples                       | examples  (cargo のプロジェクトレイアウト上におけるフォルダ名として使われている場合のみ)
| executable                     | 実行可能形式
| existentially quantified type  | 存在量型
| expression statement           | 式文
| exterior                       | 外側の
| feature                        | フィーチャ
| foreign                        | 他言語
| (マクロの) fragment specifier  | フラグメント指定子
| free                           | 解放する
| free-standing function         | フリースタンディングな関数
| fully qualified syntax         | 完全修飾構文
| garbage collector              | ガベージコレクタ
| generic parameter              | ジェネリックパラメータ
| generics                       | ジェネリクス
| glob                           | グロブ
| growable                       | 伸張可能
| guard                          | ガード
| handle                         | ハンドル
| hard error                     | ハードエラー
| hash                           | ハッシュ
| identifier                     | 識別子
| immutable                      | イミュータブル
| immutability                   | イミュータビリティ
| implement                      | 実装する
| inherent method                | 固有メソッド
| initialize                     | 初期化する
| input lifetime                 | 入力ライフタイム
| interior                       | 内側の
| install                        | インストール
| installer                      | インストーラ
| interpolate                    | 補間する
| (string) interpolation         | （文字列）補間
| Intrinsics                     | Intrinsic
| irrefutable                    | 論駁不可能
| item                           | アイテム
| iterate                        | 列挙する
| key                            | キー
| keyword                        | キーワード
| Lang Items                     | Lang Item
| leak                           | リーク
| lending                        | 貸付け
| library                        | ライブラリ
| lifetime                       | ライフタイム
| lifetime elision               | ライフタイムの省略
| lifetime parameter             | ライフタイムパラメータ
| link                           | リンク
| lint                           | リント
| literal                        | リテラル
| lock                           | ロック
| maintainer                     | メンテナ
| mangling                       | マングリング
| match                          | マッチ
| match guards                   | マッチガード
| matcher                        | マッチャ
| memory                         | メモリ
| method                         | メソッド
| monomorphization               | 単相化
| move                           | ムーブ(する)
| move out                       | ムーブする、ムーブアウトする
| mutability                     | ミュータビリティ
| mutable                        | ミュータブル
| mutable binding                | ミュータブルな束縛
| mutate                         | 変更する、書き換える
| mutual-exclusion               | 相互排他
| null                           | ヌル
| object-safe                    | オブジェクト安全
| offline                        | オフライン
| opaque                         | オペーク
| open source                    | オープンソース
| operator                       | 演算子
| option                         | オプション
| output lifetime                | 出力ライフタイム
| overflow                       | オーバーフロー
| owned                          | 所有権を持った
| owner                          | 所有者
| ownership                      | 所有権
| panic                          | パニック
| parameter                      | パラメータ
| parametric polymorphism        | パラメトリック多相
| parse                          | パース、パースする
| patch                          | パッチ
| path                           | パス
| pattern                        | パターン
| performance                    | パフォーマンス
| platform                       | プラットフォーム
| primitive                      | プリミティブ
| pointer                        | ポインタ
| prelude                        | プレリュード (or prelude)
| proc macro                     | 手続き的マクロ
| process                        | プロセス
| range                          | 範囲
| raw identifier                 | 生識別子
| raw pointer                    | 生ポインタ
| raw string literal             | 生文字列リテラル
| re-assignment                  | 再代入
| rebind                         | 再束縛
| reference (名詞)               | 参照
| reference (動詞)               | 参照付け (to dereference の対義語として)
| reference count                | 参照カウント
| refutable                      | 論駁可能
| regression                     | リグレッション
| release                        | リリース
| (lockの) release               | 解放
| resolver                       | リゾルバ
| return                         | 返す
| return type                    | リターン型
| return value                   | 戻り値
| runtime                        | 実行時
| safe                           | 安全
| safety check                   | 安全性検査
| scope                          | スコープ
| scoped                         | スコープ化された
| script                         | スクリプト
| semantics                      | セマンティクス
| shadow                         | 覆い隠す
| shadowing                      | シャドーイング
| signature                      | シグネチャ
| signed                         | 符号付き
| slice                          | スライス
| slicing                        | スライシング
| specialized                    | 特殊化された
| stablize                       | 安定化する
| standard library               | 標準ライブラリ
| statement                      | 文
| static dispatch                | 静的ディスパッチ
| strict keyword                 | 正格キーワード
| string                         | 文字列
| string interpolation           | 文字列インターポーレーション
| string slice                   | 文字列スライス
| struct                         | 構造体
| structure                      | ストラクチャ
| sum type                       | 直和型
| symbol                         | シンボル
| syntactic sugar                | 糖衣構文
| syntax tree                    | 構文木
| system                         | システム
| tagged union                   | タグ付き共用体
| term                           | 項
| tests                          | tests  (cargo のプロジェクトレイアウト上におけるフォルダ名として使われている場合のみ)
| third-party                    | サードパーティ
| thread-locality                | スレッドローカル性
| threadsafe                     | スレッドセーフ
| tick                           | クオート
| trait                          | トレイト
| tracking issue                 | 追跡用の Issue
| tuple                          | タプル
| token trees                    | トークン木
| type alias                     | 型エイリアス
| type erasure                   | 型消去
| type family                    | 型族
| type inference                 | 型推論
| type parameter                 | 型パラメータ
| uninstall                      | アンインストール
| unit  注: `()` の読み          | ユニット
| Universal Function Call Syntax | 共通の関数呼び出し構文
| unsafe                         | アンセーフ
| unsigned                       | 符号無し
| unsized type                   | サイズ不定型
| unwinding                      | 巻き戻し
| unwrap                         | アンラップ
| value constructor              | 値コンストラクタ
| variable                       | 変数
| variable binding               | 変数束縛
| variant                        | ヴァリアント
| vector                         | ベクタ
| version                        | バージョン
| virtual workspace              | 仮想ワークスペース
| warning                        | 警告
| wildcard                       | ワイルドカード
| workspace                      | ワークスペース
| wrapper                        | ラッパ
