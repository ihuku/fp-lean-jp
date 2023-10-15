# 配列と終了条件（Arrays and Termination）

効率的なコードを書くためには、適切なデータ構造を選択することが重要です。連結リストもその場面があります。リストの末尾を共有する能力が非常に重要なアプリケーションもあります。しかし、データの可変長の連続コレクションのほとんどのユースケースでは、メモリのオーバーヘッドが少なく、配置も良い配列が適しています。

ただし、リストに対して配列は2つの欠点があります：
1. 配列はパターンマッチングではなくインデックスを介してアクセスされ、これは安全性を維持するための[証明義務](../props-proofs-indexing.md)を課します。
2. 配列を左から右に処理するループは末尾再帰関数ですが、各呼び出しで減少する引数がありません。

配列を効果的に使用するには、Leanに配列のインデックスが範囲内であることを証明し、配列のサイズに近づくインデックスがプログラムを終了させることも証明する方法を知る必要があります。これらは等式命題ではなく、不等式命題を使用して表現されます。

## 不等式（Inequality）

異なる型には異なる順序の概念があるため、不等式は `LE` と `LT` という2つの型クラスによって規制されます。[標準型クラス](../type-classes/standard-classes.md#equality-and-ordering)のセクションにある表は、これらのクラスが構文とどのように関連しているかを説明しています：

| 式 | デサグガリング | クラス名 |
|------------|------------|------------|
| `{{#example_in Examples/Classes.lean ltDesugar}}` | `{{#example_out Examples/Classes.lean ltDesugar}}` | `LT` |
| `{{#example_in Examples/Classes.lean leDesugar}}` | `{{#example_out Examples/Classes.lean leDesugar}}` | `LE` |
| `{{#example_in Examples/Classes.lean gtDesugar}}` | `{{#example_out Examples/Classes.lean gtDesugar}}` | `LT` |
| `{{#example_in Examples/Classes.lean geDesugar}}` | `{{#example_out Examples/Classes.lean geDesugar}}` | `LE` |

言い換えると、型は `<` と `≤` 演算子の意味をカスタマイズすることができ、`>` と `≥` は `<` と `≤` からその意味を派生させます。`LT` と `LE` のクラスには、ブールではなく命題を返すメソッドがあります：
```lean
{{#example_decl Examples/ProgramsProofs/Arrays.lean less}}
```

`Nat` のための `LE` のインスタンスは `Nat.le` に委譲します：
```lean
{{#example_decl Examples/ProgramsProofs/Arrays.lean LENat}}
```
`Nat.le` を定義するには、まだ紹介されていないLeanの機能、帰納的に定義された関係が必要です。


### 帰納的に定義された命題、述語、および関係（Inductively-Defined Propositions, Predicates, and Relations）

`Nat.le` は _帰納的に定義された関係_ です。`inductive` は新しいデータ型を作成するために使用できるだけでなく、新しい命題を作成するためにも使用できます。命題が引数を取る場合、それは一部の潜在的な引数に対して真である場合があり、すべての引数に対して真であるわけではありません。複数の引数を取る命題は _関係_ と呼ばれます。

帰納的に定義された命題の各コンストラクタは、それを証明する方法です。言い換えると、命題の宣言は、それが真である異なる証拠の形式を説明しています。引数を持たない命題で、単一のコンストラクタを持つものは非常に簡単に証明できることがあります：
```lean
{{#example_decl Examples/ProgramsProofs/Arrays.lean EasyToProve}}
```
証明はそのコンストラクタを使用することから成り立ちます：
```lean
{{#example_decl Examples/ProgramsProofs/Arrays.lean fairlyEasy}}
```
実際、常に簡単に証明できるはずの命題 `True` は、`EasyToProve` と同じように定義されています：
```lean
{{#example_decl Examples/ProgramsProofs/Arrays.lean True}}
```

引数を取らない帰納的に定義された命題は、帰納的に定義されたデータ型ほど興味深くありません。これはデータ自体が興味深いからです。自然数 `3` は数 `35` とは異なり、3つのピザを注文した人は、30分後にドアに35のピザが届いた場合に不満に感じるでしょう。命題のコンストラクタは、命題が真である方法を説明しますが、命題が証明されると、どの _基本的な_ コンストラクタが使用されたかを知る必要はありません。これは、`Prop` の宇宙において、ほとんどの興味深い帰納的に定義された型が引数を取る理由です。

帰納的に定義された述語 `IsThree` は、その引数が3であることを述べています：
```lean
{{#example_decl Examples/ProgramsProofs/Arrays.lean IsThree}}
```
ここで使用されているメカニズムは、[`HasCol`](../dependent-types/typed-queries.md#column-pointers)などのインデックス付きファミリーと同じですが、結果の型はデータではなく証明可能な命題です。

この述語を使用して、3が本当に3であることを証明することができます：
```lean
{{#example_decl Examples/ProgramsProofs/Arrays.lean threeIsThree}}
```
同様に、`IsFive` はその引数が `5` であることを述べる述語です：
```lean
{{#example_decl Examples/ProgramsProofs/Arrays.lean IsFive}}
```

もし数が3であるなら、それに2を加えた結果は5でなければなりません。これは定理の文として表現できます：

```leantac
{{#example_in Examples/ProgramsProofs/Arrays.lean threePlusTwoFive0}}
```

結果のゴールは関数型を持っています：

```output error
{{#example_out Examples/ProgramsProofs/Arrays.lean threePlusTwoFive0}}
```

したがって、`intro` タクティクを使用して引数を前提に変換できます：

```leantac
{{#example_in Examples/ProgramsProofs/Arrays.lean threePlusTwoFive1}}
```

```output error
{{#example_out Examples/ProgramsProofs/Arrays.lean threePlusTwoFive1}}
```

`n` が3であるという前提が与えられた場合、`IsFive` のコンストラクタを使用して証明を完了できるはずです：

```leantac
{{#example_in Examples/ProgramsProofs/Arrays.lean threePlusTwoFive1a}}
```

しかし、これによりエラーが発生します：

```output error
{{#example_out Examples/ProgramsProofs/Arrays.lean threePlusTwoFive1a}}
```

このエラーは、`n + 2` が `5` と定義的に等しくないためです。通常の関数定義では、前提の `three` 上の依存パターンマッチングを使用して `n` を `3` に細分化できます。依存パターンマッチングのタクティクの相当物は `cases` で、その構文は `induction` と似ています：

```leantac
{{#example_in Examples/ProgramsProofs/Arrays.lean threePlusTwoFive2}}
```

残りのケースでは、`n` は `3` に細分化されました：

```output error
{{#example_out Examples/ProgramsProofs/Arrays.lean threePlusTwoFive2}}
```

`3 + 2` が定義的に `5` に等しいため、コンストラクタは今適用可能です：

```leantac
{{#example_decl Examples/ProgramsProofs/Arrays.lean threePlusTwoFive3}}
```

標準の偽の命題 `False` にはコンストラクタが存在せず、それに対する直接の証拠を提供することは不可能です。 `False` に証拠を提供する唯一の方法は、前提自体が不可能である場合で、これは型システムが到達不可能と見なすことができるコードをマークするために `nomatch` を使用する方法と似ています。 [証明の最初の余興](../props-proofs-indexing.md#connectives) で説明されているように、否定 `Not A` は `A → False` の省略形です。 `Not A` はまた `¬A` と書くこともできます。

4が3であるわけではありません：

```leantac
{{#example_in Examples/ProgramsProofs/Arrays.lean fourNotThree0}}
```

最初の証明目標には `Not` が含まれています：

```output error
{{#example_out Examples/ProgramsProofs/Arrays.lean fourNotThree0}}
```

実際には関数型であることを示すために `simp` を使用できます：

```leantac
{{#example_in Examples/ProgramsProofs/Arrays.lean fourNotThree1}}
```

```output error
{{#example_out Examples/ProgramsProofs/Arrays.lean fourNotThree1}}
```

ゴールが関数型であるため、引数を前提に変換するために `intro` を使用できます。 `Not` の定義自体を展開するために `simp` を保持する必要はありません：

```leantac
{{#example_in Examples/ProgramsProofs/Arrays.lean fourNotThree2}}
```

```output error
{{#example_out Examples/ProgramsProofs/Arrays.lean fourNotThree2}}
```

この証明では、`cases` タクティクがゴールを即座に解決します：

```leantac
{{#example_decl Examples/ProgramsProofs/Arrays.lean fourNotThreeDone}}
```

`Vect String 2` のパターンマッチには `Vect.nil` のケースを含める必要はないように、`IsThree 4` の場合も `isThree` のケースを含める必要はありません。

### 自然数の不等式（Inequality of Natural Numbers）

`Nat.le` の定義にはパラメーターとインデックスがあります：

```lean
{{#example_decl Examples/ProgramsProofs/Arrays.lean NatLe}}
```

パラメーター `n` は小さくすべき数で、インデックスは `n` 以上であるべき数です。 `refl` コンストラクタは両方の数が等しい場合に使用され、`step` コンストラクタはインデックスが `n` より大きい場合に使用されます。

証拠の観点から言えば、\\( n \leq k \\) を証明するには、\\( n + d = m \\) となる \\( d \\) という数を見つけることが必要です。Leanでは、証明は `Nat.le.refl` コンストラクタが \\( d \\) 個の `Nat.le.step` で囲まれたもので構成されます。各 `step` コンストラクタはそのインデックス引数に1を追加し、大きな数に \\( d \\) を追加します。たとえば、4が7以下である証拠は、`refl` の周りに3つの `step` があるものです：

```lean
{{#example_decl Examples/ProgramsProofs/Arrays.lean four_le_seven}}
```

厳密な不等関係は、左側の数に1を加えることで定義されます：

```lean
{{#example_decl Examples/ProgramsProofs/Arrays.lean NatLt}}
```

4が厳密に7より小さいという証拠は、`refl` の周りに2つの `step` があるものです：

```lean
{{#example_decl Examples/ProgramsProofs/Arrays.lean four_lt_seven}}
```

これは `4 < 7` は `5 ≤ 7` と等価であるためです。

## 停止を証明する（Proving Termination）

関数 `Array.map` は関数で配列を変換し、入力配列の各要素に関数を適用した結果を含む新しい配列を返します。これを末尾再帰関数として記述する場合、通常のパターンに従い、アキュムレータ内で出力配列を渡す関数に委任します。アキュムレータは空の配列で初期化されます。アキュムレータを渡すヘルパー関数はまた、配列内の現在のインデックスを追跡する引数も取り、これは `0` から始まります：

```lean
{{#example_decl Examples/ProgramsProofs/Arrays.lean ArrayMap}}
```

ヘルパー関数は、各反復でインデックスが範囲内にあるかどうかを確認する必要があります。もし範囲内にある場合、変換された要素をアキュムレータの末尾に追加し、インデックスを `1` 増やして再度ループする必要があります。範囲内にない場合、終了し、アキュムレータを返す必要があります。このコードの初期の実装は、Leanが配列のインデックスが有効であることを証明できないため失敗します：

```lean
{{#example_in Examples/ProgramsProofs/Arrays.lean mapHelperIndexIssue}}
```

```output error
{{#example_out Examples/ProgramsProofs/Arrays.lean mapHelperIndexIssue}}
```

ただし、条件式は既に配列のインデックスの有効性が要求する正確な条件をチェックしています（つまり、`i < arr.size`）。`if` に名前を追加することで問題を解決できます。これは、配列インデックスの前提条件を使用できるようにする前提条件を追加するからです：

```lean
{{#example_in Examples/ProgramsProofs/Arrays.lean arrayMapHelperTermIssue}}
```

しかし、修正されたプログラムはLeanに受け入れられません。なぜなら再帰呼び出しが入力コンストラクタの引数の一つで行われていないからです。実際、アキュムレータとインデックスの両方が増加し、減少しないからです：

```output error
{{#example_out Examples/ProgramsProofs/Arrays.lean arrayMapHelperTermIssue}}
```

それにもかかわらず、この関数は終了するので、単にそれを `partial` とマークするのは不適切です。

なぜ `arrayMapHelper` は終了するのでしょうか？

各ループでは、インデックス `i` が配列 `arr` の範囲内にあるかどうかを確認します。もし範囲内にあれば、`i` が増加し、ループが繰り返されます。範囲外の場合、プログラムは終了します。なぜなら `arr.size` は有限の数値であり、`i` は有限の回数しか増加しないからです。関数の各呼び出しで関数の引数が減少しないとしても、`arr.size - i` はゼロに向かって減少します。

Leanは、定義の末尾に `termination_by` 句を提供することで終了に別の式を使用するよう指示できます。`termination_by` 句には2つのコンポーネントがあります：関数の引数の名前と、それらの名前を使用した式で、各呼び出しで減少する必要があります。`arrayMapHelper` の最終的な定義は次のようになります：

```lean
{{#example_decl Examples/ProgramsProofs/Arrays.lean ArrayMapHelperOk}}
```

同様の停止性証明を使用して、`Array.find` を記述できます。これは、ブール関数を満たす最初の要素を配列内で見つけ、その要素とそのインデックスの両方を返す関数です：

```lean
{{#example_decl Examples/ProgramsProofs/Arrays.lean ArrayFind}}
```

再び、ヘルパー関数は `arr.size - i` が `i` の増加と共に減少するため、終了します：

```lean
{{#example_decl Examples/ProgramsProofs/Arrays.lean ArrayFindHelper}}
```

すべての終了引数がこのものほど単純でないこともあります。ただし、関数の引数に基づいて各呼び出しで減少する式を識別する基本的な構造は、すべての停止性証明で発生します。時には、関数がなぜ終了するのかを理解するために創造性が必要であり、Leanは終了引数を受け入れるために追加の証明が必要なこともあります。


## 演習（Exercises）

* 配列に対する末尾再帰のアキュムレータパッシング関数と `termination_by` 句を使用して、配列に対する `ForM (Array α)` インスタンスを実装してください。

* 末尾再帰のアキュムレータパッシング関数を使用して、配列を反転する（逆順にする）関数を実装してください。この場合、`termination_by` 句は必要ありません。

* `Array.map`、`Array.find`、および `ForM` インスタンスを、恒等モナド内の `for ... in ...` ループを使用して再実装し、結果のコードを比較してください。

* 恒等モナド内の `for ... in ...` ループを使用して、配列の逆転を再実装してください。これを末尾再帰関数と比較してください。
