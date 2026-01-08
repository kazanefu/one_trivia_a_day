# 今日の豆知識.15

今日はイテレータについて説明します。

イテレータは、配列などのコレクションに順にアクセスするためのものです。多くの場合`next()`メソッドを呼び出すことで次の要素にアクセスできます。

今日は主にRustを使って説明するので前提にしたいRustの構文を一つだけ先に説明しておきます。
### Rustの構文
Rustには`Option`という型があります。これは値が存在するか、または存在しないかを表す型です。つまり`Option`は`Some(value)`または`None`の2つの場合があります。
Option型に対して`.unwarp()`を呼び出すと、`Some(value)`の場合は`value`を返し、`None`の場合はパニック(プログラムを停止)を起こします。Rustでは`.next()`メソッドの戻り値は`Option<T>`なので、`.unwrap()`を呼び出すことで値を取得できます。
```rust
fn main(){
    let option:Option<i32> = Some(1);
    println!("{}", option.unwrap()); // 1を表示する
    let option:Option<i32> = None;
    println!("{}", option.unwrap()); // Noneなのでパニックを起こす
}
```
```
1
thread 'main' panicked at 'called `Option::unwrap()` on a `None` value', src/main.rs:4:24
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace
```

## 本題
では本題に戻ります。

とりあえずイテレータはどんな感じのものなのか見てみましょう。
```rust
fn main(){
    // こんな感じで消費していきながら順にアクセスできる
    let mut iter = (1..=5).into_iter(); // イテレータを作成
    println!("{}", iter.next().unwrap()); // 1を表示する
    println!("{}", iter.next().unwrap()); // 2を表示する
    println!("{}", iter.next().unwrap()); // 3を表示する
    println!("{}", iter.next().unwrap()); // 4を表示する
    println!("{}", iter.next().unwrap()); // 5を表示する
    println!("{}", iter.next().unwrap()); // Noneなのでパニックを起こす
}
```
```
1
2
3
4
5
thread 'main' panicked at 'called `Option::unwrap()` on a `None` value', src/main.rs:6:24
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace
```
こんな感じで順にアクセスしていき全てに到達したら`None`を返します。

この時どのようなことが起こっているのかというと
```
| 1 | 2 | 3 | 4 | 5 |--| // こういうデータがあって

以下の↑はポインタだとして
.next()を呼び出すと
| 1 | 2 | 3 | 4 | 5 |--|
 ↑
最初はここから読む
Some(&1)を返す(&は参照を表す。つまり返しているのは↑の場所(ポインタ))
読む場所を一つ進める
| 1 | 2 | 3 | 4 | 5 |--|
      ↑

もう一度.next()を呼び出すと
Some(&2)を返す
読む場所を一つ進める
| 1 | 2 | 3 | 4 | 5 |--|
          ↑

これを繰り返して最終的に5を取り出した後
| 1 | 2 | 3 | 4 | 5 |--|
                      ↑
このように何もない場所を指すようになる
こうなると値がないのでNoneを返す
```
こんな感じのことが起こっています。

ではわかりやすいように`↑`をポインタではなく配列のインデックスとして自分でイテレータみたいなものを作ってみましょう。
```rust
// データとインデックスを持った構造体をイテレータとして使う
struct MyIterator<'a, T> {
    data: &'a [T],
    index: usize,
}
// MyIteratorにメソッドと関連関数を定義していく
impl<'a, T> MyIterator<'a, T> {
    // dataから新しくMyIteratorを生成する
    fn new(data: &'a [T]) -> Self {
        Self { data, index: 0 }
    }

    // nextメソッドを定義する
    fn next(&mut self) -> Option<&T> {
        // indexがデータの長さ以上になったらNoneを返す
        if self.index >= self.data.len() {
            None
        } else {
            // indexの場所の値を返す
            let value = &self.data[self.index];
            // indexを一つ進める
            self.index += 1;
            // Someで包んで返す
            Some(value)
        }
    }
}

fn main() {
    let data = [1, 2, 3, 4, 5];
    let mut iter = MyIterator::new(&data);
    println!("{}", iter.next().unwrap());
    println!("{}", iter.next().unwrap());
    println!("{}", iter.next().unwrap());
    println!("{}", iter.next().unwrap());
    println!("{}", iter.next().unwrap());
    println!("{}", iter.next().unwrap());
}
```
```
1
2
3
4
5

thread 'main' (81220) panicked at src/main.rs:37:32:
called `Option::unwrap()` on a `None` value
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace
```

### 課題: 実際に自分で書いてみた方が理解が深まると思うので皆さんがいつも使っている言語でイテレータを作ってみましょう

超マイナーな言語とかでなければレビューします。これに返信する形でも構いませんし, 個チャや質問boxに送っていただいても構いません。

### 発展

Rustでは`Iterator`トレイトを実装することで本物のイテレータを自分で作ることができます。

先ほど作ってみた`MyIterator`に`Iterator`トレイトを実装することで本物のイテレータとして扱うことができます。以下のように`impl Iterator for MyIterator`を実装することで可能です。
```rust
// データとインデックスを持った構造体をイテレータとして使う
struct MyIterator<'a, T> {
    data: &'a [T],
    index: usize,
}
impl<'a, T> MyIterator<'a, T> {
    fn new(data: &'a [T]) -> Self {
        Self { data, index: 0 }
    }
}
// MyIteratorにIteratorトレイトを実装する
impl<'a, T> Iterator for MyIterator<'a, T> {
    type Item = &'a T;
    // nextメソッドを定義する
    fn next(&mut self) -> Option<Self::Item> {
        // indexがデータの長さ以上になったらNoneを返す
        if self.index >= self.data.len() {
            None
        } else {
            // indexの場所の値を返す
            let value = &self.data[self.index];
            // indexを一つ進める
            self.index += 1;
            // Someで包んで返す
            Some(value)
        }
    }
}
```
こうすることで
```rust
fn main() {
    let data = [1, 2, 3, 4, 5];
    let iter = MyIterator::new(&data);
    iter.for_each(|x| println!("{}", x));
}
```
のようにして`for_each`や`map`のようなイテレータメソッドを呼び出すことができますし,Rustではfor文は内部的にはイテレータが使われているのでfor文に渡すこともできます。
```rust
fn main() {
    let data = [1, 2, 3, 4, 5];
    for x in MyIterator::new(&data) {
        println!("{}", x);
    }
}
```
のようにしてfor文を書くことができます。

上ではポインタを使わずに配列とインデックスでイテレータを作りましたが, 本格的なイテレータはポインタを使ったもので以下のような感じです。
```rust
use std::marker::PhantomData;

struct MyIterator<'a, T> {
    // *const T は「不変の生ポインタ」。指し示す先の値を変更できない。
    ptr: *const T,
    // イテレーションの終了地点を指すポインタ
    end: *const T,
    // PhantomData は「実際には持っていないが、論理的に持っている」ことをコンパイラに伝える。
    // ここでは「ライフタイム 'a の参照（&'a T）を持っている」と伝え、安全性を確保している。
    _marker: PhantomData<&'a T>,
}

impl<'a, T> MyIterator<'a, T> {
    fn new(slice: &'a [T]) -> Self {
        let ptr = slice.as_ptr(); // 配列の先頭ポインタを取得
        let len = slice.len();
        // 終了地点（先頭ポインタ + 長さ）を計算。unsafeが必要。
        let end = unsafe { ptr.add(len) };
        Self {
            ptr,
            end,
            _marker: PhantomData,
        }
    }
}

// Iterator トレイトの実装
impl<'a, T> Iterator for MyIterator<'a, T> {
    // このイテレータが返す型を指定
    type Item = &'a T;

    fn next(&mut self) -> Option<Self::Item> {
        // 現在地が終了地点に達したら終了
        if self.ptr == self.end {
            return None;
        }
        // 現在のポインタが指す値を参照（&）として取り出す
        let value = unsafe { &*self.ptr };
        // ポインタを次に進める
        self.ptr = unsafe { self.ptr.add(1) };
        // 取り出した値を Some で包んで返す
        Some(value)
    }
}
```
さらに, `DoubleEndedIterator`トレイトを実装することで、逆方向にもイテレーションを行うことができます。
```rust
impl<'a, T> DoubleEndedIterator for MyIterator<'a, T> {
    fn next_back(&mut self) -> Option<Self::Item> {
        if self.ptr == self.end {
            return None;
        }
        self.end = unsafe { self.end.sub(1) };
        let value = unsafe { &*self.end };
        Some(value)
    }
}
```

### イテレータの用途

ここまで読んでfor文でよくね?って思ったかもしれません。ではどのようなときにイテレータを使うのでしょうか?

イテレータは、特に大きなデータを扱う際に有用です。例えば、大きなファイルを一行ずつ読み込む場合、一度に全データをメモリに読み込むのではなく、一行ずつ読み込むことができます。また、イテレータは、データの構造を隠蔽することができます。例えば、データの構造を変更する必要がある場合、イテレータを介してアクセスするので、データの構造を変更する必要はありません。

さらに手軽にコレクションを扱うことができます。例えば、配列を逆順に並び替えてから、要素をすべて2乗してn項目までの和の配列を作る際、for文では以下のように書く必要があります。
```rust
fn main() {
    let data = [1, 2, 3, 4, 5];
    let mut reverse = [0; 5];
    for i in 0..reverse.len() {
        reverse[i] = data[data.len() - 1 - i];
    }
    let mut result = [0; 5];
    for i in 0..reverse.len() {
        if i == 0 {
            result[i] = reverse[i] * reverse[i];
        } else {
            result[i] = result[i - 1] + reverse[i] * reverse[i];
        }
    }
    println!("{:?}", result);
}
```
これをイテレータを使って書くと
```rust
fn main() {
    let data = [1, 2, 3, 4, 5];
    let result = data
        .iter()
        .rev()
        .map(|x| x * x)
        .scan(0, |acc, x| {
            *acc += x;
            Some(*acc)
        })
        .collect::<Vec<_>>();
    println!("{:?}", result);
}
```
このように1文で書くことができます。1文になったところでその1文が長くなってるんだからそこまで変わらないじゃん!と思うかもしれませんが、ここで重要なのは記述量が少ないことではありません。`.rev()`(reverseの意味)や`.map(|x| x * x)`(mapは写像の意味)のように宣言的に記述をすることができ、
```rust
for i in 0..reverse.len() {
    reverse[i] = data[data.len() - 1 - i];
}
```
のように何をするのかを書くのではなく、どんな結果が欲しいのかを書くことができ、この抽象化によってコードの意味が分かりやすくなりますし、細かい処理を隠蔽することができ、コードの可読性が向上します。(関数型プログラミングの考え方)


