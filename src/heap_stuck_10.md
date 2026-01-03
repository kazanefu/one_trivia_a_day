# 今日の豆知識.10

今日はメモリのヒープ領域とスタック領域についてです。

まず昨日説明しようと思っていて忘れた配列とポインタについて説明します。<-これあったほうがヒープ領域とスタック領域の利用例がわかりやすい

C言語で,以下の`a`という配列と`p`というポインタのしていることは全く同じです。
```c
#include<stdio.h>

int main(){
    int a[5] = {1,2,3,4,5};
    int* p = a; // 配列は(ここはa)は先頭のポインタなのでint* pに代入できる

    printf("a3: %d\n", a[3]); // a3: 4
    printf("p3: %d\n", p[3]); // p3: 4

    printf("a1: %d\n", *(a + 1)); // a1: 2
    printf("p1: %d\n", *(p + 1)); // p1: 2

    void* none_type_ptr = (void*)p; // pはint*だが,void*という型無ポインタにキャスト
    
    // intは32bit整数なので 32bit = 4byte = sizeof(int) より以下のようになる
    printf("p2 = none_type_ptr(2 * 4): %d\n", *(int*)(none_type_ptr + 2 * 4)); // p2 = none_type_ptr(2 * 4): 3

    return 0;
}
```
このように実は配列って連続したメモリ領域上に値を置いて,その先頭のポインタなんです。
そして,`a[n]`は`*(a + n)`や`*(int*)((void*)a + n * sizeof(int))`と等価であることがわかると思います。

> 正確にはClangでコンパイルした時,`a`の中間言語での型は`[5 x i32]`であるのに対して`p`の型は`ptr`だが`a`が読み取られるときは毎回先頭のポインタである`ptr`型に変換されている。
> ちなみに,Rustなどでは`let a = [1,2,3,4,5];`のようにしたとき中間言語での型は`[5 x i32]`のまま(正確には最適化とかのために`[ 20 x i8`)扱われていた配列とポインタは明確に異なる。(ただし,Vec&lt;T&gt;などの可変長配列は内部でポインタを使っている)
> さらにちなむと, WapLではRust的なポインタとは異なる配列とポインタによる配列の両方が存在する。

では今回の本題のヒープ領域とスタック領域についてです。
メモリは以下のような領域に分けられています。

----------------------------------------

テキスト領域

----------------------------------------

静的領域

----------------------------------------

ヒープ領域<br>
↓

----------------------------------------
↑<br>
スタック領域

----------------------------------------



今日はヒープ領域とスタック領域のみについて説明します。

ヒープ領域とスタック領域はそれぞれ境界に向かってデータが記憶されていき, その境界以上にデータを記憶しようとするとオーバーフローになります。そのため使わなくなったデータのあるところは解放して,再び他のデータを入れられるようにしていく必要があります。

ヒープ領域ではメモリの確保と解放をC言語ではプログラマが直接記述します
> (他の言語ではコンパイラがそのデータを使っているかを検出するアルゴリズムを使って自動で解放していたり,Rustみたいに所有権の仕組みで所有者がスコープから外れるタイミングで自動で解放しているが、これらも中間言語の段階だったりではメモリの確保と解放が記述されている)

一方, スタック領域は関数に入ったときにメモリが確保されて,関数から抜けるときに自動で解放されます。

## スタック領域について

まず,一般にスタックというデータ構造について説明すると<br>
|-|-|-|-|-|-|<br>
のような6個のものを入れることのできる構造があったとしてここに新たに`5`をいれると<br>
|5|-|-|-|-|-|<br>
のようになり,次に`3`を入れると<br>
|5|3|-|-|-|-|<br>
続けて`7`,`6`,`0`を入れると<br>
|5|3|7|6|0|-|<br>
のようにデータが積み重ねられていきます(スタック構造ではこの新たにデータを入れる操作をpushと呼ぶ)<br>

そこから取り出す(スタック構造ではデータを取り出す操作をpopと呼ぶ)と<br>
|5|3|7|6|-|-| => 0<br>
|5|3|7|-|-|-| => 6<br>
|5|3|-|-|-|-| => 7<br>
のようにpopするとあとからpushしたものから取り出されていきます。<br>
ここからさらに`1`をpushすると<br>
|5|3|1|-|-|-|<br>
のようになります。<br>

メモリのスタック領域ではメモリがスタック構造になっていてメモリの確保がpush、メモリの解放がpopに相当します。
そして関数に入ったときにスタック領域にデータが保存される変数が作られるたびに**push**つまり**メモリ確保**されて,関数が終わるときにその関数のときにpushされた分をすべて**pop**つまり**メモリ解放**します。
例えば以下のようなRustのコードを考えてみましょう
```rust
fn func_c(y_from_a: i32){
    let beta = 53;
    println!("{}",y_from_a + beta);
}

fn func_b(x_from_main: i32, y_from_a: i32) {
    let z = 99;
    let alpha = 12;
    println!("{}", x_from_main + y_from_a + z + alpha);
}

fn func_a(x_from_main: i32) {
    let y = 10;
    func_b(x_from_main, y);
    func_c(y);
}

fn main() {
    let x = 42;
    func_a(x);
    // プロセス終了
}
```
|-|-|-|-|-|-|をスタック領域に見立てたとき<br>
`let x = 42;`でまず`42`がpushされます<br>
|42|-|-|-|-|-|<br>
次に`func_a`が呼ばれてその中で`let y = 10;`で`10`がpushされます<br>
|42|10|-|-|-|-|<br>
次に`func_b`が呼ばれてその中で `let z = 99; let alpha = 12;`により`99`と`12`がpushされます<br>
|42|10|99|12|-|-|<br>
`println!("{}", x_from_main + y_from_a + z + alpha);`で和を表示した後何もしていないのでここで`func_b`は終了です。<br>
すると`func_b`で確保された分がpopされて<br>
|42|10|-|-|-|-|<br>
となります。<br>
次に`func_a`で`func_c`が呼ばれて,その中で`let beta = 53;`で`53`がpushされます<br>
|42|10|53|-|-|-|<br>
また`func_c`が終わった後にその中で確保されていた`53`がpopされます<br>
|42|10|-|-|-|-|<br>
これで`func_a`も終わったので`10`もpopされます<br>
|42|-|-|-|-|-|<br>
また,これで`main`も終わったので`42`もpopされます<br>
|-|-|-|-|-|-|<br>

スタック領域の状態をまとめると<br>
|-|-|-|-|-|-|<br>
|42|-|-|-|-|-|<br>
|42|10|-|-|-|-|<br>
|42|10|99|12|-|-|<br>
|42|10|-|-|-|-|<br>
|42|10|53|-|-|-|<br>
|42|10|-|-|-|-|<br>
|42|-|-|-|-|-|<br>
|-|-|-|-|-|-|<br>
のようになります。<br>
|-|-|-|-|-|-|を例えにすると6個より多くのデータがpushされた時にヒープ領域との間の境界を越えてしまい**スタックオーバーフロー**となります。<br>
そのため回数の多いループ処理を再帰関数(関数内で自身を呼ぶ関数)で実装すると**スタックオーバーフロー**になります。<br>

## ヒープ領域について

スタック領域についての説明では`func_b`の中で確保されている`z`や`alpha`は`func_c`が呼ばれるときにはすでに解放されてしまっているので`func_c`の中では使うことができません。これらを`func_a`の中で宣言したり,`func_b`で戻り値として渡すことで`func_a`の中でも確保するようにすれば解決できますが,前者の解決方法では`func_a`の中では使わない変数が増えて可読性が下がったり関数の意味が明確に表現されていない状態になってしまいますし,後者の解決方法ではデータのアドレスは`func_a`のなかで再確保していることで変わってしまっているので参照(ポインタ)を使った操作をしたいときに破綻します。

このような問題を解決できるのがヒープ領域です。
ヒープ領域では手動でメモリ確保とメモリ解放を行うため,`func_b`を抜けても`z`や`alpha`を解放しないようにすることができます。

この後ヒープの説明はメモリ確保/解放を明示的にプログラマが書くC言語を使って説明するので先ほどのスタック領域の説明で使ったRustのコードをCに翻訳したものを一応貼っておきます。
```c
#include <stdio.h>

void func_c(int y_from_a) {
    int beta = 53;
    printf("%d\n", y_from_a + beta);
}

void func_b(int x_from_main, int y_from_a) {
    int z = 99;
    int alpha = 12;
    printf("%d\n", x_from_main + y_from_a + z + alpha);
}

void func_a(int x_from_main) {
    int y = 10;
    func_b(x_from_main, y);
    func_c(y);
}

int main(void) {
    int x = 42;
    func_a(x);
    return 0; // プロセス終了
}
```
では`func_b`でヒープ領域にメモリを確保してそれを`func_c`で使うようにした例を見てみましょう
```c
#include <stdio.h>
#include <stdlib.h>

void func_c(int* shared_value) {
    int beta = 53;
    printf("%d\n", *shared_value + beta);
}

int* func_b(int x_from_main, int y_from_a) {
    int z = 99;
    int alpha = 12;

    // ヒープに確保
    int* heap_value = (int*)malloc(sizeof(int)); // mallocは引数に確保する領域のサイズを渡して戻り値としてvoid*型(型なしのポインタ)で確保した領域の先頭のポインタを返す

    *heap_value = x_from_main + y_from_a + z + alpha; // ヒープ領域に確保したところに値を記憶する
    return heap_value; // ポインタを返す
}

void func_a(int x_from_main) {
    int y = 10;

    int* value_from_b = func_b(x_from_main, y);

    func_c(value_from_b);

    // ヒープ解放
    free(value_from_b); // freeはmallocでヒープ領域に確保したものをその先頭のポインタを渡すことで解放する。
}

int main(void) {
    int x = 42;
    func_a(x);
    return 0;
}
```
このようにすることで`func_b`で作った値をアドレスを移動させることなく`func_c`でも使うことができている。

さらに,ヒープ領域はスタック領域に比べて容量も大きいです。

この説明だけだと"すべてヒープ領域を使ってしまった方が自由な操作ができていいじゃないか"。と思うかもしれませんが,ヒープ領域にも欠点があります。

**ヒープ領域の欠点**
- スタック領域を使ったコードと比較して実行速度が遅い。
- ヒープ上に確保した領域のポインタはスタック領域に保持されるのでそのポインタがpopされる前に`free`で解放するのを忘れるとその領域を一生解放できなくなり(これを**メモリリーク**という)使わないデータが蓄積していってしまい,長い時間動かしているとオーバーフローしてしまう。
- たとえちゃんと解放できていたとしてもメモリが断片化することでオーバフローしてしまうことがある。
- 解放済みの領域のポインタなど無効な領域を解放しようとして壊れることがある(**ダングリング**という)

メモリの断片化を説明すると<br>
|-|-|-|-|-|-|をヒープ領域に見立てたときに<br>
42を入れる<br>
|42|-|-|-|-|-|<br>
[ 53, 22]という配列を入れる<br>
|42|53|22|-|-|-|<br>
13を入れる<br>
|42|53|22|13|-|-|<br>
[ 53, 22]のために確保した分を解放する<br>
|42|-|-|13|-|-|<br>
ここで,[ 99, 81, 44]のようなサイズが3の配列のためのメモリを確保しようとしても3つ連続した領域は存在しないので空いている場所は3つ以上存在しているにも関わらず**確保に失敗**します。<br>
スタック領域ではこのようなことは起こりません。<br>

ちなみに**メモリリーク**や**ダングリング**はC以外の言語ではガベージコレクション(以降GC)やRustの所有権/借用/ライフタイムやC++のRAIIなどによって`free`をプログラマが直接書かないで自動で解放されるようにすることで防いでいます。

ちなみにGCにもデメリットがあって, GCはそのメモリ領域の参照が使われているかを動的に(実行時に)判断しているため,実行速度がGCなしの言語と比べて遅い傾向にあります。

Rustの所有権/借用/ライフタイムが画期的と言われる理由はメモリの解放処理はコンパイル時に静的に挿入されるので実行速度を落とさずに安全を実現できるという点ですが,これも欠点としてコンパイルにかかる時間が長くなったり,新しい仕組みなので他の言語を使ってた人は慣れるまでに時間がかかることなどが挙げられます。

C++のRAIIいつ破棄(解放)されるかを保証するだけで完全に安全と言えるものではありません

先ほどヒープ領域の説明で使ったCのコードを他の言語で書くとどうなるかも一応書いておきます

**C#**(GCによって管理)
```cs
using System;

class HeapInt
{
    public int Value;
}

class Program
{
    static void FuncC(HeapInt sharedValue)
    {
        int beta = 53;
        Console.WriteLine(sharedValue.Value + beta);
    }

    static HeapInt FuncB(int xFromMain, int yFromA)
    {
        int z = 99;
        int alpha = 12;

        // ヒープに確保（GC管理）
        return new HeapInt
        {
            Value = xFromMain + yFromA + z + alpha
        };
    }

    static void FuncA(int xFromMain)
    {
        int y = 10;

        HeapInt valueFromB = FuncB(xFromMain, y);

        FuncC(valueFromB);

        // free は書かない
        // GCが必要になったら回収する
    }

    static void Main()
    {
        int x = 42;
        FuncA(x);
    }
}

```
C#では**new**によってクラスのインスタンスを生成した時にヒープ上に確保されます。(クラスやインスタンスについてはオブジェクト指向のときに解説すると思う)

**Go**(GCによって管理)
```go
package main

import "fmt"

type HeapInt struct {
    Value int
}

func funcC(sharedValue *HeapInt) {
    beta := 53
    fmt.Println(sharedValue.Value + beta)
}

func funcB(xFromMain int, yFromA int) *HeapInt {
    z := 99
    alpha := 12

    // ヒープに逃がす（エスケープ）
    return &HeapInt{
        Value: xFromMain + yFromA + z + alpha,
    }
}

func funcA(xFromMain int) {
    y := 10

    valueFromB := funcB(xFromMain, y)

    funcC(valueFromB)

    // free は書かない（GC 管理）
}

func main() {
    x := 42
    funcA(x)
}
```
Goでは構文によってヒープ上に確保するかスタックに確保するかは決まらず,コンパイラが エスケープ解析で決定しています。

**Rust**(所有権/借用によって静的にチェック)
```rust
fn func_c(shared_value: &i32) {
    let beta = 53;
    println!("{}", *shared_value + beta);
}

fn func_b(x_from_main: i32, y_from_a: i32) -> Box<i32> {
    let z = 99;
    let alpha = 12;

    // ヒープに確保
    let heap_value = Box::new(x_from_main + y_from_a + z + alpha);
    heap_value
}

fn func_a(x_from_main: i32) {
    let y = 10;

    let value_from_b = func_b(x_from_main, y);

    func_c(&value_from_b); // BoxはスマートポインタでDerefトレイトがついているため&Box<i32>は&i32

    // freeは書かない（スコープ終了時に自動解放）
}

fn main() {
    let x = 42;
    func_a(x);
}
```

このコードのRustでは`func_b`での`heap_value`は`func_a`の`value_from_b`に所有権を譲渡して,その後`func_c`の引数に借用を渡して,`func_a`が終わるときに所有者である`value_from_b`がいるので自動で解放処理が挿入される。

ちなみにRustではCみたいに書くこともできはします。
```rust
use std::alloc::{alloc, dealloc, Layout};

unsafe fn func_c(shared_value: *mut i32) {
    let beta = 53;
    println!("{}", *shared_value + beta);
}

unsafe fn func_b(x_from_main: i32, y_from_a: i32) -> *mut i32 {
    let z = 99;
    let alpha = 12;

    let layout = Layout::new::<i32>();
    let ptr = alloc(layout) as *mut i32; // Cのmallocに相当

    *ptr = x_from_main + y_from_a + z + alpha;
    ptr
}

unsafe fn func_a(x_from_main: i32) {
    let y = 10;

    let value_from_b = func_b(x_from_main, y);
    if value_from_b.is_null() {
        return;
    }

    func_c(value_from_b);

    let layout = Layout::new::<i32>();
    dealloc(value_from_b as *mut u8, layout); // Cのfreeに相当
}

fn main() {
    unsafe {
        let x = 42;
        func_a(x);
    }
}
```
さらにRcやArcのような参照カウンタ方式での解放ができたりもする(ここでは長くなるので解説しない)

**C++**(RAII)
```cpp
#include <iostream>
#include <memory>

void func_c(const int* shared_value) {
    int beta = 53;
    std::cout << (*shared_value + beta) << std::endl;
}

std::unique_ptr<int> func_b(int x_from_main, int y_from_a) {
    int z = 99;
    int alpha = 12;

    // ヒープ確保（RAII）
    return std::make_unique<int>(x_from_main + y_from_a + z + alpha);
}

void func_a(int x_from_main) {
    int y = 10;

    std::unique_ptr<int> value_from_b = func_b(x_from_main, y);

    func_c(value_from_b.get());

    // スコープ終了時に自動で delete (解放)
}

int main() {
    int x = 42;
    func_a(x);
}
```
RustがBoxというスマートポインタを使ったようにunique_ptrというスマートポインタを使っている

**ヒープ領域に値が保持されている型**
RustのVec&lt;T&gt;/StringやC++のvector&lt;T&gt;/stringでは
内部構造が
```rust
struct Vec<T> {
    ptr: *mut T,
    len: usize,
    cap: usize,
}
```
のようになっていて
`ptr`がCのポインタによる配列であり,配列の値はヒープ上に確保されておりこのポインタはその配列の先頭のポインタである。
`len`は要素の数
`cap`は確保済みのメモリ領域のサイズ(byte)を要素の型のサイズで割ったもの
が入っていて`String`は`Vec<char>`と同等です。
C言語で`Vec<i32>`を再現しようとすると
```c
#include<stdlib.h>

typedef struct c_vec{
    int* ptr;
    long len;
    long cap;
}c_vec;

c_vec c_vec_new(long cap) {
    c_vec v;
    v.len = 0;
    v.cap = cap;

    if (cap > 0) {
        v.ptr = (int*)malloc(sizeof(int) * cap); // メモリをヒープ上に確保
        if (!v.ptr) {
            perror("malloc failed");
            exit(1);
        }
    } else {
        v.ptr = NULL;
    }

    return v;
}

void c_vec_grow(c_vec* v) {
    long new_cap = (v->cap == 0) ? 1 : v->cap * 2;

    int* new_ptr = (int*)realloc(v->ptr, sizeof(int) * new_cap);
    if (!new_ptr) {
        perror("realloc failed");
        exit(1);
    }

    v->ptr = new_ptr;
    v->cap = new_cap;
}

void c_vec_push(c_vec* v, int value) {
    if (v->len == v->cap) {
        c_vec_grow(v);
    }

    v->ptr[v->len] = value;
    v->len++;
}

int c_vec_get(c_vec* v, long index) {
    if (index < 0 || index >= v->len) {
        fprintf(stderr, "index out of bounds\n");
        exit(1);
    }
    return v->ptr[index];
}

void c_vec_free(c_vec* v) {
    free(v->ptr);
    v->ptr = NULL;
    v->len = 0;
    v->cap = 0;
}
```
のようにして再現することができます。