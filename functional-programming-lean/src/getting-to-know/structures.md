# 構造体（Structures）

プログラムを書く際の最初のステップは通常、問題領域の概念を特定し、それらをコード内で適切に表現する方法を見つけることです。時に、ある問題領域の概念は、他のより単純な概念の集合であることがあります。その場合、これらより単純な要素を一つの「パッケージ」としてまとめ、それに意味のある名前を付けることが便利です。Leanでは、これは「構造体 (structures)」を用いて行われ、これはCやRustの`struct`やC#の`record`と類似しています。

構造を定義することは、Leanに完全に新しい型を導入することを意味し、他の型に還元できません。これは、異なる概念を表現する複数の構造が同じデータを含むような場面を考えると、有用なものになります。例えば、点は直交座標または極座標を使用して表現できますが、データとしては、どちらも浮動小数点数のペアとなるでしょう。別々の構造を定義することで、APIクライアントがそれらを混同しないようにするのに役立ちます。

Leanの浮動小数点数型は`Float`と呼ばれ、浮動小数点数は通常の表記法で書かれます。

```lean
{{#example_in Examples/Intro.lean onePointTwo}}
```
```output info
{{#example_out Examples/Intro.lean onePointTwo}}
```
```lean
{{#example_in Examples/Intro.lean negativeLots}}
```
```output info
{{#example_out Examples/Intro.lean negativeLots}}
```
```lean
{{#example_in Examples/Intro.lean zeroPointZero}}
```
```output info
{{#example_out Examples/Intro.lean zeroPointZero}}
```

浮動小数点数が小数点を伴って書かれている場合、Leanは型`Float`を自動的に推論します。小数点を伴わない場合、型の注釈が必要かもしれません。

```lean
{{#example_in Examples/Intro.lean zeroNat}}
```
```output info
{{#example_out Examples/Intro.lean zeroNat}}
```

```lean
{{#example_in Examples/Intro.lean zeroFloat}}
```
```output info
{{#example_out Examples/Intro.lean zeroFloat}}
```

直交座標点は、`x` と `y` と呼ばれる2つの「Float」フィールドを持つ構造です。
これは`structure`キーワードを使用して宣言されます。

```lean
{{#example_decl Examples/Intro.lean Point}}
```

この宣言の後、`Point` は新しい構造型です。
最後の行は「deriving Repr」と書かれており、Leanに対して型 `Point` の値を表示するコードを生成するように求めています。
このコードは、プログラマが評価結果を表示するために使用され、Pythonの「repr」関数に類似しています。
また、コンパイラが生成した表示コードを上書きすることも可能です。

構造型の値を作成する一般的な方法は、波括弧内でそのすべてのフィールドに値を提供することです。
直交座標平面の原点は、`x` と `y` が両方ともゼロの場所です。

```lean
{{#example_decl Examples/Intro.lean origin}}
```

`Point`の定義内の`deriving Repr`行が省略された場合、`{{#example_in Examples/Intro.lean PointNoRepr}}` を試みると、関数の引数を省略した場合に発生するのと似たエラーが発生します。

```output error
{{#example_out Examples/Intro.lean PointNoRepr}}
```

そのメッセージは、評価の結果をユーザーに伝える方法が評価機構にわからないことを示しています。

幸運なことに、「deriving Repr」を使用すると、`{{#example_in Examples/Intro.lean originEval}}` の結果は、`origin` の定義と非常に似ています。

```output info
{{#example_out Examples/Intro.lean originEval}}
```

構造がデータのコレクションを「まとめて」1つの単位として扱うために存在するため、構造体の個々のフィールドを取り出すことも重要です。
これは、C、Python、またはRustのように、ドット記法を使用して行われます。

```lean
{{#example_in Examples/Intro.lean originx}}
```
```output info
{{#example_out Examples/Intro.lean originx}}
```

```lean
{{#example_in Examples/Intro.lean originy}}
```
```output info
{{#example_out Examples/Intro.lean originy}}
```

これは、構造体を引数として受け取る関数を定義するために使用できます。
たとえば、点の加算は基になる座標値を足すことで実行されます。
`{{#example_in Examples/Intro.lean addPointsEx}}` が以下のような結果を示すはずです。

```output info
{{#example_out Examples/Intro.lean addPointsEx}}
```

この関数自体は、`p1` と `p2` と呼ばれる2つの`Points`を引数として受け取ります。
生成される点は、`p1` と `p2` の両方の`x`と`y`のフィールドに基づいています：
```lean
{{#example_decl Examples/Intro.lean addPoints}}
```

同様に、2つの点間の距離、つまり`x`と`y`の成分の差の二乗の合計の平方根は、次のように記述できます：
```lean
{{#example_decl Examples/Intro.lean distance}}
```
たとえば、(1, 2) と (5, -1) の間の距離は5です：
```lean
{{#example_in Examples/Intro.lean evalDistance}}
```
```output info
{{#example_out Examples/Intro.lean evalDistance}}
```

複数の構造体は同じ名前のフィールドを持つことがあります。
たとえば、三次元の点データ型はフィールド`x`と`y`を共有し、同じフィールド名でインスタンス化できます：
```lean
{{#example_decl Examples/Intro.lean Point3D}}

{{#example_decl Examples/Intro.lean origin3D}}
```
これは、波括弧の構文を使用するために、構造の予想される型を知っている必要があることを意味します。型が不明な場合、Leanは構造をインスタンス化できません。
例えば、
```lean
{{#example_in Examples/Intro.lean originNoType}}
```
これはエラーを引き起こします
```output error
{{#example_out Examples/Intro.lean originNoType}}
```

通常通り、型注釈を提供することでこの状況を改善できます。
```lean
{{#example_in Examples/Intro.lean originWithAnnot}}
```
```output info
{{#example_out Examples/Intro.lean originWithAnnot}}
```

プログラムをより簡潔にするために、Leanは波括弧内で構造体の型注釈を許可します。
```lean
{{#example_in Examples/Intro.lean originWithAnnot2}}
```
```output info
{{#example_out Examples/Intro.lean originWithAnnot2}}
```

## 構造体の更新（Updating Structures）

`Point` の `x` フィールドを `0.0` で置き換える関数 `zeroX` を想像してみてください。
ほとんどのプログラミング言語コミュニティでは、この文は `x` が指すメモリ位置が新しい値で上書きされることを意味するでしょう。
しかし、Lean には可変状態がありません。
関数型プログラミングコミュニティでは、ほとんどの場合、この種の文によって意味されるのは、新しい `x` フィールドを持つ新しい `Point` が割り当てられ、他のすべてのフィールドは入力からの元の値を指すことです。
`zeroX` を書く方法の一つは、この説明を文字通りに実行し、新しい `x` の値を埋めて `y` を手動で転送することです：
```lean
{{#example_decl Examples/Intro.lean zeroXBad}}
```
しかし、このプログラミングスタイルには欠点があります。
まず第一に、構造に新しいフィールドが追加されると、すべてのフィールドを更新するすべての場所を更新する必要があり、保守の難しさが生じます。
第二に、構造に同じ型の複数のフィールドが含まれている場合、コピーアンドペーストのコーディングによってフィールドの内容が重複したり入れ替わったりするリスクがあります。
最後に、プログラムは長くて煩雑になります。

Lean は、構造体内の一部のフィールドを置き換えながら他のフィールドをそのままにするための便利な構文を提供しています。
これは、構造体の初期化で `with` キーワードを使用して行われます。
変更されていないフィールドの情報は `with` の前に来て、新しいフィールドはその後に続きます。
たとえば、`zeroX` は新しい `x` 値のみで書くことができます：

```lean
{{#example_decl Examples/Intro.lean zeroX}}
```

この構造体の更新構文は既存の値を変更するのではなく、古い値と一部のフィールドを共有する新しい値を作成することを覚えておいてください。
例えば、点 `fourAndThree` が与えられた場合：
```lean
{{#example_decl Examples/Intro.lean fourAndThree}}
```
それを評価し、次に `zeroX` を使用して更新し、再度評価すると元の値が返されます：

```lean
{{#example_in Examples/Intro.lean fourAndThreeEval}}
```
```output info
{{#example_out Examples/Intro.lean fourAndThreeEval}}
```
```lean
{{#example_in Examples/Intro.lean zeroXFourAndThreeEval}}
```
```output info
{{#example_out Examples/Intro.lean zeroXFourAndThreeEval}}
```
```lean
{{#example_in Examples/Intro.lean fourAndThreeEval}}
```
```output info
{{#example_out Examples/Intro.lean fourAndThreeEval}}
```

構造体の更新が元の構造を変更しないという事実の結果の一つは、新しい値が古い値から計算される場合の理由付けが容易になることです。
新しい値が提供される場合でも、古い構造へのすべての参照は同じフィールドの値を参照し続けます。



## 構造体の裏側（Behind the Scenes）

すべての構造体には「コンストラクタ (constructor)」があります。
ここでの用語「コンストラクタ」は混乱の原因となるかもしれません。
JavaやPythonなどの言語のコンストラクタとは異なり、Leanのコンストラクタはデータ型の初期化時に実行される任意のコードではありません。
代わりに、コンストラクタは単に新たに割り当てられたデータ構造に格納されるデータを収集するものです。
カスタムコンストラクタを提供してデータを前処理するか無効な引数を拒否することはできません。
これは、言葉「コンストラクタ」が2つの文脈で異なるが関連する意味を持つケースです。

デフォルトで、構造体`S`のコンストラクタの名前は`S.mk`となります。
ここで、`S`は名前空間修飾子であり、`mk`はコンストラクタ自体の名前です。
波括弧の初期化構文を使用する代わりに、コンストラクタは直接適用することもできます。

```lean
{{#example_in Examples/Intro.lean checkPointMk}}
```

ただし、これは一般的には良いLeanのスタイルとは考えられておらず、Leanは標準の構造体の初期化構文を使用してフィードバックを返します。

```output info
{{#example_out Examples/Intro.lean checkPointMk}}
```

コンストラクタは関数型を持っており、これは関数が期待される場所でどこでも使用できることを意味します。
例えば、`Point.mk` は2つの「Float」（それぞれ `x` と `y`）を受け入れ、新しい「Point」を返す関数です。

```lean
{{#example_in Examples/Intro.lean Pointmk}}
```
```output info
{{#example_out Examples/Intro.lean Pointmk}}
```

構造体のコンストラクタ名を上書きするには、名前の冒頭にコロンを2つ付けて書きます。
例えば、`Point.mk` の代わりに `Point.point` を使用するには、次のように記述します：

```lean
{{#example_decl Examples/Intro.lean PointCtorName}}
```

コンストラクタのほかに、構造体の各フィールドに対してアクセッサ関数が定義されます。
これらは構造体の名前空間内でフィールドと同じ名前を持っています。
`Point`の場合、アクセッサ関数として`Point.x`と`Point.y`が生成されます。

```lean
{{#example_in Examples/Intro.lean Pointx}}
```
```output info
{{#example_out Examples/Intro.lean Pointx}}
```

```lean
{{#example_in Examples/Intro.lean Pointy}}
```
```output info
{{#example_out Examples/Intro.lean Pointy}}
```

実際、波括弧で囲んだ構造の構築構文が裏で構造体のコンストラクタの呼び出しに変換されるのと同様に、前述の「addPoints」の定義内の「p1.x」という構文は「Point.x」アクセッサの呼び出しに変換されます。
つまり、`{{#example_in Examples/Intro.lean originx}}` と `{{#example_in Examples/Intro.lean originx1}}` はどちらも次の結果を示します：
```output info
{{#example_out Examples/Intro.lean originx1}}
```

アクセッサのドット表記は、構造体のフィールドだけでなく、任意の数の引数を取る関数にも使用できます。
より一般的に、アクセッサ表記は `TARGET.f ARG1 ARG2 ...` の形を持ちます。
`TARGET` の型が `T` である場合、`T.f` という名前の関数が呼び出されます。
`TARGET` は、通常は最初の引数ですが、必ずしも最初の引数ではない場合があり、`ARG1 ARG2 ...` は残りの引数として順番に提供されます。
例えば、`String.append` はアクセッサ表記から文字列から呼び出すことができます。`String` は `append` フィールドを持つ構造体ではないにもかかわらずです。
```lean
{{#example_in Examples/Intro.lean stringAppendDot}}
```
```output info
{{#example_out Examples/Intro.lean stringAppendDot}}
```
この例では、`TARGET` は `"one string"` を表し、`ARG1` は `" and another"` を表します。

関数 `Point.modifyBoth`（つまり、`Point`名前空間で定義された`modifyBoth`）は、`Point` の両方のフィールドに関数を適用します：
```lean
{{#example_decl Examples/Intro.lean modifyBoth}}
```
関数の引数が後に続くにもかかわらず、アクセッサ表記を使用しても構いません：
```lean
{{#example_in Examples/Intro.lean modifyBothTest}}
```
```output info
{{#example_out Examples/Intro.lean modifyBothTest}}
```
この場合、`TARGET` は `fourAndThree` を表し、`ARG1` は `Float.floor` を表します。
これは、アクセッサ表記の対象は、型が一致する最初の引数として使用されるためであり、必ずしも最初の引数ではないことに注意してください。

## 演習（Exercises）

* `RectangularPrism` という名前の構造体を定義してください。この構造体は直方体の高さ、幅、奥行きをそれぞれ `Float` 型で含む必要があります。

* `volume : RectangularPrism → Float` という名前の関数を定義してください。この関数は直方体の体積を計算する必要があります。

* `Segment` という名前の構造体を定義してください。この構造体は線分をその端点で表現する必要があり、`length : Segment → Float` という名前の関数を定義してください。この関数は線分の長さを計算する必要があります。また、`Segment` は最大で2つのフィールドを持つべきです。

* `RectangularPrism` の宣言によって導入される名前は何でしょうか？

* `Hamster` と `Book` の後続の宣言によって導入される名前は何でしょうか？また、それらの型は何でしょうか？

```lean
{{#example_decl Examples/Intro.lean Hamster}}
```

```lean
{{#example_decl Examples/Intro.lean Book}}
```
