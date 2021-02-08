# UnityにもESLintがほしい！ ~ StyleCop.Analyzers ~

[`#Unity`, `#StyleCop.Analyzers`, `#ESLint`, `#Mac`, `#VSCode`]

## ESLint

> ESLintは、JavaScriptコードで見つかった問題のあるパターンを識別するための静的コード分析ツール

JavaScriptの実装をしている時に使用しているツールで、  
コードの悪い部分を指摘してくれる。

例

- `var` は、禁止！(no-varルール)
- この変数、使ってなくない？(no-unused-varsルール)
- インデントずれてるよ。

など、コードの品質として問題がある部分を、  
警告もしくは、エラーとして指摘してくれる。  
さらに、ルールによっては自動でなおしてくれる。

## StyleCop.Analyzers

Unityという違う環境下であってもあの温もりを忘れることができない僕らのためのツールがStyleCop.Analyzers。

StyleCop.Analyzersは、C#の静的コード分析ツール。  
Roslyn(ロズリン)をベースにつくられているらしい。

> .NETコンパイラプラットフォームは、C#及びVisual Basic .NETのフリーかつオープンソースのコンパイラ・コード解析APIである。Roslyn (ロズリン) の通称でも知られている。

こちらもESLintと同じように未使用の変数などを指摘してくれる。

## 要望

- **Mac**で動く。
- **VSCode**で動く。

## 導入

### Unity の準備

バージョン `2020.2` から導入されたこちらの機能を使用する。

[Roslyn analyzers and ruleset files](https://docs.unity3d.com/2020.2/Documentation/Manual/roslyn-analyzers.html)

ので、それ以上のバージョンをインストールし、プロジェクトを用意する。

この時、動作確認用にスクリプトファイルを一つ用意しておく。  
ここでは、こちらのサンプルを利用する。  
[RethrowError.cs](https://docs.unity3d.com/2020.2/Documentation/Manual/roslyn-analyzers.html)

### StyleCop.Analyzers を用意

StyleCop.Analyzers(他のパッケージもかなたぶん)は、  
必要な `.dll` ファイルをUnityに読み込ませることで導入できるため、  
NuGet(ニューゲット)から必要なパッケージをダウンロードする。

[StyleCop.Analyzers](https://www.nuget.org/packages/StyleCop.Analyzers/)

「Download package」を押すと `stylecop.analyzers.*.*.***.nupkg` ファイルがダウンロードされる。
  
次は、このダウンロードされたファイルから `.dll` ファイルを取り出す必要があるが、  
`.nupkg` の実態は、 `.zip` ファイルとのことで、   
拡張子を `.zip` に変更して解凍することで中身を見ることが可能。

※ この作業、Macだと成功したり失敗したりするため、Windowsで行うと確実。標準ではない別のツールを使うと上手くいくかもしれない。

解凍後、 `StyleCop.Analyzers.dll` を、 `Assets` 以下に置く。  
一旦、どこでも良いが、 `Assets/Modules` など、 `.dll` ファイルたちを置く専用の場所を用意した方が良いきっと。

### StyleCop.Analyzers の依存を解決

`StyleCop.Analyzers.dll` を `Assets` 以下に置くと、以下のようなエラーが発生する。

> Assembly 'Assets/Modules/StyleCop.Analyzers.dll' will not be loaded due to errors:

> Unable to resolve reference 'Microsoft.CodeAnalysis'. Is the assembly missing or incompatible with the current platform?
Reference validation can be disabled in the Plugin Inspector.

> Unable to resolve reference 'Microsoft.CodeAnalysis.CSharp'. Is the assembly missing or incompatible with the current platform?
Reference validation can be disabled in the Plugin Inspector.

> Unable to resolve reference 'System.Collections.Immutable'. Is the assembly missing or incompatible with the current platform?
Reference validation can be disabled in the Plugin Inspector.

これは...

- `Microsoft.CodeAnalysis` 
- `Microsoft.CodeAnalysis.CSharp`
- `System.Collections.Immutable`

「が見つからないんですけどどういうことですか？」というエラー。 

ということで、先ほどと同じようにNuGetからパッケージをダウンロード、 `.dll` ファイルを取り出して、同じように置く。

すると、また同じように「これがない」「あれがない」と怒られるので、相手が落ち着くまでしっかりと話を聞いてあげる。

※ 言われたファイルを置いても怒られる場合がある。その場合は、バージョンに互換性がない可能性があり、エラー文に書かれているバージョンのものをNuGetからダウンロードしてきてあげることで落ち着く。

今回は、上記に加え以下を置くことで落ち着いた。

- `System.Reflection.Metadata`

### RoslynAnalyzer ラベルを設定

まずは、現状の `.csproj` ファイルを確認してみる。  
「Unityの準備」をした時にスクリプトファイルを用意している場合、ルートに `Assembly-CSharp.csproj` が生成されているはず。  
生成されていない場合、簡単なので「Unity csproj」などで調べて対応する。

StyleCop.Analyzersを有効化するには、  
この `.csproj` ファイルに、以下の記述が存在する必要がある。

```xml
<Analyzer Include="Assets/Modules/StyleCop.Analyzers.dll" />
```

この1文を追記するだけなので、自身の手でファイルを書き換えたくなるが、 `.csproj` ファイルは、Unityの管理下にあり、
手動で追記したとしてもいつの間にか上書きされ、追記した部分が消えてしまう。

ここで使用するのが、バージョン `2020.2` の機能。
`StyleCop.Analyzers.dll` に対して `RoslynAnalyzer` ラベルをつけることで、Unityが追記してくれる。

もし、この機能を使わずに有効化したい場合は、 `.csproj` ファイルに、以下のような参照を追記する方法もある。

```xml
<PackageReference Include="StyleCop.Analyzers" Version="1.1.118">
  <PrivateAssets>all</PrivateAssets>
  <IncludeAssets>runtime; build; native; contentfiles; analyzers</IncludeAssets>
</PackageReference>
```

しかし、こちらも同じような問題で記述が上書きされてしまう。  
この問題の解決策として、  
`AssetPostprocessor.OnGeneratedCSProjectFiles` を使用して `.csproj` ファイルを書き換えるというものがある。

しかし、この方法は、MacだからなのかVSCodeだからなのかは、不明だが失敗した( `OnGeneratedCSProjectFiles` が呼ばれない)。

そのため、 `chokidar` で監視して上書きしてやろうと思った...ところにこのラベル機能が飛び込んできた。  
自身で仕組みを用意することは、大変。  
私は、素直にこのラベル機能に頼ることにした。

### 動作確認

ラベルを設定後、VSCodeでスクリプトファイルを見てみると、以下のようなエラーが表示される場合がある。

> The reference assemblies for .NETFramework,Version=v4.7.1 were not found. To resolve this, install the Developer Pack (SDK/Targeting Pack) for this framework version or retarget your application. You can download .NET Framework Developer Packs at https://aka.ms/msbuild/developerpacks

こちらは、正直よくわからない。わからないが、解決方法はある。

### VSCodeの設定

VSCode側にも設定が必要。

拡張機能は、以下が必要になる。

- `ms-dotnettools.csharp`

次に `settings.json` で以下の設定をすることで、前節のエラーを解消できる。  
この項目は、ユーザー側の設定でしか変更できないようだ。

```json
"omnisharp.useGlobalMono": "always"
```

次に、以下の設定を追記する。  
これは、VSCode上でAnalyzerを有効化するための設定。  
こちらは、ユーザー・ワークスペースどちらでも設定可能。

```json
"omnisharp.enableRoslynAnalyzers": true
```

設定後、VSCodeを再起動する。  
すると、 `RethrowError.cs` ファイルにいくつかの警告が表示されるようになる。

これで導入成功。

### コードを修正してみる

`RethrowError.cs` の10行目、

```C#
DoSomethingInteresting();
```

この部分に警告が出ているはず。
カーソルを合わせると...

> Prefix local calls with this [Assembly-CSharp]csharp(SA1101)

と、警告の説明文が表示される。  
今回は、「 `this` をつけてください」という警告。  
指摘通り、 `this` をつけると警告がなくなる。

```C#
this.DoSomethingInteresting();
```

### ルールのカスタマイズ

StyleCop.Analyzersは、ルールを自由にカスタマイズできる。  
先ほどの `this` についての警告も出さないようにすることが可能だ。  
とはいえ、これらのルールは、けっこう強い人たちがこうあるべきだと設定したものたちである(はずの)ため、  
基本的には、あまり変えないことをオススメする。

しかし、中には、無効化した方が良い場合もある。  
例えば、以下のようなルールがある。

> Documentation text should end with a period.

これは、コード中のドキュメントコメントの文末にピリオドを必要とするルール。  
気持ちはよくわかるが、私たちは日本人なので無効化したい。

ルールの設定には、 `.editorconfig` を使用する。

まずは、以下の拡張機能をインストールする。

- `editorconfig.editorconfig`

インストール後、 `settings.json` に以下を追記する。  
追記後、一度VSCodeを再起動する。

```json
"omnisharp.enableEditorConfigSupport": true
```

再起動後、ルートに `.editorconfig` ファイルをつくる。  
次にルールを記述...の前に、実際に警告を表示させる。

`RethrowError.cs` の4行目、

```C#
public class RethrowError : MonoBehaviour
```

の警告にカーソルを合わせると、  
「ドキュメントコメントを書いてください」と言われるので、  
以下のように対応してみる。

```C#
/// <summary>
/// コメント。
/// </summary>
public class RethrowError : MonoBehaviour
```

そうすると、次は、句点の後ろに警告が出る。  
これが目的のピリオドルール。  
ルールの無効化には、ルールのIDが必要となる。  
ルールのIDは、警告の説明文の末尾に書かれている。

> Documentation text should end with a period [Assembly-CSharp]csharp(SA1629)

このルールのIDは、 `SA1629` とのこと。  
このIDのルールを無効化する。  
`.editorconfig` に以下のように記述する。

```
root = true

[*.cs]
dotnet_diagnostic.SA1629.severity = none
```

そうすると、ピリオドルールの警告がなくなったはず。

今回は、ルールを無効化する `none` を指定したが、  
警告にする `warning` 、エラーにする `error` 、提案する `suggestion` を指定することが可能。

StyleCop.Analyzersのルールは、 `SA` から始まるIDを持つ。  
しかし、警告を見てみると違う文字から始まるIDも存在している。  
`IDE` や `CS` などがあるが、  
これらの違いは、誰が設定したルールなのかということ...のはず(違うかもしれない)。  
その為ルールによっては、違う誰かが設定した別のルールと競合している場合がある。  
こういう場合は、どちらか片方を無効化すると良い。

### AutoFix

慣れてくると保存時に自動で修正してほしいという欲も出てくる。  
これを実現するためには、 `StyleCop.Analyzers.dll` と一緒に入っていた `StyleCop.Analyzers.CodeFixes.dll` も導入することで可能。  
こちらの導入時も依存パッケージの不足を解決していく必要がある。  
今回は、以下を追加でダウンロードした。

- `Microsoft.CodeAnalysis.Workspaces`
- `System.Composition.AttributedModel`
- `System.Composition.Runtime`
- `System.Composition.TypedParts`
- `System.Composition.Hosting`

ダウンロード後、 `StyleCop.Analyzers.CodeFixes.dll` にも `RoslynAnalyzer` ラベルをつけ、  
`settings.json` に以下の設定を追記する。

```json
"editor.codeActionsOnSave": {
  "source.fixAll.csharp": true
}
```

追記後、 `RethrowError.cs` で試してみたいところだが...  
以下の状況では、失敗することを確認している。  
解決方法は、わかっていない。

- アクセス修飾子が明記されていない。
- `throw` を使用している。
- `this` をつけていない。

`RethrowError.cs` でこれらを対応した後、保存してみるとコードに変化が起きるはず。

上記の失敗する問題は、  
OmniSharpの「Fix all occurrences of a code issue within document」機能で「Fix all issues」を選択すると発生する。  
一つずつ解決する分には、発生せず、クイックフィックス( `⌘.` )も問題なく行える。

### ルールセット

以下のルールがある。

> The file header is missing or not located at the top of the file.

これは、ファイルヘッダーを求めるルール。  
しかし、正直なところ面倒なため、無効化してみる。

```
dotnet_diagnostic.SA1633.severity = none
```

これでVSCodeの警告が消えたはず。  
しかし...Unityのコンソールに警告が表示されている。

説明を飛ばしていたが、  
StyleCop.Analyzersの警告たちは、Unityのコンソールにも表示される。

実際に確認してみると、無効化したはずのファイルヘッダールールの警告が継続して表示されている。  
この原因は、わかっていない。わからないが、解決方法はある。

ルールセットファイルを用意することで解決できる。  
`Assets` 直下に `Default.ruleset` ファイルをつくり、以下を記述する。

```xml
<?xml version="1.0" encoding="utf-8"?>
<RuleSet Name="名前" Description="説明" ToolsVersion="1.0">
  <Rules AnalyzerId="StyleCop.Analyzers" RuleNamespace="StyleCop.Analyzers.DocumentationRules">
    <Rule Id="SA1633"  Action="None" />
  </Rules>
</RuleSet>

```

これでUnity側の警告も消えたはず。  
使い分けとしては、以下のようになる。

|表示|ルールファイル|
|---|---|
|VSCode|.editorconfig|
|Unity|.ruleset|

両方を管理するとなるとそこそこ面倒だが統一する方法は、わかっていない。

ちなみにルールセットについては、公式のものを参考にするとどんなルールがあるのかもわかりやすいのでおすすめ。

[公式のルールセットファイル](https://github.com/DotNetAnalyzers/StyleCopAnalyzers/blob/master/StyleCop.Analyzers/StyleCop.Analyzers.ruleset)

### Assembly Definition

ライブラリを使用する際、  
インポートしたソースコードにも警告を出してくれることに困ることがある。  
この場合は、 [Assembly Definition](https://docs.unity3d.com/ja/2020.2/Manual/ScriptCompilationAssemblyDefinitionFiles.html) を使用すると解決できる.

Assembly Definitionを簡単に説明すると、  
C#コードをグループ分けする機能で、これを使用することで特定のグループに属するC#コードにのみAnalyzerをかけることができる。

詳しい方法は、公式のドキュメントに書かれているので、  
そちらを確認した方が誤解が少ないと思う。

[Roslyn Analyzers のドキュメント](https://docs.unity3d.com/2020.2/Documentation/Manual/roslyn-analyzers.html)

この作業を後からする場合、一度 `RoslynAnalyzer` ラベルを外してから作業した方が良さそう。  
Assembly Definitionを分けたけど参照が切れない場合がある。

もし、この状況になった場合、  
`.csproj` ファイルを削除して再生成させるか、直接参照の記述を削除することで対応できる場合がある。  
直接参照の記述を削除するというのは、対象の `.csproj` ファイルの中から以下の記述を削除するということ。

```xml
<ItemGroup>
  <Analyzer Include="Assets/Modules/StyleCop.Analyzers.dll" />
  <Analyzer Include="Assets/Modules/StyleCop.Analyzers.CodeFixes.dll" />
</ItemGroup>
```

### Ignoreの方法

Assembly Definition以外にも、Analyzerを無効にする方法がある。

`.editorconfig` で以下のように記述するとAnalyzerは、指定したファイルを無視する。

```
[Assets/Scripts/Sample.cs]
generated_code = true
```

### 簡単に導入したい場合

以下の条件で問題ない場合、 `omnisharp.json` で導入する方法がある。

- Unity側での警告・エラー出力ができない。

こちらは、より簡単かつ、バージョンも自由。  
[VSCode の公式ドキュメント](https://code.visualstudio.com/docs/other/unity)にも書かれている方法。

また、「Fix all issues」で失敗するパターンが減ることも確認している。

[omnisharp.json の参考](https://www.strathweb.com/2019/04/roslyn-analyzers-in-code-fixes-in-omnisharp-and-vs-code/)

### 他の選択肢

- Microsoft.Unity.Analyzers
    - Unity固有のルールを追加・解析してくれる。
    - 併用すると良いかも。

### 解決したい項目

- 「Fix all issues」の安定化。
- ルールファイルを一つにする方法。
- 表面に現れないルールを表示する方法。
  - `dotnet_analyzer_diagnostic.severity = suggestion`
  - 一応、 ↑ のように書くことで表示できるが、全てのルールに対して設定する書き方なので、検討中。
  - `dotnet_analyzer_diagnostic.category-style.severity = suggestion`
  - ルールは、それぞれカテゴリーに属しているらしく、 ↑ のように特定のカテゴリーに対して、と書くことも可能。
  - [ドキュメント](https://docs.microsoft.com/ja-jp/visualstudio/code-quality/use-roslyn-analyzers?view=vs-2019)
- Unityのコンソールで謎のエラーが発生する時がある。
  - `Could not find a part of the path ... RoslynAnalysisRunner`
  - このエラーは、他でも結構発生しているよう。
  - 対応待ち？
