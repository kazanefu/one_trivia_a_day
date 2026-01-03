# 今日の豆知識.1
CやC#の従来型forと、PythonやRustのforは意味が違う

- Cのforは「どう回すか」を書く
- Rust/Pythonのforは「何を回すか」を書く

C系の
```c
for(int i = 0; i < n; i++)
{
  body(i);
}
```
は意味的には以下と同等
```c
int i = 0;
while(i < n)
{
  body(i);
  i++;
}
```
一方、PythonやRustの
```rust
for i in 0..n {
  body(i);
}
```
は
```rust
let mut it = IntoIterator::into_iter(0..n);
loop {
    match it.next() {
        Some(x) => body(x),
        None => break,
    }
}
```
と同等で、イテレータを順に消費しています。

ちなみに C++ には
```cpp
for (auto x : v)
{
  body(x);
}
```
という range-based for があり、これは Python や Rust の for と同じ意味を持ちます。

また C# には
```cs
foreach (var x in v)
{
  body(x);
}
```
という書き方があり、これも IEnumerable を使ったイテレータベースの for です。