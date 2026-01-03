# 今日の豆知識.8

今日はJavaScript(以降JS)についてです。

## 歴史

開発当初はMochaという名称<br>
↓<br>
その後LiveScriptという名称に変わる<br>
↓<br>
1995年12月に当時人気だったJavaにあやかってJavaScriptという名前が付けられてNetscape Navigator 2.0に実装されました<br>
ちなみに"JavaScript"という名称はSun Microsystemsが商標登録していてSun MicrosystemsはOracleに買収されているので現在はOracleが商標権を持ってます<br>
↓<br>
ECMAScript標準化されたことでWebブラウザーさえあれば動かせるようになることが期待されたが,拡張機能が条件付きで認められていたためブラウザによって機能に違いがあった<br>
↓<br>
jQueryというライブラリでブラウザ間の差が吸収されてJSがより一層普及した<br>
↓<br>
Google Chromeが作られてそれにJIT(Just In Time)コンパイラ(実行時コンパイラ)が搭載される<br>
↓<br>
どのWebで動く言語JSぐらいしかないけどJSの進化遅いからJSにコンパイルできる言語つくろうぜっていう感じでCoffeeScriptとかが作られる。このようにJSにコンパイルされる言語をaltJSと呼ぶ。このころから**言語→言語**へコンパイルすることを**トランスパイル**と呼ぶようになる。(JSにコンパイルって言葉がわからないかもしれないのでもっと簡単に言うと**新しい言語A**を**JS**へ変換してから実行するみたいなこと)<br>
↓<br>
Node.jsが登場したことでWebのフロントエンドだけでなくバックエンドでもJSを使えるようになった<br>
↓<br>
JSのライバル的な存在だったFlashがiOSでサポート打ち切ったこともありWebのフロントエンドではJSが一強となった<br>
↓<br>
Reactがこの辺りで登場 <- Reactはあとで説明する<br>

## 特徴

- HTMLに埋め込んで書ける (<script></script> で囲んだ中に書く)
- 動的型付け
- JavaScriptエンジンで動く
- GCによってメモリ管理されている
- アロー関数を使って無名関数が使える
- `==`での比較では例えば`"10" == 10`が`true`だが`===`での比較では型の違いも比較されるため`"10" === 10`は`false`になる
- プロトタイプチェーンベースのオブジェクト指向プログラミング(以降OOP = オブジェクト指向プログラミング)(C#/C++/JavaなどはクラスベースのOOP)(JSもclassはあとから実装され存在するが内部的にはプロトタイプチェーンであり,糖衣構文に過ぎない)
- JSON(JavaScript Object Notation)(JavaScriptにおけるオブジェクトの表記法に由来するデータ記述言語)がいろんなところで世界標準になっている

これの例として(あとWebってOSによらずに動作させることができるからその辺が便利だったりしてネイティブで動いてるように見えるものもWebの技術が使われてたりしてWeb系らしくないWebの使い道もある)
## Reactについて簡単に
超簡単に言うとUI作るのめっちゃ簡単にしたぜ!ってやつ。React自体はhtmlを動的に生成している感じでネイティブではなくWebで動く
### React Nativeについて超簡単に
超簡単に言うとReactをネイティブで使えるようにしました!ってやつ

なんでこれを紹介したかというとDiscordで使われているからです
開発は便利だけどパフォーマンス面で問題があるみたいです
> Discord's desktop and mobile clients are built with React & React Native respectively. Using React enables our teams to ship features rapidly across different platforms, yet requires careful attention to performance optimization as there’s less margin for error.
> 
> Initially, we shied away from using React Native on Android due to performance concerns, but recent advances in Android device capabilities and the introduction of Hermes, React Native's new JavaScript engine, changed the landscape. This led us to transition our Android client to React Native in 2022.
> 
> While the switch came with performance trade-offs (particularly startup times on lower-end devices) we tackled these challenges by cutting our median startup times in half during 2023 – a story we'll detail in a future post.
> 
> Most recently, we've turned our attention to optimizing the things that people use the most, with a particular focus on power users who push Discord's capabilities to (and sometimes beyond) their limits. If you're reading this, you're probably one of 'em!

https://discord.com/blog/supercharging-discord-mobile-our-journey-to-a-faster-app

## TypeScriptについて

歴史のところで
> どのWebで動く言語JSぐらいしかないけどJSの進化遅いからJSにコンパイルできる言語つくろうぜっていう感じでCoffeeScriptとかが作られる。このようにJSにコンパイルされる言語をaltJSと呼ぶ。このころから言語→言語へコンパイルすることをトランスパイルと呼ぶようになる。(JSにコンパイルって言葉がわからないかもしれないのでもっと簡単に言うと新しい言語AをJSへ変換してから実行するみたいなこと)
と書いたところで出てきたJSにコンパイルされる言語として**TypeScript**(以降TS)というものがあります。(これは思想的にJSを置き換えようとするaltJSというよりも既存のJSのコードはすべてコンパイルを通せるのでJSのスーパーセットというほうが正しい)

JSで動くコードはすべてTSでも動きますが,違いとしてはTSは文法としてtype annotation(型注釈)が存在し,JSにコンパイルするときに型の食い違いがあったらエラーを出せます。<br>
例えば
JSでは足し算をする関数を以下のように作ったとして,引数に数字ではないものを渡していますが,エラーにはなりません
```js
function add(a, b) {
  return a + b;
}

add(1, 2);
add(1, "2"); // 文字列の2を渡してしまっている
```
一方TSでは以下のように何の型であるかを定義することができ,このように型注釈をつけていればコンパイル時にエラーを出すことができます。
```ts
function add(a: number, b: number): number {
  return a + b;
}

add(1, 2);
add(1, "2"); // エラー
```
```
srcts/main.ts:6:8 - error TS2345: Argument of type 'string' is not assignable to parameter of type 'number'.

6 add(1, "2"); // エラー
         ~~~


Found 1 error in srcts/main.ts:6
```
プログラミング初めてすぐのときは**エラーなんて出ないで動いてくれた方がいいだろ**と思ってしまう人もいるらしいですが,エラーをコンパイル時に出してくれるというのはつまりミスを実際に動かす前に見つけてくれて,コンパイルを通ったならある程度動作が担保されるわけですし,どこを間違えているかの原因がすぐわかるのでとても大切なことなんです。(Undefined Behavior(以降UB)が多い言語が嫌われる理由)

さらに型注釈があることはエラーを出せる以外にも大きなメリットが2つあります。
- コードが読みやすくなる
- AIがコード生成/修正しやすい
　一つ目については,プログラムコードは書かれる時間よりも読まれる時間の方が長いとよく言われ, コードを読みやすく書くことはとても重要であり, 型注釈によってそのコードの意図がより伝わりやすくなります。一応JSでも多くの会社で独自のルールを設けてコメントとして型を書くようにしているらしいですが, TSによって型注釈が文法に組み込まれたことで,書き方も統一されたという点も大きいです。
　二つ目に関しては最近言われるようになったものです。基本的にAIにコードを書かせるってなったらAI自身にそのコードをチェックさせたりエラーが出たらエラーメッセージを渡すみたいなことをよくやると思います。この時に型注釈やエラーメッセージは定型的かつより具体的な情報であり,AIが精度を高い回答をしやすいです。

ちなみにTSをコンパイルして生成されるJSのコードは型情報は持ちません。そのため,静的型付けではあるものの,動作しているときはJSとまったく同じ動作をします。

## Hello, world!

JS/TS共通
```js
console.log("Hello, world!");
```

## コード例

**フィボナッチ数列の第n項を返す関数**
JS
```js
function fibo(n) {
    let a = 0;
    let b = 1;

    for (let i = 0; i < n; i++) {
        const tmp = b;
        b = a + b;
        a = tmp;
    }

    return a;
}

console.log(fibo(10)); 
```
TS
```ts
function fibo(n: number): number {
    let a: number = 0;
    let b: number = 1;

    for (let i: number = 0; i < n; i++) {
        const tmp: number = b;
        b = a + b;
        a = tmp;
    }

    return a;
}

console.log(fibo(10));
```