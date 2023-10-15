# 間奏：タクティクス、帰納法、そして証明（Interlude: Tactics, Induction, and Proofs）

## 証明とユーザーインターフェースに関する注意（A Note on Proofs and User Interfaces）

この本では、証明が一気に書かれ、Leanに提出され、その後、未だに行われていないことを説明するエラーメッセージが返されるプロセスを提示しています。
Leanとの対話の実際のプロセスはもっと快適です。
Leanは、カーソルを移動させながら証明に関する情報を提供し、証明を容易にするいくつかの対話型の機能があります。
詳細については、Lean開発環境のドキュメンテーションをご参照ください。

この本のアプローチは、証明を段階的に構築し、その結果として表示されるメッセージを示すもので、専門家が使用するプロセスよりもはるかに遅いですが、証明を書く際にLeanが提供する対話型フィードバックの種類を示しています。
同時に、不完全な証明が完全性に向かって進化する様子は、証明に対する有用な視点です。
証明を書くスキルが向上するにつれて、Leanのフィードバックはエラーというよりも、自分自身の思考プロセスをサポートしているように感じるようになるでしょう。
対話的なアプローチを学ぶことは非常に重要です。


## 再帰と帰納法（Recursion and Induction）

前章の関数 `plusR_succ_left` と `plusR_zero_left` は、2つの視点から見ることができます。
一方で、それらは命題の証拠を構築する再帰関数であり、他の再帰関数がリスト、文字列、または他のデータ構造を構築するように、証拠を構築します。
一方、それらは _数学的帰納法_ による証明に対応しています。

数学的帰納法は、ある文が2つのステップで _すべての_ 自然数について証明される証明テクニックです：
 1. その文が \\( 0 \\) に対して成立することを示します。これを _基本ケース_ と呼びます。
 2. ある任意の数 \\( n \\) についてその文が成立するという仮定のもとで、\\( n + 1 \\) についても成立することを示します。これを _帰納ステップ_ と呼びます。文が \\( n \\) について成立するという仮定は _帰納仮説_ と呼ばれます。

すべての自然数について文を確認することは不可能なため、帰納法は、原理的には特定の自然数に展開できる証明を書く手段を提供します。
たとえば、数値3に対する具体的な証明が必要な場合、まず基本ケースを使用して、その後帰納ステップを3回繰り返し、0、1、2、最後に3について文が成立することを示すことができます。
したがって、それはすべての自然数に対して文を証明します。


## 帰納法タクティク（The Induction Tactic）

再帰関数として帰納法による証明を書くことは、証明の意図を表現するのには常に適していません。
再帰関数は確かに帰納の構造を持っていますが、それらはおそらく証明の _エンコード_ と見なすべきです。
さらに、Leanのタクティクシステムは、再帰関数を明示的に書く際には利用できない証明の構築を自動化する機会を提供しています。
Leanは帰納法に基づいた再帰関数を構築するための帰納 _タクティク_ を提供しています。

帰納法タクティクを使用して `plusR_zero_left` を証明するために、まずそのシグネチャを記述します（これは実際には証明なので、`theorem` を使用します）。
その後、定義の本文として `by induction k` を使用します：
```leantac
{{#example_in Examples/Induction.lean plusR_ind_zero_left_1}}
```
その結果、2つのゴールが存在することが示されます：
```output error
{{#example_out Examples/Induction.lean plusR_ind_zero_left_1}}
```
タクティクブロックは、Leanの型検査器がファイルを処理する際に実行されるプログラムであり、かなりパワフルなCプリプロセッサマクロのようなものです。
タクティクは実際のプログラムを生成します。

タクティク言語では、複数のゴールが存在することがあります。
各ゴールは型といくつかの仮定から構成されます。
これはアンダースコアをプレースホルダーとして使用することに類似しており、ゴールの型は何を証明するかを表し、仮定はスコープ内にあるもので使用できます。
`case zero` のゴールの場合、仮定は存在せず、その型は `Nat.zero = Nat.plusR 0 Nat.zero` です。これは `k` の代わりに `0` を使用した定理文です。
`case succ` のゴールの場合、2つの仮定が存在し、それぞれ `n✝` と `n_ih✝` という名前が付けられています。
裏では、`induction` タクティクは、全体の型を細かくし、`n✝` がパターン内の `Nat.succ` の引数を表します。
仮定 `n_ih✝` は生成された関数を `n✝` に対して再帰的に呼び出した結果を表します。
その型は定理の全体の型であり、ただし `k` の代わりに `n✝` を使用しています。
ゴール `case succ` の一部として達成されるべき型は、 `k` の代わりに `Nat.succ n✝` を使用した定理文です。

`induction` タクティクを使用した結果、2つのゴールが生成されますが、これは数学的帰納法の説明における基本ケースと帰納ステップに対応しています。
基本ケースは `case zero` です。
`case succ` では、`n_ih✝` が帰納仮説に対応し、全体の `case succ` が帰納ステップです。


証明の次のステップは、2つのゴールそれぞれに焦点を当てることです。
`pure ()` が「何もしない」と示すために `do` ブロックで使用できるのと同様に、タクティク言語には何もしない `skip` という文もあります。
これは、Leanの構文がタクティクを必要とする場合でも、どのタクティクを使用すべきかはまだ明確でない場合に使用できます。
`induction` 文の末尾に `with` を追加すると、パターンマッチングに似た構文が提供されます：
```leantac
{{#example_in Examples/Induction.lean plusR_ind_zero_left_2a}}
```
2つの `skip` 文にはそれぞれメッセージが関連付けられています。
最初のものは基本ケースを示します：
```output error
{{#example_out Examples/Induction.lean plusR_ind_zero_left_2a}}
```
2番目のものは帰納ステップを示します：
```output error
{{#example_out Examples/Induction.lean plusR_ind_zero_left_2b}}
```
帰納ステップでは、ダガーのついたアクセス不可能な名前が `succ` の後に提供された `n` と `ih` に置き換えられています。

`induction ... with` の後のケースはパターンではありません。これらはゴールの名前と、ゼロ個以上の名前で構成されます。
これらの名前はゴール内で導入された仮定に使用され、ゴールが導入する名前よりも多くの名前を提供することはエラーです：
```leantac
{{#example_in Examples/Induction.lean plusR_ind_zero_left_3}}
```
```output error
{{#example_out Examples/Induction.lean plusR_ind_zero_left_3}}
```

基本ケースに焦点を当てると、`rfl` タクティクは再帰関数内でも同様に機能します：
```leantac
{{#example_in Examples/Induction.lean plusR_ind_zero_left_4}}
```
証明の再帰関数バージョンでは、型の注釈によって期待される型がより理解しやすいものになりました。
タクティク言語では、ゴールを解決しやすくするためのさまざまな方法があります。
`unfold` タクティクは定義済みの名前をその定義で置き換えます：
```leantac
{{#example_in Examples/Induction.lean plusR_ind_zero_left_5}}
```
すると、ゴール内の等式の右辺が `Nat.plusR 0 n + 1` になり、`Nat.plusR 0 (Nat.succ n)` の代わりになりました：
```output error
{{#example_out Examples/Induction.lean plusR_ind_zero_left_5}}
```

`congrArg` などの関数や `▸` のような演算子に訴える代わりに、等式の証明を変換するために使用できるタクティクがあります。
最も重要なのは `rw` で、等式の証明を取るリストを指定し、ゴール内で左辺を右辺で置き換えます。
これは `plusR_zero_left` ではほぼ正しいことを行います：
```leantac
{{#example_in Examples/Induction.lean plusR_ind_zero_left_6}}
```
ただし、書き換えの方向が間違っています。
`n` を `Nat.plusR 0 n` に置き換えることで、ゴールがより複雑になりました：
```output error
{{#example_out Examples/Induction.lean plusR_ind_zero_left_6}}
```
これは、`rewrite` の呼び出しに `ih` の前に左矢印を置くことで修正できます。これにより、等式の右辺を左辺で置き換えるように指示されます：
```leantac
{{#example_decl Examples/Induction.lean plusR_zero_left_done}}
```
この書き換えにより、方程式の両側が同一になり、Leanは `rfl` を自動で処理します。
これで証明が完了しました。


## タクティクゴルフ（Tactic Golf）

これまでのところ、タクティク言語はその真の価値を示していませんでした。
上記の証明は再帰関数よりも短くないだけで、フルのLean言語の代わりに特定のドメイン向けの言語で書かれています。
しかし、タクティクを使用した証明は、より短く、簡単で保守的になることがあります。
ゴルフのゲームでは、スコアが低いほど良いように、タクティクゴルフのゲームでは証明が短いほど良いのです。

`plusR_zero_left` の帰納ステップは、簡略化タクティク `simp` を使用して証明できます。
`simp` を単体で使用しても助けにはなりません：
```leantac
{{#example_in Examples/Induction.lean plusR_zero_left_golf_1}}
```
```output error
{{#example_out Examples/Induction.lean plusR_zero_left_golf_1}}
```
ただし、`simp` を使用するように設定することができ、一連の定義を利用するようにできます。
`rw` と同様、これらの引数はリストで提供されます。
`simp` に `Nat.plusR` の定義を考慮に入れるように指示すると、より単純なゴールになります：
```leantac
{{#example_in Examples/Induction.lean plusR_zero_left_golf_2}}
```
```output error
{{#example_out Examples/Induction.lean plusR_zero_left_golf_2}}
```
特に、ゴールは今、帰納仮説と同一です。
簡略化タクティクは、単純な等式の証明だけでなく、ゴールを `Nat.succ A = Nat.succ B` から `A = B` に自動的に置き換えることも行います。
帰納仮説 `ih` がまさに適切な型を持っているため、`exact` タクティクを使用することを示すことができます：
```leantac
{{#example_decl Examples/Induction.lean plusR_zero_left_golf_3}}
```

ただし、`exact` の使用は多少デリケートです。
帰納仮説の名前を変更すると、証明が機能しなくなる可能性があります。
`assumption` タクティクは、仮定のいずれかがそれに一致する場合に、現在のゴールを解決します：
```leantac
{{#example_decl Examples/Induction.lean plusR_zero_left_golf_4}}
```

この証明は、前の証明と比べてそれほど短くありません。
ただし、多くの種類のゴールを解決できる点を活用して、一連の変換を行うことで、はるかに短くなる可能性があります。
最初のステップは、`induction` の最後の `with` を省略することです。
構造的で読みやすい証明の場合、`with` 構文は便利です。
ケースが欠けているとエラーを出力し、帰納の構造を明確に表示します。
ただし、証明を短縮するには、より自由なアプローチが必要なことがあります。

`with` を使用せずに `induction` を使うと、2つのゴールを持つ証明状態が得られます。
`case` タクティクを使用して、`induction ... with` タクティクのブランチと同じように、それらのうちの1つを選択できます。
言い換えれば、次の証明は前の証明と等価です：
```leantac
{{#example_decl Examples/Induction.lean plusR_zero_left_golf_5}}
```

単一のゴールを持つコンテキスト（具体的には、`k = Nat.plusR 0 k` というゴール）で、`induction k` タクティクは2つのゴールを生成します。
一般的に、タクティクはエラーを出力するか、ゴールを取り、それをゼロ個以上の新しいゴールに変換します。
各新しいゴールは、証明すべき残りの部分を表します。
結果がゴールがゼロ個の場合、タクティクは成功し、その証明のその部分は完了です。

`<;>` 演算子は、2つのタクティクを引数に取り、新しいタクティクを生成します。
`T1 <;> T2` は `T1` を現在のゴールに適用し、`T1` によって作成された _すべて_ のゴールに対して `T2` を適用します。
言い換えれば、`<;>` は多くの種類のゴールを解決できる汎用タクティクを、一度に複数の新しいゴールに使用できるようにします。
そのような一般的なタクティクの1つが `simp` です。

`simp` はベースケースの証明を完了し、帰納ステップの証明を進めることができるため、`induction` と `<;>` と組み合わせて使用することで証明が短縮されます：
```leantac
{{#example_in Examples/Induction.lean plusR_zero_left_golf_6a}}
```
その結果、変換された帰納ステップの1つだけのゴールが得られます：
```output error
{{#example_out Examples/Induction.lean plusR_zero_left_golf_6a}}
```
このゴールに `assumption` を実行することで証明が完了します：
```leantac
{{#example_decl Examples/Induction.lean plusR_zero_left_golf_6}}
```
ここでは、`ih` が明示的に名前付けられていなかったため、`exact` は使用できませんでした。

初心者にとって、この証明は読みやすくなるわけではありません。
ただし、専門家のユーザーにとって一般的なパターンは、`simp` のような強力なタクティクを使用して多くの単純なケースを処理し、証明の本文を興味深いケースに焦点を当てることです。
さらに、これらの証明は、証明に関与する関数とデータ型の小さな変更に対しても、通常よりも堅牢である傾向があります。
タクティクゴルフのゲームは、証明を書く際の優れたスタイルとセンスを開発するための有用な要素です。


## 他の型における帰納（Induction on Other Datatypes）

数学的帰納法は、自然数に関する命題を`Nat.zero`の基本ケースと`Nat.succ`の帰納ステップを提供することによって証明します。
帰納法の原則は、他のデータ型に対しても有効です。
再帰引数を持たないコンストラクタは基本ケースを形成し、再帰引数を持つコンストラクタは帰納ステップを形成します。
帰納法によって証明を行う能力は、それが _帰納的_ データ型と呼ばれる理由そのものです。

その一例が、二分木に対する帰納法です。
二分木に対する帰納法は、二つのステップで_すべて_の二分木についての命題を証明する証明技法です：
 1. 命題が `BinTree.leaf` に適用されることが示されます。これは基本ケースと呼ばれます。
 2. 仮定として、命題がいくつかの任意の木 `l` と `r` に適用されることが示され、新しいデータ点 `x` が任意に選択された `BinTree.branch l x r` についても成立することが示されます。これを帰納ステップと呼びます。`l` と `r` について命題が成立するという仮定は、帰納仮説と呼ばれます。

`BinTree.count` は木の枝の数を数えます：
```lean
{{#example_decl Examples/Induction.lean BinTree_count}}
```
[木のミラーリング](monads/conveniences.md#leading-dot-notation) は、それに含まれる枝の数を変更しません。
これは木に対する帰納法を使用して証明できます。
最初のステップは、定理を述べ、`induction` を呼び出すことです：
```leantac
{{#example_in Examples/Induction.lean mirror_count_0a}}
```
基本ケースでは、葉のミラーリングの枝の数を葉の枝の数と同じであると述べています：
```output error
{{#example_out Examples/Induction.lean mirror_count_0a}}
```
帰納ステップでは、左右のサブツリーをミラーリングしてもその枝の数には影響を及ぼさないという仮定を行い、これらのサブツリーを持つ枝をミラーリングしても全体の枝の数が保存される証明を求めています：
```output error
{{#example_out Examples/Induction.lean mirror_count_0b}}
```

次の英語の文章を日本語に翻訳します：

基本ケースは`leaf`を`leaf`に対応づけることで真となり、したがって左側と右側は定義的に等しいです。
これは、`BinTree.mirror`を展開するように`simp`を使用して表現できます：
```leantac
{{#example_in Examples/Induction.lean mirror_count_1}}
```
帰納ステップでは、ゴール内で即座に帰納仮説と一致する部分はありません。
`BinTree.count`および`BinTree.mirror`の定義を使用して簡略化すると、次の関係が明らかになります：
```leantac
{{#example_in Examples/Induction.lean mirror_count_2}}
```
```output error
{{#example_out Examples/Induction.lean mirror_count_2}}
```
両方の帰納仮説を使用して、ゴールの左側を右側にほぼ等しいものに書き換えることができます：
```leantac
{{#example_in Examples/Induction.lean mirror_count_3}}
```
```output error
{{#example_out Examples/Induction.lean mirror_count_3}}
```

`simp_arith`タクティク、追加の算術的同値性を使用できる`simp`のバージョン、はこのゴールを証明するのに十分です。次のようになります：
```leantac
{{#example_decl Examples/Induction.lean mirror_count_4}}
```

展開すべき定義の他に、簡略化器には証明のゴールを簡略化する際に使用する等式証明の名前も渡すことができます。
`BinTree.mirror_count`は次のようにも書けます：
```leantac
{{#example_decl Examples/Induction.lean mirror_count_5}}
```
証明がより複雑になると、前提条件を手動で列挙することは面倒になります。
さらに、前提条件の名前を手動で書くことは、証明ステップを複数のサブゴールに再利用するのが難しくなることがあります。
`*`引数を`simp`または`simp_arith`に渡すことで、ゴールを簡略化または解決する際に_すべて_の前提条件を使用するように指示できます。
言い換えれば、証明は次のようにも書けます：
```leantac
{{#example_decl Examples/Induction.lean mirror_count_6}}
```
両方のブランチが簡略化器を使用しているため、証明は次のように簡約できます：
```leantac
{{#example_decl Examples/Induction.lean mirror_count_7}}
```

## 演習（Exercises）

 * `induction ... with`タクティクを使用して`plusR_succ_left`を証明してください。
 * `plus_succ_left`の証明を1行で`<;>`を使用して書き換えてください。
 * リストの連結が結合法則であることをリストの帰納法を使用して証明してください：`theorem List.append_assoc (xs ys zs : List α) : xs ++ (ys ++ zs) = (xs ++ ys) ++ zs`