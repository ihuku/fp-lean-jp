# 等価性の証明（Proving Equivalence）

末尾再帰とアキュムレータを使用して書き直されたプログラムは、元のプログラムとはかなり異なる見た目になることがあります。
元の再帰関数は通常理解しやすいですが、実行時にスタックを枯渇させる危険があります。
両方のプログラムのバージョンを単純なバグがないことを確認するために例を使用してテストした後、証明を使用してプログラムが等価であることを一度確認できます。

## `sum` の等価性の証明

`sum` の両バージョンが等しいことを証明するために、スタブ証明を含む定理文を記述して始めます：
```leantac
{{#example_in Examples/ProgramsProofs/TCO.lean sumEq0}}
```
予想通り、Leanは未解決のゴールを示します：
```output error
{{#example_out Examples/ProgramsProofs/TCO.lean sumEq0}}
```

`rfl` タクティクはここでは適用できません。なぜなら、`NonTail.sum` と `Tail.sum` は定義的に等しくないからです。
ただし、関数は定義的な等価性だけでなく、他の方法でも等しいことが証明できます。
つまり、2つの関数が同じ入力に対して同じ出力を生成することを証明することで、2つの関数が等しいことが証明できます。
言い換えれば、\\( f = g \\) は、すべての可能な入力 \\( x \\) に対して \\( f(x) = g(x) \\) であることを証明することによって証明できます。この原理は _関数の拡張性_ と呼ばれます。
関数の拡張性は、`NonTail.sum` が `Tail.sum` と等しい理由そのままです。両方が数値のリストを合計するからです。

Leanのタクティク言語では、関数の拡張性は `funext` を使用して呼び出し、任意の引数に使用する名前に続けて指定されます。
この任意の引数はコンテキストに仮定として追加され、ゴールはこの引数を使用して関数が等しいことを証明する証明を要求するものに変更されます：
```leantac
{{#example_in Examples/ProgramsProofs/TCO.lean sumEq1}}
```
```output error
{{#example_out Examples/ProgramsProofs/TCO.lean sumEq1}}
```

このゴールは `xs` 引数に対する帰納法によって証明できます。
両方の `sum` 関数は空のリストに適用されると `0` を返し、これが基本ケースとして機能します。
入力リストの先頭に数値を追加すると、両方の関数はその数値を結果に加えるため、これが帰納ステップとして機能します。
`induction` タクティクを呼び出すと、2つのゴールが生成されます。

```leantac
{{#example_in Examples/ProgramsProofs/TCO.lean sumEq2a}}
```
```output error
{{#example_out Examples/ProgramsProofs/TCO.lean sumEq2a}}
```
```output error
{{#example_out Examples/ProgramsProofs/TCO.lean sumEq2b}}
```

`nil` の基本ケースは、空リストが渡されたときに両方の関数が `0` を返すため、`rfl` を使用して解決できます：
```leantac
{{#example_in Examples/ProgramsProofs/TCO.lean sumEq3}}
```

帰納ステップを解決する最初のステップは、ゴールを簡単にすることで、`NonTail.sum` と `Tail.sum` を展開するように `simp` に要請することです：
```leantac
{{#example_in Examples/ProgramsProofs/TCO.lean sumEq4}}
```
```output error
{{#example_out Examples/ProgramsProofs/TCO.lean sumEq4}}
```
`Tail.sum` を展開することで、それがすぐに `Tail.sumHelper` に委譲することが明らかになり、これも簡略化する必要があります：
```leantac
{{#example_in Examples/ProgramsProofs/TCO.lean sumEq5}}
```
生成されたゴールでは、`sumHelper` が計算のステップを踏んで、アキュムレータに `y` を追加しました：
```output error
{{#example_out Examples/ProgramsProofs/TCO.lean sumEq5}}
```

帰納仮説を使用して、ゴールから `NonTail.sum` のすべての言及を削除できます：
```leantac
{{#example_in Examples/ProgramsProofs/TCO.lean sumEq6}}
```
```output error
{{#example_out Examples/ProgramsProofs/TCO.lean sumEq6}}
```

この新しいゴールでは、リストの合計に何らかの数値を追加することが、その数値を `sumHelper` の初期アキュムレータとして使用することと同じであることが述べられています。
明確さのために、この新しいゴールを別の定理として証明できます：
```leantac
{{#example_in Examples/ProgramsProofs/TCO.lean sumEqHelperBad0}}
```
```output error
{{#example_out Examples/ProgramsProofs/TCO.lean sumEqHelperBad0}}
```

これも再び、基本ケースでは `rfl` を使用した帰納法の証明です：
```leantac
{{#example_in Examples/ProgramsProofs/TCO.lean sumEqHelperBad1}}
```
```output error
{{#example_out Examples/ProgramsProofs/TCO.lean sumEqHelperBad1}}
```

これは帰納ステップなので、ゴールは帰納仮説 `ih` に一致するように簡略化されるべきです。
`Tail.sum` と `Tail.sumHelper` の定義を使用して簡略化すると、次のようになります：
```leantac
{{#example_in Examples/ProgramsProofs/TCO.lean sumEqHelperBad2}}
```
```output error
{{#example_out Examples/ProgramsProofs/TCO.lean sumEqHelperBad2}}
```

理想的には、帰納仮説を使用して `Tail.sumHelper (y + n) ys` を置き換えることができますが、一致しません。
帰納仮説は `Tail.sumHelper n ys` に対してではなく、`Tail.sumHelper (y + n) ys` に対して使用する必要があります。
言い換えれば、この証明は行き詰まっているということです。

## 2度目の挑戦（A Second Attempt）

証明をこなそうとするのではなく、一歩後ろに退いて考える時が来ました。
なぜ、関数の末尾再帰バージョンが非末尾再帰バージョンと等しいのか、それは何故でしょうか？
根本的に言えば、リストの各エントリにおいて、アキュムレータは再帰の結果に追加されるものと同じだけ成長します。
この洞察を使用して、エレガントな証明を書くことができます。
重要なのは、帰納法による証明は、帰納仮説が_任意の_アキュムレータ値に適用できるように設定されなければならないことです。

以前の試みを捨てて、この洞察は以下のように記述できます：
```leantac
{{#example_in Examples/ProgramsProofs/TCO.lean nonTailEqHelper0}}
```
この文では、`n`がコロンの後にある型の一部であることが非常に重要です。
生成されたゴールは「∀（n：Nat）」で始まり、これは「すべての`n`に対して」という意味です。
```output error
{{#example_out Examples/ProgramsProofs/TCO.lean nonTailEqHelper0}}
```
帰納タクティクを使用すると、ゴールにはこの「すべての」文が含まれるようになります：
```leantac
{{#example_in Examples/ProgramsProofs/TCO.lean nonTailEqHelper1a}}
```
「nil」の場合、ゴールは次のようになります：
```output error
{{#example_out Examples/ProgramsProofs/TCO.lean nonTailEqHelper1a}}
```
「cons」の帰細ステップの場合、帰納仮説と特定のゴールの両方が「すべての`n`」を含むことになります：
```output error
{{#example_out Examples/ProgramsProofs/TCO.lean nonTailEqHelper1b}}
```
言い換えれば、ゴールは証明が難しくなりましたが、帰納仮説はそれに応じてより有用になりました。

「すべてのxから始まる」という文の数学的証明は、何らかの任意のxを仮定し、その文を証明するものです。
「任意の」とは、xの追加のプロパティを仮定しないことを意味し、結果の文は_任意の_ xに対して機能します。
Leanでは、「すべての」文は依存関数です：それが適用される具体的な値が何であっても、それは命題の証拠を返します。
同様に、任意のxを選択するプロセスは、`fun x => ...`を使用することと同じです。
タクティク言語では、この任意のxを選択するプロセスは、タクティクスクリプトが完了したときに背後で関数を生成する`intro`タクティクによって実行されます。
`intro`タクティクにはこの任意の値に使用される名前が提供される必要があります。

`nil`の場合、`intro`タクティクを使用すると、ゴールから`∀ (n : Nat),`が削除され、`n : Nat`という仮定が追加されます：
```leantac
{{#example_in Examples/ProgramsProofs/TCO.lean nonTailEqHelper2}}
```
```output error
{{#example_out Examples/ProgramsProofs/TCO.lean nonTailEqHelper2}}
```
この命題の等式の両側は、定義的に`n`に等しいため、`rfl`で十分です：
```leantac
{{#example_in Examples/ProgramsProofs/TCO.lean nonTailEqHelper3}}
```
`cons`のゴールにも「すべての」が含まれています：
```output error
{{#example_out Examples/ProgramsProofs/TCO.lean nonTailEqHelper3}}
```
これは`intro`の使用を示唆しています。
```leantac
{{#example_in Examples/ProgramsProofs/TCO.lean nonTailEqHelper4}}
```
```output error
{{#example_out Examples/ProgramsProofs/TCO.lean nonTailEqHelper4}}
```
証明のゴールには今、`y :: ys`に適用された`NonTail.sum`と`Tail.sumHelper`の両方が含まれています。
簡略化は次のステップをより明確にできます：
```leantac
{{#example_in Examples/ProgramsProofs/TCO.lean nonTailEqHelper5}}
```
```output error
{{#example_out Examples/ProgramsProofs/TCO.lean nonTailEqHelper5}}
```
このゴールは帰納仮説に非常に近いものです。
一致しない部分は2つあります：
* 方程式の左辺は `n + (y + NonTail.sum ys)` ですが、帰納仮説では左辺が `NonTail.sum ys` に追加される数値である必要があります。
  言い換えれば、このゴールは `(n + y) + NonTail.sum ys` に書き換える必要があります。これは自然数の加法が結合的であるため有効です。
* 左辺が `(y + n) + NonTail.sum ys` に書き換えられた場合、右辺のアキュムレータ引数は一致するために `y + n` ではなく `n + y` である必要があります。
  この書き換えも有効で、加法は可換であるためです。

加法の結合性と可換性は、すでにLeanの標準ライブラリで証明されています。
結合性の証明は `{{#example_in Examples/ProgramsProofs/TCO.lean NatAddAssoc}}` という名前で、その型は `{{#example_out Examples/ProgramsProofs/TCO.lean NatAddAssoc}}` です。一方、可換性の証明は `{{#example_in Examples/ProgramsProofs/TCO.lean NatAddComm}}` と呼ばれ、その型は `{{#example_out Examples/ProgramsProofs/TCO.lean NatAddComm}}` です。
通常、`rw`タクティクには型が等式である式が提供されます。
ただし、引数が代数関数で、その戻り値の型が等式である場合、その等式がゴール内の何かに一致するような関数の引数を見つけようとします。
結合性を適用できる機会は1つだけですが、`{{#example_in Examples/ProgramsProofs/TCO.lean NatAddAssoc}}` の等式の右側が証明のゴールに一致するため、書き換えの方向を逆にする必要があります：
```leantac
{{#example_in Examples/ProgramsProofs/TCO.lean nonTailEqHelper6}}
```
```output error
{{#example_out Examples/ProgramsProofs/TCO.lean nonTailEqHelper6}}
```
ただし、`{{#example_in Examples/ProgramsProofs/TCO.lean NatAddComm}}` を直接書き換えると、誤った結果につながります。
`rw`タクティクは書き換えの位置を誤って推測し、意図しないゴールに導きます：
```leantac
{{#example_in Examples/ProgramsProofs/TCO.lean nonTailEqHelper7}}
```
```output error
{{#example_out Examples/ProgramsProofs/TCO.lean nonTailEqHelper7}}
```
これは、`Nat.add_comm` に `y` と `n` を明示的に引数として提供することで修正できます：
```leantac
{{#example_in Examples/ProgramsProofs/TCO.lean nonTailEqHelper8}}
```
```output error
{{#example_out Examples/ProgramsProofs/TCO.lean nonTailEqHelper8}}
```
ゴールは現在、帰納仮説に一致します。
特に、帰納仮説の型は依存関数型です。
`ih` を `n + y` に適用すると、まさに必要な型が得られます。
`exact` タクティクは、その引数がまさに所望の型である場合、証明ゴールを完了させます：
```leantac
{{#example_decl Examples/ProgramsProofs/TCO.lean nonTailEqHelperDone}}
```

実際の証明には、ゴールを補助定理（ヘルパー）の型に一致させるためにわずかな追加作業しか必要ありません。最初のステップは、まだ関数の拡張性を呼び出すことです：

```leantac
{{#example_in Examples/ProgramsProofs/TCO.lean nonTailEqReal0}}
```

```output error
{{#example_out Examples/ProgramsProofs/TCO.lean nonTailEqReal0}}
```

次のステップは、`Tail.sum`を展開し、`Tail.sumHelper`を表示することです：

```leantac
{{#example_in Examples/ProgramsProofs/TCO.lean nonTailEqReal1}}
```

```output error
{{#example_out Examples/ProgramsProofs/TCO.lean nonTailEqReal1}}
```

これを行った後、型はほぼ一致します。ただし、補助定理の左側に追加の被加数があります。
言い換えれば、証明のゴールは `NonTail.sum xs = Tail.sumHelper 0 xs` ですが、`non_tail_sum_eq_helper_accum`を `xs` と `0` に適用すると、型は `0 + NonTail.sum xs = Tail.sumHelper 0 xs` になります。
もう1つの標準ライブラリの証明、`{{#example_in Examples/ProgramsProofs/TCO.lean NatZeroAdd}}` は型 `{{#example_out Examples/ProgramsProofs/TCO.lean NatZeroAdd}}` を持っています。
これを `NonTail.sum xs` に適用すると、所望のゴールになるように右から左に書き換えることができます：

```leantac
{{#example_in Examples/ProgramsProofs/TCO.lean nonTailEqReal2}}
```

```output error
{{#example_out Examples/ProgramsProofs/TCO.lean nonTailEqReal2}}
```

最後に、補助定理を使用して証明を完成させることができます：

```leantac
{{#example_decl Examples/ProgramsProofs/TCO.lean nonTailEqRealDone}}
```

この証明は、アキュムレータを渡す末尾再帰関数が非末尾再帰バージョンと等しいことを証明する際に使用できる一般的なパターンを示しています。
最初のステップは、初期アキュムレータ引数と最終結果の関係を見つけることです。
例えば、初期アキュムレータを `n` で始めると、最終の合計は `n` に追加され、初期アキュムレータを `ys` で始めると、最終の逆順リストが `ys` に前置されます。
2番目のステップは、この関係を定理文として書き留め、帰納法によって証明することです。
実際にはアキュムレータは常に「0」や「[]」などの中立的な値で初期化されますが、初期アキュムレータを任意の値に設定できるようにするこのより一般的な文は、強力な帰納仮説を得るために必要です。
最後に、この補助定理を実際の初期アキュムレータ値とともに使用して、所望の証明が得られます。
例えば、`non_tail_sum_eq_tail_sum`では、アキュムレータが「0」で指定されています。
このため、中立的な初期アキュムレータ値が正しい位置に現れるようにゴールを書き換える必要があるかもしれません。


## 練習（Exercise）

### ウォーミングアップ（Warming Up）

`induction`タクティクを使用して、`Nat.zero_add`、`Nat.add_assoc`、および `Nat.add_comm` の証明を自分で行ってください。

### アキュムレータ証明のさらなる例（More Accumulator Proofs）

#### リストの反転（Reversing Lists）

`sum`の証明を `NonTail.reverse` と `Tail.reverse` の証明に適応させてください。
最初のステップは、`Tail.reverseHelper` に渡されるアキュムレータ値と非末尾再帰の反転との関係を考えることです。
`Tail.sumHelper` にアキュムレータに数値を追加するのが合計全体に数値を追加するのと同じように、`Tail.reverseHelper` のアキュムレータに新しいエントリを追加することは、全体の結果に何らかの変更をもたらすことに等しいです。
関係が明確になるまで、鉛筆と紙で3つまたは4つの異なるアキュムレータの値を試してみてください。
この関係を使用して適切な補助定理を証明してください。
その後、全体の定理を書き留めます。
`NonTail.reverse` と `Tail.reverse` は多相的なため、その等式を述べるには、Leanが`α`の型をどれに使用するかを判断しようとしないように、`@`を使用する必要があります。
`α`が通常の引数として扱われると、`funext`を `α` と `xs` の両方に適用すべきです：

```leantac
{{#example_in Examples/ProgramsProofs/TCO.lean reverseEqStart}}
```

これにより、適切なゴールが生成されます：

```output error
{{#example_out Examples/ProgramsProofs/TCO.lean reverseEqStart}}
```

#### 階乗（Factorial）

前のセクションの演習で示された `NonTail.factorial` が、末尾再帰による方法と等しいことを証明してください。これは、アキュムレータと結果との関係を見つけ、適切な補助定理を証明することによって行います。