# 今日の豆知識.12

構造体とクラス

まずは**構造体**
百聞は一見に如かずということでとりあえず使っている例を見てみましょう
C++では
```cpp
#include <bits/stdc++.h> // これは一旦無視でいい

using namespace std; // これも一旦無視でいい

// Personという構造体を定義
struct Person{
    // ---データ構造---
    // ここにある要素をフィールドという
    string name;
    int age;
};

int main(){
    Person hayate = {"Hayate",19};
    cout <<format("name: {}, age: {}",hayate.name,hayate.age)<<endl; // '.'でフィールドにアクセス
    return 0;
}
```
```
name: Hayate, age: 19
```
このように`struct`というキーワードで`Person`という構造体を`{string, int}`という構造であることを定義して`Person`を型のようにして扱うことができるようになる。また,`.`を使ってフィールドにアクセスすることもできる
C++では構造体に関数を組み込むこともできて,このような関数を**メソッド**と呼ぶ
```cpp
#include <bits/stdc++.h>

using namespace std;

struct Person{
    // ---データ構造---
    string name;
    int age;

    // ---関数(メソッド)---
    string say_hello(){
        string hello = format("Hello!, I'm {}", this->name); // thisは自身のポインタ(もしhayate.say_hello()で呼んだら this == &hayate)
        cout << hello << endl;
        return hello;
    }
};

int main(){
    Person hayate = {"Hayate",19};
    cout <<format("name: {}, age: {}",hayate.name,hayate.age)<<endl;
    hayate.say_hello(); // フィールドと同じように'.'でアクセス

    return 0;
}
```
```
name: Hayate, age: 19
Hello!, I'm Hayate
```

基本的に構造体はデータ構造で値型

構造体が意味的に**値**であるのに対してクラスは**主体**があることが重要で,値とそのふるまいがセットで,外からのアクセスは制限するようになっていることが多い。
C++では実装上はフィールドとメソッドがデフォルトで`public`か`private`かの違いでしかないが,意味的には上記のような違いがあるので一応使い分けは意識した方がいいと思う。(C#とかだとstructとclassは実装上もっと大きな違いがあったりもする)
先ほどstructでPersonと全く同じものをclassで作ると
```cpp
#include <bits/stdc++.h>

using namespace std;

class Person{
    // ---データ構造---
public: // デフォルトでprivateだが,publicにするとstructと実装上全く同じ
    string name;
    int age;

    // ---関数(メソッド)---
    string say_hello(){
        string hello = format("Hello!, I'm {}", this->name);
        cout << hello << endl;
        return hello;
    }
};

int main(){
    Person hayate = {"Hayate",19};
    cout <<format("name: {}, age: {}",hayate.name,hayate.age)<<endl;
    hayate.say_hello();

    return 0;
}
```
となる。

また, classらしい書き方にすると
```cpp
#include <bits/stdc++.h>

using namespace std;

class Person{
    // ---データ構造---
public: 
    string name;
    int age;

    // ---関数(メソッド)---
    string say_hello(){
        string hello = format("Hello!, I'm {}", this->name);
        cout << hello << endl;
        return hello;
    }
};

int main(){
    Person* hayate = new Person{"Hayate",19}; // newでヒープ上にPersonのデータを記録して, そのポインタで管理
    cout <<format("name: {}, age: {}",hayate->name,hayate->age)<<endl;
    hayate->say_hello();

    return 0;
}
```
のようにポインタで管理することでそのインスタンスの同一性は値の一致ではなく主体としての一致(つまりポインタの一致)で決まるようにできる。
```cpp
#include <bits/stdc++.h>

using namespace std;

class Person{
    // ---データ構造---
public: 
    string name;
    int age;

    // ---関数(メソッド)---
    string say_hello(){
        string hello = format("Hello!, I'm {}", this->name);
        cout << hello << endl;
        return hello;
    }
};

bool Person_eq(Person p1,Person p2){
    return (p1.name == p2.name && p1.age == p2.age);
}

int main(){
    Person* hayate = new Person{"Hayate",19};
    Person* hayate2 = new Person{"Hayate",19};

    Person hayate3 = Person{"Hayate",19};
    Person hayate4 = Person{"Hayate",19};

    cout<< ((hayate == hayate2)? "true":"false")<< endl;
    cout<< (Person_eq(hayate3,hayate4)? "true":"false")<< endl;

    return 0;
}
```
```
false
true
```
このようにclass的に扱う場合は`hayate == hayate2`が`false`になることを期待し,struct的に扱う場合は`hayate3`と`hayate4`が等しいように扱いたいときである。

ここで`public`や`private`について説明していなかったので説明する
publicはclass(or struct)の外からもアクセスできる
privateはclass(or struct)の外からはアクセスできず,内側からしかアクセスできないようにする
```cpp
#include <bits/stdc++.h>

using namespace std;

class Person
{
    // ---データ構造---
private:
    int id; // get_idやset_idを介さないとidに外から触れることはできない

public:
    string name;
    int age;

    // ---関数(メソッド)---
    string say_hello()
    {
        string hello = format("Hello!, I'm {}", this->name);
        cout << hello << endl;
        return hello;
    }
    Person(int id, string name, int age) : id(id), name(name), age(age) {}; // コンストラクタ
    // 外からidを取得するにはpublicにあるメソッドを使う必要がある
    int get_id()
    {
        return this->id;
    }
    // 外からidを設定するにはpublicにあるメソッドを使う必要がある
    void set_id(int new_id)
    {
        this->id = new_id;
    }
};

bool Person_eq(Person p1, Person p2)
{
    return (p1.name == p2.name && p1.age == p2.age);
}

int main()
{
    Person *hayate = new Person(1234, "Hayate", 19);

    cout << hayate->age << endl; // ageはpublicなのでどこからでもアクセスできる
    // cout<< hayate->id<<endl;  // <- これをするとエラー idはprivateフィールド
    cout << hayate->get_id() << endl;
    

    hayate->set_id(5678);
    // hayate->id = 5678; // <- これもエラー idはprivateフィールド
    cout << hayate->get_id() << endl;

    return 0;
}
```

structとclassが明確に違うC#の例も示す
**struct**
```cs
using System;
using System.Collections.Generic;

struct Person
{
    private int id { set; get; }
    public string name;
    public int age;
    public Person Clone()
    {
        return new Person(this.id,this.name,this.age);
    }

    // コンストラクタ
    public Person(int newId,string newName,int newAge)
    {
        id = newId;
        name = newName;
        age = newAge;

    }

    // idはprivateなのでgetterとsetterを設定してここからアクセスするようにする
    public int Id
    {
        get { return id; }
        set { id = value; }
    }
}
class Program
{
    // エントリーポイント
    static void Main()
    {
        Person hayate = new Person(1234,"Hayate",19);
        Person hayate2 = hayate;
        hayate2.age = 200;
        Console.WriteLine(hayate.age.ToString() + "\n" + hayate2.age.ToString());
    }
}
```
**class**
```cs
using System;
using System.Collections.Generic;

class Person
{
    private int id { set; get; }
    public string name;
    public int age;
    public Person Clone()
    {
        return new Person(this.id,this.name,this.age);
    }

    // コンストラクタ
    public Person(int newId,string newName,int newAge)
    {
        id = newId;
        name = newName;
        age = newAge;

    }

    // idはprivateなのでgetterとsetterを設定してここからアクセスするようにする
    public int Id
    {
        get { return id; }
        set { id = value; }
    }
}
class Program
{
    // エントリーポイント
    static void Main()
    {
        Person hayate = new Person(1234,"Hayate",19);
        Person hayate2 = hayate;
        hayate2.age = 200;
        Console.WriteLine(hayate.age.ToString() + "\n" + hayate2.age.ToString());
    }
}
```
コード自体は`struct`って書いたか`class`って書いたかの違いしかありません。
ではこれで実行結果にどのような違いが生じるのでしょうか
**struct**
```
19
200
```
**class**
```
200
200
```
見ての通り`hayate2.age = 200;`が`struct`では`hayate`に反映されませんが`class`では反映されます。これは`struct`は値渡しで,`class`は参照渡し(広義の)だからです。
さらに,`struct`は値がスタック上に置かれているのに対して`class`はヒープ上に置かれています。

`class`でも`struct`みたいに値渡しのようにするためにはメソッドに作った`Clone`(先ほどのコードにもすでに書いてある)のようなものを使います。
```cs
using System;
using System.Collections.Generic;

class Person
{
    private int id { set; get; }
    public string name;
    public int age;
    public Person Clone()
    {
        return new Person(this.id,this.name,this.age);
    }

    // コンストラクタ
    public Person(int newId,string newName,int newAge)
    {
        id = newId;
        name = newName;
        age = newAge;

    }

    // idはprivateなのでgetterとsetterを設定してここからアクセスするようにする
    public int Id
    {
        get { return id; }
        set { id = value; }
    }
}
class Program
{
    static void Main()
    {
        Person hayate = new Person(1234,"Hayate",19);
        Person hayate2 = hayate.Clone(); // ここをCloneにした
        hayate2.age = 200;
        Console.WriteLine(hayate.age.ToString() + "\n" + hayate2.age.ToString());
    }
}
```
```
19
200
```
逆に`struct`を`class`的に参照渡しするなら`ref`を使う
```cs
using System;
using System.Collections.Generic;

struct Person
{
    private int id { set; get; }
    public string name;
    public int age;
    public Person Clone()
    {
        return new Person(this.id,this.name,this.age);
    }

    // コンストラクタ
    public Person(int newId,string newName,int newAge)
    {
        id = newId;
        name = newName;
        age = newAge;

    }

    // idはprivateなのでgetterとsetterを設定してここからアクセスするようにする
    public int Id
    {
        get { return id; }
        set { id = value; }
    }
}
class Program
{
    static void Main()
    {
        Person hayate = new Person(1234,"Hayate",19);
        ref Person hayate2 = ref hayate; // refで渡す
        hayate2.age = 200;
        Console.WriteLine(hayate.age.ToString() + "\n" + hayate2.age.ToString());
    }
}
```
```
200
200
```