# 今日の豆知識.4

変数や構造体や関数などの命名規則(慣習的なもの)は言語によって結構違う

区切り方でよくありもの
| 形式                     | 例          | 呼び方      |
| ---------------------- | ---------- | -------- |
| `camelCase`            | `myValue`  | キャメルケース  (ローワーキャメルケース)|
| `PascalCase`           | `MyValue`  | パスカルケース  (アッパーキャメルケース)|
| `snake_case`           | `my_value` | スネークケース  (ローワースネークケース)|
| `SCREAMING_SNAKE_CASE` | `MAX_SIZE` |( アッパースネークケース)    |
| `kebab-case`           | `my-value` | ケバブケース |

接頭・接尾記号でよくあるもの
| 記号       | 代表的意味              |
| -------- | ------------------ |
| `_name`  | 内部用 / 未使用 / プライベート |
| `__name` | 特殊用途（予約・マジック）      |
| `mName`  | メンバ変数              |
| `IName`  | インターフェース           |
| `T`      | 型パラメータ             |

代表的な言語での例

## C/C++
規則
|対象|命名|
|-|-|
|変数・関数|`snake_case`|
|型・構造体|`PascalCase or snake_case`|
|定数| `SCREAMING_SNAKE_CASE` |

`_`で始めるものの意味

```c
int _internal;      // 慣習的に内部用
int __reserved;    // 規格上「予約」
```
## C\#
規則（Microsoft公式）
| 対象            | 命名           |
| ------------- | ------------ |
| クラス / 構造体     | `PascalCase` |
| メソッド          | `PascalCase` |
| ローカル変数        | `camelCase`  |
| private フィールド | `_camelCase` |
```cs
private int _count;
public int Count { get; }
```
`_`で始めるものの意味
明確に「privateフィールド」
未使用変数は`_` 1文字
```cs
var _ = SomeMethod(); // 使わない値
```

## Rust
規則（rustfmt & lint前提）
| 対象       | 命名                     |
| -------- | ---------------------- |
| 変数 / 関数  | `snake_case`           |
| 型 / トレイト | `PascalCase`           |
| 定数       | `SCREAMING_SNAKE_CASE` |
| ライフタイム   | `'a`                   |
`_` の意味
```rust
let _x = 10;   // 未使用（警告抑制）
let _ = foo(); // 完全に捨てる
```

## Python
規則（PEP8）
| 対象      | 命名                     |
| ------- | ---------------------- |
| 変数 / 関数 | `snake_case`           |
| クラス     | `PascalCase`           |
| 定数      | `SCREAMING_SNAKE_CASE` |
`_`の意味
```python
_x        # 内部用
__x       # 名前マングリング
__init__  # マジックメソッド
_, y = func()  # 使わない値
```
## Java
規則
| 対象        | 命名                     |
| --------- | ---------------------- |
| クラス       | `PascalCase`           |
| メソッド / 変数 | `camelCase`            |
| 定数        | `SCREAMING_SNAKE_CASE` |
`_` の扱い
Java 9以降：_ 単体は 予約語
基本的に _ は非推奨

## JavaScript / TypeScript
規則
| 対象      | 命名                     |
| ------- | ---------------------- |
| 変数 / 関数 | `camelCase`            |
| クラス     | `PascalCase`           |
| 定数      | `SCREAMING_SNAKE_CASE` |
`_`の意味
```ts
function f(_unused: number) {}
```
未使用引数
ライブラリ内部用

## WapL
ちなみにWapL(僕の言語)では基本的にRustと同じ命名規則に従っています
また,メソッド的に使う関数に関しては`型名_`で必ず始めるようにしてます(Iteratorとか頻繁に使うけど長いやつは`iter`みたいに小文字にして省略したのを使ってる)
また`__`に関してはC同様予約語