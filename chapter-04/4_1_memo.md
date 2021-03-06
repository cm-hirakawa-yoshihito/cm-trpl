# TRPL勉強メモ

## 4.1. 所有権とは？

ある値の所有権を持っている変数を所有者と呼ぶ
変数は1つの値しか持てない（束縛されない）ため、変数が所有権を持っている値はただ1つ
値の所有権は1つしかないため、常に所有者となる変数はただ1つ
→「所有者」と「値」は常に1対1の関係（それ以外は「所有者への参照」である）
所有者がスコープ（ここで言っているスコープはレキシカルスコープのこと。つまりブロック）から外れたら、値は破棄される

`&'static str`型は**プログラムにハードコードされた文字列データへの参照**という意味らしい。

データの保存領域はスタティック領域、スタック、ヒープの3個ある（これはどの言語でも同じ）。スタティック領域はプログラムの開始から終了まで生存するデータでグローバル変数および文字列が該当する。データは基本的にスタックに置かれるっぽい。ヒープに置く場合は専用の処理が必要（Javaなんかはnewしたら全部ヒープ。C#みたいなstruct/classの使い分けとかはない）。どちらも所有者がスコープから外れる際に破棄される。

スタティック領域、スタックは言語間の違いはない。ヒープに確保したデータを破棄する手法が（認識している範囲で）3個ある。

1. CやC++などのように明示的にプログラマがメモリ解放をする（`free()`や`delete`）
    - メモリリークの危険が伴う
2. JavaやPythonその他のように暗黙的にGCがメモリ解放をする
    - GCのオーバーヘッドが気になる（特にシステムプログラミングの場合）
3. Rustのように所有者がスコープから外れたらメモリ解放をする
    - メモリリークを抑えつつ、パフォーマンスも向上させる

### String型

strは上記の通りプログラムにハードコードされた文字列なのでイミュータブル。それに対してStringはヒープに確保される文字列でミュータブル（にできる）。

所有者がスコープを外れる際にRustは自動的に値を破棄するが、その際に暗黙的に呼ばれるのが`drop`。

Rustの文字列は次のバリエーションがあるように思われる。

- &'static str
- &str
- std::string::String
- &std::string::String

`'static`はスタティックライフタイムの注釈なので本質的には`&str`と同じ。`String`にはDerefというトレイトが実装されており、`&String`を`&str`に自動変換する（この辺は15章で詳しく）。

[string.rs.html -- source](https://doc.rust-lang.org/src/alloc/string.rs.html#2056)

整理すると、`&str`と`String`の使い分けということになり、これはスライスと配列の関係に似ている（ただし配列は固定長だけど）。

型を確認するコード。

```rust
let s = String.from("");
let u: () = s;
```

Stringの変数を別の変数に束縛すると、スタック上のデータをコピーする。これにはヒープメモリへのポインタも含まれる。つまり他の言語で言うところの、「参照型の変数の代入」と同じ動き。このとき所有権が新しい変数に移動する。これを**ムーブセマンティクス**と呼ぶ。Rustのデフォルトの挙動はこのムーブセマンティクスである。

i32などのプリミティブ型の場合、別の変数に束縛すると値がコピーされる。それぞれの値の所有権をそれぞれの変数が持つ。これを**コピーセマンティクス**と呼ぶ。内部的には、Copyトレイトを実装している場合にコピーセマンティクスとなり、Rustのプリミティブ型はすべてCopyトレイトを実装している。
