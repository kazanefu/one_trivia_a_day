# 今日の豆知識.7

ネタのストック0で書き始めてすでにネタ考えるのがめんどくさくなってきてしまったのでこれからしばらく言語の紹介をしていきたいと思います。

とりあえず今日は**WapL**についてです

## 歴史?

WapLはC#で書いたインタプリタから開発がスタート<br>
↓<br>
Unityのゲーム内で動かせるインタプリタが作られた<br>
↓<br>
C#で書いたインタプリタにUnity版の処理方式を逆輸入&コマンドラインでPythonみたいに対話形式で実行できるものも作られた<br>
↓<br>
Rustで書かれたコンパイラが作られた. 文法はインタプリタ版とコンパイラ版で異なるところが結構多い<br>

## 特徴

インタプリタ/コンパイラ共通の特徴
- ほとんどすべての記述が関数呼び出し形式(ポーランド記法)
- ポインタを直接扱うことが多く,低レベル的な記述ができる
- `warpto`/`warptoif`/`point` によるgoto文みたいなものが基本的な制御機構(コンパイラ版では途中からloopifやifが実装されたのでそうじゃない感じが少しあるけど...)
- 型明示的
- 規模が小さいので覚えることが少ない

インタプリタ特有
- Unityで使える
- 内部的にはすべての型が同じオブジェクトとして扱われている(Python的)
- インタプリタさえインストールすればPythonみたいに簡単に使える
- 動的型付け

コンパイラ特有
- 低レベル処理ができたりGCなどのリッチなランタイムを持たなく,LLVMバックエンドであるためかなり高速(C/C++ ,Rustなどと同等かそれ以上)
- C/C++や他のLLVMバッグエンドの言語のライブラリを使ったりリンクをさせることができる
- waplupやwapl-cliのような便利なツールを使うことでバージョン管理やビルドなどが簡単にできる
- WebAssembly対応なのでOSやハードウェアによらないものを作ることができ,Web開発にも使える
- 標準ライブラリがある
- 複数ファイルでの開発ができる
- 静的型付け
- 簡易的な所有権の仕組みを取り入れていてコンパイル時にある程度の安全性を担保

## Hello, world!

```wapl
println("Hello ,world!");
```

## コードの例
**フィボナッチ数列を求める関数**

インタプリタ版
```wapl
fn fibo(i32 n){;
    =(a,0,i32);
    =(b,1,i32);
    =(i,0,i32);
    point start;
    warptoif(!(<(i,n)),break);
        =(tmp, b,i32);
        =(b,+(a,b));
        =(a,tmp);
        +=(i,1);
        warpto(start);
    point break;
    return a;
};
println(fibo(10));
```
なるべくインタプリタ版の似た記法で書いたコンパイラ版
```wapl
fn fibo(i32 n):i32{;
    #=(a,0s,i32);
    #=(b,1s,i32);
    #=(i,0s,i32);
    warpto(start)
    point start;
    warptoif(<(i,n),inloop,break);
    point inloop
        #=(tmp, b,i32);
        =(b,+(a,b));
        =(a,tmp);
        =(i,+(i,1s));
        warpto(start);
    point break;
    return a;
};
println(format("%d",fibo(10s)));
```
コンパイラ版にしかない文法も使ったもの
```wapl
fn fibo(i32 n):i32{
    #=(a,0s,i32);
    #=(b,1s,i32);
    #=(i,0s,i32);
    loopif:(<(i,n)){
        #=(tmp, b,i32);
        =(b,+(a,b));
        =(a,tmp);
        =(i,+(i,1s));
    }
    return a;
}
println(format("%d",fibo(10s)));
```

参考文献:

- [https://github.com/kazanefu/WapL_for_Unity_InGame/blob/main/HowToUse.md](https://github.com/kazanefu/WapL_for_Unity_InGame/blob/main/HowToUse.md)  <-インタプリタ版(Unity版)の説明書
- [https://github.com/kazanefu/WapL_interpreter/blob/main/HowToUse.md](https://github.com/kazanefu/WapL_interpreter/blob/main/HowToUse.md) <-インタプリタ版(コマンドライン版)の説明書
- [https://kazanefu.github.io/WapL_Book/](https://kazanefu.github.io/WapL_Book/) <-コンパイラ版の説明書
- [https://github.com/kazanefu/WapL_for_Unity_InGame/tree/main](https://github.com/kazanefu/WapL_for_Unity_InGame/tree/main) <-インタプリタ(Unity版)のソースコードというかUnityのサンプルプロジェクト
- [https://github.com/kazanefu/WapL_interpreter/tree/main](https://github.com/kazanefu/WapL_interpreter/tree/main) <-インタプリタ(コマンドライン版)のソースコード
- [https://github.com/kazanefu/WapL_Compiler/tree/codefixed](https://github.com/kazanefu/WapL_Compiler/tree/codefixed) <-コンパイラのソースコード