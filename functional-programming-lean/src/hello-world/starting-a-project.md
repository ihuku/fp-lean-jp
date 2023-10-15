# プロジェクトを開始する（Starting a Project）

Leanというプログラムがより本格的になるにつれて、実行可能ファイルが生成される事前コンパイラベースのワークフローがより魅力的になります。
他の言語と同様に、Leanには複数のファイルからなるパッケージを構築し依存関係を管理するためのツールがあります。
標準のLeanビルドツールは「Lake（「Lean Make」の略）」と呼ばれ、Leanで構成されています。
Leanには、効果を持つプログラムを書くための特別な目的の言語である「do」言語が含まれているように、Lakeにもビルドの構成を行うための特別な目的の言語が含まれています。
これらの言語は「組み込みドメイン固有言語（embedded domain-specific languages）」（または「ドメイン固有組み込み言語」、EDSLまたはDSELと略すこともあります）と呼ばれています。
これらは特定の目的に使用され、特定のサブドメインの概念に関連しており、一般的な汎用プログラミングには適していないことが一般的です。
これらは、他の言語の構文の内部で発生するため、「組み込み」されています。
LeanにはEDSLを作成するための豊富な機能が備わっていますが、それらはこの本の範囲外です。

## 最初のステップ（First steps）

Lakeを使用するプロジェクトを始めるには、すでに「greeting」という名前のファイルやディレクトリを含まないディレクトリで、次のコマンドを使用してください：`{{#command {first-lake} {lake} {lake new greeting} }}`。
これにより、`greeting`という名前のディレクトリが作成され、次のファイルが含まれます：

 * `Main.lean`はLeanコンパイラが「main」アクションを検索するファイルです。
 * `Greeting.lean`と`Greeting/Basic.lean`はプログラムのサポートライブラリの骨組みです。
 * `lakefile.lean`にはアプリケーションをビルドするために`lake`が必要とする構成が含まれています。
 * `lean-toolchain`にはプロジェクトで使用されるLeanの特定バージョンの識別子が含まれています。

さらに、`lake new`はプロジェクトをGitリポジトリとして初期化し、中間ビルド製品を無視するための`.gitignore`ファイルを構成します。
通常、アプリケーションの大部分のロジックはプログラムのライブラリのコレクションに含まれ、`Main.lean`にはこれらの部分をパースしてコマンドラインを実行し、中心のアプリケーションロジックを実行する小さなラッパーが含まれます。
既存のディレクトリでプロジェクトを作成する場合は、`lake new`の代わりに`lake init`を実行してください。

デフォルトでは、ライブラリファイルである `Greeting/Basic.lean` には単一の定義が含まれています：

```lean
{{#file_contents {lake} {first-lake/greeting/Greeting/Basic.lean} {first-lake/expected/Greeting/Basic.lean}}}
```

ライブラリファイル `Greeting.lean` は `Greeting/Basic.lean` をインポートしています：

```lean
{{#file_contents {lake} {first-lake/greeting/Greeting.lean} {first-lake/expected/Greeting.lean}}}
```

これは、`Greetings.lean` をインポートするファイルにとっても `Greetings/Basic.lean` で定義されているものが利用可能であることを意味します。`import` ステートメントでは、ドットはディスク上のディレクトリとして解釈されます。名前の周りにギロメ（«Greeting»）を配置することにより、通常はLeanの名前で許可されていないスペースや他の文字を含めることができ、また、`if` や `def` などの予約語を `«if»` や `«def»` と書いて通常の名前として使用できます。これにより、`lake new` に提供されたパッケージ名にそのような文字が含まれている場合の問題を防ぎます。

実行可能ソースである `Main.lean` には以下が含まれています：

```lean
{{#file_contents {lake} {first-lake/greeting/Main.lean} {first-lake/expected/Main.lean}}}
```

`Main.lean` は `Greetings.lean` と `Greetings.lean` は `Greetings/Basic.lean` をインポートするため、`hello` の定義は `main` で利用可能です。

パッケージをビルドするには、次のコマンドを実行します：`{{#command {first-lake/greeting} {lake} {lake build} }}`。
いくつかのビルドコマンドが表示された後、生成されたバイナリは `build/bin` に配置されます。
`{{#command {first-lake/greeting} {lake} {./build/bin/greeting} }}` を実行すると、`{{#command_out {lake} {./build/bin/greeting} }}` が表示されます。


## Lakefile（Lakefiles）

`lakefile.lean` は、配布用のLeanコードの一貫したコレクションである _パッケージ_ を記述します。これは、`npm`や`nuget`パッケージやRustのクレートに類似するものです。パッケージには任意の数のライブラリや実行可能ファイルを含めることができます。[Lakeのドキュメンテーション](https://github.com/leanprover/lake#readme)では、lakefile内の利用可能なオプションについて説明していますが、ここでまだ説明されていないLeanのいくつかの機能を使用しています。生成された `lakefile.lean` には以下が含まれます：

```lean
{{#file_contents {lake} {first-lake/greeting/lakefile.lean} {first-lake/expected/lakefile.lean}}}
```

この初期のLakefileには、次の3つの項目が含まれています：
 * `greeting` という名前の _パッケージ_ 宣言
 * `Greeting` という名前の _ライブラリ_ 宣言
 * `greeting` という名前の _実行可能ファイル_

これらの名前は、パッケージ名を選ぶ際にユーザーにより多くの自由を与えるためにギロメで囲まれています。

各Lakefileには正確に1つのパッケージが含まれますが、任意の数のライブラリや実行可能ファイルが含まれることがあります。さらに、Lakefileには _外部ライブラリ_（Leanで書かれていない結果の実行可能ファイルに静的リンクするためのライブラリ）、_カスタムターゲット_（ライブラリ/実行可能ファイルのタクソノミーに適合しないビルドターゲット）、_依存関係_（ローカルまたはリモートGitリポジトリからの他のLeanパッケージの宣言）、および _スクリプト_（基本的には `main` に似た `IO` アクション）が含まれることがあります。Lakefile内の項目によって、ソースファイルの場所、モジュール階層、およびコンパイラフラグなどが設定できます。一般的には、デフォルト値が妥当です。

ライブラリ、実行可能ファイル、およびカスタムターゲットはすべて _ターゲット_ と呼ばれます。デフォルトでは、`lake build` は `@[default_target]` で注釈付けされたターゲットをビルドします。この注釈は _属性_ で、Leanの宣言に関連付けることができるメタデータです。属性は、JavaのアノテーションやC#およびRustの属性に似ており、Lean全体で広く使用されています。`@[default_target]` で注釈付けされていないターゲットをビルドするには、`lake build`の後にターゲットの名前を引数として指定します。


## Libraries and Imports

## ライブラリとインポート

Leanのライブラリは、名前をインポートできるソースファイルの階層的に整理されたコレクションから成り立っており、これは _モジュール_ と呼ばれています。
デフォルトでは、ライブラリには名前と一致する単一のルートファイルがあります。
この場合、ライブラリ `Greeting` のルートファイルは `Greeting.lean` です。
`Main.lean` の最初の行、すなわち `import Greeting` は、`Main.lean` で `Greeting.lean` の内容を使用できるようにします。

ライブラリには、ディレクトリ `Greeting` を作成し、その中に追加のモジュールファイルを配置することで追加のモジュールファイルを追加できます。
これらの名前は、ディレクトリ区切り文字をドットに置き換えることでインポートできます。
たとえば、ファイル `Greeting/Smile.lean` を以下の内容で作成すると：
```lean
{{#file_contents {lake} {second-lake/greeting/Greeting/Smile.lean}}}
```
`Main.lean` は次のようにその定義を使用できます：
```lean
{{#file_contents {lake} {second-lake/greeting/Main.lean}}}
```

モジュール名の階層は、ネームスペースの階層とは切り離されています。
Leanでは、モジュールはコードの配布単位であり、ネームスペースはコードの組織単位です。
つまり、モジュール `Greeting.Smile` で定義された名前は自動的に対応するネームスペース `Greeting.Smile` にはありません。
モジュールは任意のネームスペースに名前を配置でき、それらをインポートするコードはネームスペースを `open` してもしなくても構いません。
`import` はソースファイルの内容を使用可能にするために使用され、`open` はネームスペースからの名前を接頭辞なしで現在のコンテキストで使用可能にします。
Lakefileでは、`import Lake` の行は `Lake` モジュールの内容を使用可能にし、`open Lake DSL` の行は `Lake` と `Lake.DSL` ネームスペースの内容を接頭辞なしで使用可能にします。
`Lake.DSL` は `Lake` を `Lake.DSL` として使用可能にすることによって開かれ、他のすべての名前と同様に `Lake` ネームスペース内のすべての名前が `DSL` として使用可能になります。
`Lake` モジュールは名前を `Lake` および `Lake.DSL` ネームスペースの両方に配置します。

ネームスペースは、明示的な接頭辞なしで一部の名前のみを使用可能にするよう _選択的に_ 開くことができます。
これは、所望の名前を括弧で囲んで書くことで行います。
たとえば、`Nat.toFloat` は自然数を `Float` に変換します。
`open Nat (toFloat)` を使用して、`toFloat` を `toFloat` として利用可能にすることができます。
