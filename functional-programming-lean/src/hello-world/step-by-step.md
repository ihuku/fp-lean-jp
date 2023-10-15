# ステップバイステップ（Step By Step）

`do`ブロックは1行ずつ実行できます。
まず、前のセクションのプログラムから始めましょう：
```lean
{{#include ../../../examples/hello-name/HelloName.lean:block1}}
```

## 標準入出力（Standard IO）

最初の行は `{{#include ../../../examples/hello-name/HelloName.lean:line1}}` で、残りは次のとおりです：

```lean
{{#include ../../../examples/hello-name/HelloName.lean:block2}}
```

`←` を使用する `let` 文を実行するには、矢印の右側の式（この場合 `IO.getStdIn`）を評価することから始めます。
この式は変数であるため、その値が調べられます。
その結果の値は、組み込みの基本的な `IO` アクションです。
次に、この `IO` アクションを実行し、標準入力ストリームを表す値が得られます。この値の型は `IO.FS.Stream` です。
その後、標準入力は矢印の左側の名前（ここでは `stdin`）と関連付けられ、`do`ブロックの残りの部分で使用できるようになります。

2行目、`{{#include ../../../examples/hello-name/HelloName.lean:line2}}` を実行する場合、同様に進みます。
まず、式 `IO.getStdout` を評価し、標準出力を返す `IO` アクションが生成されます。
次に、このアクションが実行され、実際に標準出力を返します。
最後に、この値は `stdout` という名前に関連付けられ、`do` ブロックの残りの部分で使用できるようになります。


## 質問する（Asking a Question）

今、`stdin` と `stdout` が見つかったので、ブロックの残りは質問と回答から成ります：

```lean
{{#include ../../../examples/hello-name/HelloName.lean:block3}}
```

ブロック内の最初の文、`{{#include ../../../examples/hello-name/HelloName.lean:line3}}` は式から成ります。
式を実行するには、まず評価されます。
この場合、`IO.FS.Stream.putStrLn` の型は `IO.FS.Stream → String → IO Unit` です。
これは、ストリームと文字列を受け入れて `IO` アクションを返す関数です。
この式は、関数呼び出しのための [アクセサー表記](../getting-to-know/structures.md#behind-the-scenes) を使用しています。
この関数には2つの引数が適用されています：標準出力ストリームと文字列です。
この式の値は、文字列と改行文字を出力ストリームに書き込む `IO` アクションです。
この値を見つけたら、次にそれを実行し、文字列と改行が実際に `stdout` に書き込まれるようにします。
式だけから成る文には新しい変数が導入されないことに注意してください。

ブロック内の次の文は `{{#include ../../../examples/hello-name/HelloName.lean:line4}}` です。
`IO.FS.Stream.getLine` の型は `IO.FS.Stream → IO String` で、これはストリームから文字列を返す `IO` アクションを表す関数です。
再度、これはアクセサー表記の例です。
この `IO` アクションが実行され、プログラムはユーザーが完全な入力行を入力するのを待ちます。
ユーザーが `"David"` と書いたと仮定します。
生成された行（`"David\n"`）は `input` に関連付けられ、ここでエスケープシーケンス `\n` は改行文字を示します。

```lean
{{#include ../../../examples/hello-name/HelloName.lean:block5}}
```

次の行、`{{#include ../../../examples/hello-name/HelloName.lean:line5}}` は `let` 文です。
このプログラム内の他の `let` 文とは異なり、こちらは `:=` を使用しています。
これは、式が評価されるが、その結果の値は `IO` アクションである必要はなく、実行されません。
この場合、`String.dropRightWhile` は文字列と文字の述語を取り、述語を満たす文字を持つ文字列の末尾から削除された新しい文字列を返します。
例えば
、
```lean
{{#example_in Examples/HelloWorld.lean dropBang}}
```

は

```output info
{{#example_out Examples/HelloWorld.lean dropBang}}
```

となり、

```lean
{{#example_in Examples/HelloWorld.lean dropNonLetter}}
```

は

```output info
{{#example_out Examples/HelloWorld.lean dropNonLetter}}
```

となります。
これらの例では、文字列の右側から非英数字文字が削除されています。
プログラムの現在の行では、空白文字（改行を含む）が入力文字列の右側から削除され、`"David"` が生成され、これはブロックの残りの部分において `name` と関連付けられます。


## ユーザに挨拶する（Greeting the User）

`do` ブロックで実行する必要のあるものは、次の 1 つの文だけです：
```lean
{{#include ../../../examples/hello-name/HelloName.lean:line6}}
```

`putStrLn` に渡される文字列の引数は文字列補間を使用して構築され、文字列 `"Hello, David!"` が生成されます。
この文は式であるため、この文字列を新しい行と共に標準出力に出力する `IO` アクションを生成するために評価されます。
式が評価されると、生成された `IO` アクションが実行され、挨拶が表示されます。


## 値としての `IO` アクション（`IO` Actions as Values）

上記の説明では、なぜ式の評価と `IO` アクションの実行の区別が必要なのかがわかりにくいかもしれません。
結局のところ、各アクションは生成された直後に実行されます。
なぜ他の言語で行われているように、評価中に効果を実行しないのでしょうか？

その答えは二つあります。
まず、評価と実行を分離することで、プログラムはどの関数が副作用を持つことができるかについて明示的である必要があります。
効果のないプログラムの部分は、数学的な推論に非常に適しており、プログラマの頭の中で行うか、Leanの形式的な証明のための施設を使用して行うかに関係なく、この分離はバグを回避しやすくすることができます。
さらに、生成された瞬間にすぐに実行される必要のない `IO` アクションもあります。
アクションを実行せずに言及できる能力は、通常の関数を制御構造として使用できるようにします。

たとえば、関数 `twice` は、その引数として `IO` アクションを受け取り、最初のアクションを2回実行する新しいアクションを返します。

```lean
{{#example_decl Examples/HelloWorld.lean twice}}
```

たとえば、次のコードを実行すると、

```lean
{{#example_in Examples/HelloWorld.lean twiceShy}}
```

次の結果が表示されます：

```output info
{{#example_out Examples/HelloWorld.lean twiceShy}}
```

これは、アクションを任意の回数実行するバージョンに一般化できます：

```lean
{{#example_decl Examples/HelloWorld.lean nTimes}}
```

`Nat.zero` の基本ケースでは、結果は `pure ()` です。
`pure` 関数は、副作用のない `IO` アクションを作成し、ただし、このアクションは `pure` の引数である `Unit` のコンストラクタを返します。
何もしないし、面白みのないアクションとして、`pure ()` は同時に非常に退屈で非常に便利です。
再帰ステップでは、`do` ブロックを使用して、最初に `action` を実行し、再帰呼び出しの結果を実行するアクションを作成します。
`{{#example_in Examples/HelloWorld.lean nTimes3}}` を実行すると、次の出力が生成されます：

```output info
{{#example_out Examples/HelloWorld.lean nTimes3}}
```

関数を制御構造として使用することに加えて、`IO` アクションがファーストクラスの値であるという事実は、それらを後で実行するためにデータ構造に保存できることを意味します。
たとえば、関数 `countdown` は `Nat` を受け取り、各 `Nat` に対して実行されていない `IO` アクションのリストを返します：

```lean
{{#example_decl Examples/HelloWorld.lean countdown}}
```

この関数は副作用を持たず、何も印刷しません。
たとえば、引数に適用でき、結果のアクションのリストの長さを確認できます：

```lean
{{#example_decl Examples/HelloWorld.lean from5}}
```

このリストには 6 つの要素が含まれています（ゼロ用の `"Blast off!"` アクションを含む各数値用のアクションがあります）：

```lean
{{#example_in Examples/HelloWorld.lean from5length}}
```

```output info
{{#example_out Examples/HelloWorld.lean from5length}}
```

関数 `runActions` はアクションのリストを受け取り、それらをすべて順番に実行する単一のアクションを構築します：

```lean
{{#example_decl Examples/HelloWorld.lean runActions}}
```

その構造は基本的に `nTimes` と同じですが、各 `List.cons` の下に実行されるアクションがあります。
同様に、`runActions` はアクションを実行しません。
それらを実行する新しいアクションを作成し、そのアクションは `main` の一部として実行される位置に配置する必要があります：

```lean
{{#example_decl Examples/HelloWorld.lean main}}
```

このプログラムを実行すると、次の出力が得られます：

```output info
{{#example_out Examples/HelloWorld.lean countdown5}}
```

このプログラムを実行すると何が起こるのでしょうか？
最初のステップは `main` を評価することです。その手順は次のとおりです：
```lean
{{#example_eval Examples/HelloWorld.lean evalMain}}
```
生成された `IO` アクションは `do` ブロックです。
その後、`do` ブロックの各ステップが一度に実行され、予想される出力が生成されます。
最後のステップである `pure ()` には効果がなく、単に `runActions` の基本ケースが必要であるため存在しています。


## 練習（Exercise）

紙に次のプログラムの実行をステップ・バイ・ステップで書いてください：

```lean
{{#example_decl Examples/HelloWorld.lean ExMain}}
```

プログラムの実行をステップ・バイ・ステップで進めながら、式が評価されている時と `IO` アクションが実行されている時を特定してください。
`IO` アクションを実行することで副作用が発生する場合、それを書き留めてください。
これを行った後、Leanでプログラムを実行し、副作用に関する予測が正しいか確認してください。
