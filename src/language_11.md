# 今日の豆知識.11

インタプリタとコンパイラについてです

簡単に言うと
インタプリタ = コードの解釈をしてその都度実行する
コンパイラ = コードを他の言語(多くの場合翻訳先は機械語で実行可能ファイルを生成する)

Pythonの公式実装であるCPythonなんかは典型的なインタプリタ言語で,pythonというインタプリタが.pyファイルを一文ずつ解釈&実行をしていてCPython自体はC言語で作られている。

一方典型的なコンパイラ言語としてはC言語などが挙げられ,gccやClangといったコンパイラによって.cファイルから実行可能ファイル(Windowsでは.exeファイルなど)を生成します。

有名な言語のインタプリタやコンパイラのソースコードは巨大で解説するのが大変なのでWapLのインタプリタとコンパイラのソースコードの一部を見ながら説明していきます。

以下のように`start`関数を呼んで`println`で`Hello, world!`を表示するプログラムを考えます。
```wapl
fn start(){;
    println("Hello, world!");
};
start();
```
**WapLインタプリタ(C#製)**

まずWapLのコードを文字列として受け取ってすべて`;`で分割&トリムする
この段階で
```
["fn start(){" , "println("Hello, world!")" , "}" , "start()"]
```
のような文字列の配列になっている

次に`"fn "`というキーワードをもとに関数の定義が書かれているかを探す
`fn start(){`を見つける
```cs
class Function
{
    public List<(string Type, string Name)> Parameters = new List<(string, string)>(); // 引数
    public List<string> Body = new List<string>(); // 関数の本体
}
```
のような型で関数を表す型を作って`Dictionary<string, Function> functions = new Dictionary<string, Function>();`のように関数名をカギにして登録されている`Function`を読み取ることができるようにしてあるので
```cs
functions["start"] = new Function{Parameters =new List<(string,string)>, Body = new List<string>(){"println(\"Hello, world!\")"}}
```
のようにして記録する。
次に
`"start()"`を読み取って先ほど記録した`start`関数の`Body`にあるコードを読み取っていって
`"println("Hello, world!")"`を読み取って
`"Hello, world!"`を評価してC#で作った型の`record StringValue(string Data) : VariableValue;`で`StringValue("Hello, world!")`を作る
C#の関数である`Console.WriteLine`を使って`Hello, world!`が出力される。
ちなみにどの行を読むかは`["fn start(){" , "println("Hello, world!")" , "}" , "start()"]`の配列をfor文で`i`をインクリメントしながら回してもし`if`や`while`みたいなものが来たらその`i`を直接書き換えることで読む場所を指定しています。
また,メモリは`Memory vmemory = new Memory { memory = new VariableValue[10000], emptyArea = new List<EmptyArea>() };`のような配列で再現しています。

これだけ読んで"そういうことか!"ってはならないと思いますが, インタプリタの実装の特徴を正確さを欠きますが端的にまとめると**コンピュータの再現**といえるでしょう
コンピュータが機械語を実行しているのと同じようにソースコードを既存の言語で作った疑似的なコンピュータで逐次実行しているようなものです。

**WapLコンパイラ(Rust製)**
コンパイラではまず先ほどのコードを単語レベルに分解して配列にします(これをトークン列という) これはさっき`;`で区切ったのと同じようなことでこっちの方が後で作ったので実装がより本格的になっているだけでインタプリタとコンパイラの違いとは関係ありません
```rust
0:Fn
1:Ident("start")
2:Lsep(LParen)
3:Rsep(RParen)
4:Lsep(LBrace)
5:Semicolon
6:Ident("println")
7:Lsep(LParen)
8:StringLiteral("Hello, world!")
9:Rsep(RParen)
10:Semicolon
11:Rsep(RBrace)
12:Semicolon
13:Ident("start")
14:Lsep(LParen)
15:Rsep(RParen)
16:Semicolon
```
こんな感じになっています。
これに構造を持たせて
```rust
0:Function(Function {
    name: "start",
    return_type: Ident("void"),
    args: [],
    body: [
      Stmt {
        expr: Call {
            name: "println",
            args: [String("Hello, world!")]
        }
      }
    ]
})
1:Function(Function {
    name: "toplevel_child.0",
    return_type: Ident("void"),
    args: [],
    body: [
      Stmt {
        expr: Call {
            name: "start",
            args: []
        }
      }
    ]
})
```
のように抽象構文木(abstract syntax tree以降AST)というものを作る(この段階もインタプリタとコンパイラの違いの本質ではなく私がコンパイラをあとに作ったため実装が本格的になっているだけです。)

このASTをもとに
```llvm
; ModuleID = 'wapl_module'
source_filename = "wapl_module"
target datalayout = "e-m:e-p270:32:32-p271:32:32-p272:64:64-i64:64-i128:128-f80:128-n8:16:32:64-S128"
target triple = "x86_64-unknown-linux-gnu"

@str_0 = private unnamed_addr constant [14 x i8] c"Hello, world!\00", align 1
@println_fmt_1 = private unnamed_addr constant [4 x i8] c"%s\0A\00", align 1

declare i64 @strtol(ptr, ptr, i32)

declare double @atof(ptr)

declare i32 @printf(ptr, ...)

declare i32 @sprintf(ptr, ptr, ...)

declare ptr @realloc(ptr, i64)

declare ptr @malloc(i64)

declare void @free(ptr)

declare i32 @scanf(ptr, ...)

define void @start() {
entry:
  %printf = call i32 (ptr, ...) @printf(ptr @println_fmt_1, ptr @str_0)
  ret void
}

define void @toplevel_child.0() {
entry:
  call void @start()
  ret void
}

define i32 @main(i32 %argc, ptr %argv) {
entry:
  call void @toplevel_child.0()
  ret i32 0
}
```
のような中間言語を生成する。
またさらにこの中間言語に対しても同じようにして今度は機械語(実行可能ファイル)を作る

このようにコンパイラの本質は
言語から言語(機械語)への翻訳です。

長くなったのでもう一度まとめると

**インタプリタ**:
- 解釈しながら実行(=高級なコンピュータをソフトウェアで再現/仮想マシン(Virtual Machine)を実装した実行エンジン)
- CPU : 機械語 = インタプリタ : 高級言語
- コンピュータを直接的に動かしているわけではない(間にソフトウェアをはさむ)ため実行速度が遅くなりがち
- 仮想マシンがOS差を吸収するので実行環境さえ用意していればたいていどのOSでも動くソースコードを配布することができる
- 実行してみないとエラーが出ない

**コンパイラ**:
- 事前に別の言語へ翻訳(多くの場合機械語へ)
- 機械語を生成するものは実行速度は速い傾向にある
- その言語のための実行環境を用意する必要がない
- 実行前にエラーを出せる

有名な言語を分類していくと

純粋なインタプリタ
- Python(CPython)
- JS(昔の)
- Ruby(昔の)

純粋なコンパイラ
- C/C++
- Rust
- Zig
- Go

ハイブリッド(バイトコードや中間言語へのコンパイル + 仮想マシン  (+  JITコンパイラ))
- JS(現代の)
- C#
- Java
- Kotlin

トランスパイル(高級言語->高級言語の翻訳をするコンパイラ)
- TS

一応**JIT(Just In Time)コンパイラ**と**AOT(Ahead Of Time)コンパイラ**の違いも説明しておきます

AOTは実行前にコンパイルして実行可能ファイルを作るもの

JITは実行時に機械語に翻訳して実行するもの

C#やJavaやKotlinはAOT + JITになっている

これらの言語ではAOTのときにバイトコードを生成してそれを実行ファイル内に格納して,実行時にバイトコードからネイティブコード(機械語)を生成して実行する

**AOT + JIT の利点**

- 起動前にエラーを潰せる
- 実行時情報を使って最適化できる
- CPU差分をJITに任せられる

ちなみに現代のJSでは
ソースコードをASTやバイトコードにするのも実行時に行っています


ちなみにインタプリタだとPythonみたいに対話型で実行することができたりします。
WapLインタプリタで対話モードで実行している様子
```
PS C:\Users\hayate\wapl_projects> wli
対話形式で開始します:exit();すると終了します

>>println("Hello, world!");
Hello, world!
>>fn add(i32 a,i32 b):i32{;return +(a,b);};
>>println(add(3,2));
5
>>fn mul(i32 a,i32 b):i32{;return *(a,b);};
>>println(add(5,mul(6,7)));
47
>>=(x,10,i32);
>>println(x);
10
>>exit();
```