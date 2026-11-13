# スクリプトの言語の選び方

新しくスクリプト・CLIを作るとき、どの言語で書くかを決める順番を定めます。
人や生成AIが、同じ種類の処理を作るたびに違う言語を選ばないようにするための指針です。
まだ決まっていない点は、9章にまとめています。

## 1. 位置付けと適用範囲

- 本書は、新しく作るスクリプト・CLI・補助スクリプトの言語を選ぶときと、既存のものを別の言語で書き直すかを決めるときに使います。
- 選んだ言語での書き方は、それぞれの規約に従います。
  Pythonは [Python CLI開発基本方針](python-cli-development.md)、改行コード・文字コードは [改行コード・文字コード規約](line-endings-and-encoding.md) に従います。
- 対象リポジトリの指示や個別の依頼が別の言語を求めるときは、そちらを優先します。
  本書と違う言語を選んだときは、理由をREADMEか作業の記録に残します。
- 既存のスクリプトを、本書だけを理由に別の言語へ書き直しません（8章）。

## 2. 選ぶ順番

次の順に当てはめ、最初に当たったもので決めます。

1. 動く場所が言語を決めているなら、その言語にします（2.1）。
2. 既存の共通処理を使うなら、その処理の言語に合わせます（2.2）。
3. それ以外は、Python（uv）を基本にします（3章）。
   PowerShell（4章）・Node.js（5章）は、それぞれの条件に当たるときだけ使います。

処理の種類（Windowsの操作、Web、Officeなど）より先に、動く場所と既存の処理を見ます。
処理の種類は、たいていどの言語でも書けます。
動く場所と既存の処理は、言語を選べないことが多いためです。

### 2.1. 動く場所が決める言語

| 動く場所 | 言語 |
| --- | --- |
| Obsidian・ブラウザー・Electronアプリの中（プラグインから呼ぶスクリプト、userscript、拡張機能） | JavaScript |
| Excel・WordなどOfficeの中（ブックや文書に付けるマクロ、ボタン） | VBA |
| 常駐してキー入力を受け、ウィンドウを操作するもの（ホットキー、ほかのアプリへのキーの送信） | AutoHotkey |
| ダブルクリック・ショートカット・タスクからの起動口 | BAT・CMD（6章） |

Officeを外から操作するもの（ブックを開いて値を読む、保存し直すなど）は、Officeの中で動くものではないため、この表に当たりません。
3章・4章で選びます。

### 2.2. 既存の共通処理に合わせる

- 使いたい処理（解析・整形・検証など）が、すでにある言語で書かれていれば、新しいスクリプトもその言語で書きます。
- 同じ処理を別の言語で書き直して、2つ持ちません。
  片方だけを直すと、2つの結果がずれるためです。
- 例：アプリ（Obsidianなど）のプラグインの中とCLIの両方で、同じ整形処理を使うなら、CLIもNode.jsで書きます。
- 新しいスクリプトの中身のほとんどが別の処理で、共通処理を少し使うだけなら、共通処理をCLIとして呼び出してもかまいません（7章）。

## 3. Pythonを基本にする

2.1・2.2に当たらないときは、Pythonで書きます。

- 標準ライブラリで足りる範囲が広く、ファイル・パス、JSON・CSV、SQLite、ZIP、日時、外部コマンドの起動を、外部パッケージなしで扱えます。
- uvで、Pythonの版と外部パッケージを固定して動かせます。
- 繰り返し使うCLIには [Python CLI開発基本方針](python-cli-development.md) があり、設定・ログ・テスト・配布の決め方をそろえられます。

外部パッケージが要ることだけを理由に、ほかの言語へ移しません。
uvでは、1ファイルのスクリプトでも、ファイルの先頭に使うパッケージを書けます（inline script metadata。PEP 723）。
`uv run <script>.py` で実行すると、uvがそのパッケージを入れた環境を用意して動かします。

```python
# /// script
# requires-python = ">=3.11"
# dependencies = ["openpyxl"]
# ///
```

外部パッケージを避けるべきなのは、動かす場所でuvやPythonを前提にできないときです（4章）。

Officeを外から操作するときは、次のように選びます。

- 値の読み書きだけなら、Officeを起動しないパッケージ（Excelならopenpyxlなど）を使います。
- 書式・フィルター・数式などを保つためにOfficeそのものに操作させるなら、COMを使います。
  Pythonではpywin32で、PowerShellでは標準の機能で呼べます。
  処理が短く、中身がCOMの呼び出しだけなら、PowerShellでもかまいません（4章）。

## 4. PowerShellにするとき

次のどれかに当たるときは、PowerShellで書きます。

- 中身のほとんどがWindowsの機能を呼ぶことで、処理が短い。
  例：タスクスケジューラへの登録、レジストリ、サービス、ショートカット、ファイルのアクセス権、アプリの起動、Officeを開いて保存し直すだけのCOMの操作。
- uvやPythonが入っていない、または入れられないPCで動かす。
  例：PCの初期設定、uvやPythonを入れるスクリプトそのもの、ほかの人のPC。

次のようになったら、Pythonに移すか、処理の中心をPythonにしてPowerShellの部分を呼び出す形（7章）にします。

- データの変換・集計や、条件の分岐が増えた。
- テストで確かめたい処理ができた。
- 100行を超えた（目安）。

PowerShellは、Windows PowerShell 5.1とPowerShell 7で動きが違うところがあります。
文字コードや、外部コマンドが失敗したときの扱いにも癖があります。
長くなるほど、こうした見落としが入りやすくなるためです。

PowerShellで書くときは、次のことを守ります。

- 「インストールなしで動く」ことを理由に選ぶなら、Windowsに初めから入っているWindows PowerShell 5.1で動くように書きます。
  PowerShell 7は、5.1とは別に入れる必要があります。
- それ以外は、PowerShell 7を前提にしてかまいません。
  PowerShell 7が入っているPCでだけ動かすものまで5.1に合わせると、2つの版の違いを避ける手間だけが増えるためです。
  そのときは、スクリプトの先頭に `#Requires -Version 7` を書き、READMEにも要る版を書きます。
  Windows PowerShell 5.1で実行すると、`#Requires` の行で止まり、要る版が表示されます。
- 改行コードはCRLFにし、日本語などASCII以外の文字を含むならBOM付きUTF-8にします（[改行コード・文字コード規約](line-endings-and-encoding.md)の3章）。

## 5. Node.jsにするとき

次のどれかに当たるときは、Node.jsで書きます。

- 2.1・2.2に当たる。
  たとえば、ブラウザーやアプリの中で動くJavaScriptと同じ処理を、CLIでも使う。
- 使いたいライブラリがnpmでしか配られておらず、プログラムの中から呼ぶ必要がある。
  CLIとして配られている道具（Mermaid CLIなど）を呼ぶだけなら、どの言語からでも呼べるため、この条件に当たりません。

次のことだけでは、Node.jsにしません。

- HTMLを書き出す、Webページを取得する。
  Pythonでも書けます。
  ブラウザーの中で動かすときや、ブラウザーと同じコードを使うときに、2.1・2.2として選びます。
- 参考にした資料・サンプルコードがNode.jsで書かれている。
  生成AIは、サンプルを別の言語に移せます。
  ライブラリそのものがnpmにしかないときは、上の条件に当たります。

Node.jsで書くときは、次のことを守ります。

- モジュールの方式を拡張子で示します。
  `require()` を使うCommonJSは `.cjs`、`import` を使うES Modulesは `.mjs` にします。
  `.js` は、近くの `package.json` の `"type"` などによって、どちらとして読まれるかが変わるためです。
  動く場所（アプリのプラグインなど）が `.js` を求めるときは、それに従います。
- 外部パッケージを使うなら、`package.json` とlockfile（`package-lock.json` など）をcommitし、`node_modules` はcommitしません。
  `node_modules` を置けない場所（同期するフォルダなど）では、どこに置くかをREADMEに書きます。
- 改行コードはLFにします（[改行コード・文字コード規約](line-endings-and-encoding.md)の3.3）。

## 6. ほかの形式

| 形式 | 使うもの | 使わないもの |
| --- | --- | --- |
| BAT・CMD | ダブルクリック・ショートカット・タスクからの起動口。中では別のスクリプトを呼ぶだけにする | 処理の中身。分岐や繰り返しが要るなら、呼び出す先のスクリプトに書く |
| AutoHotkey | 常駐してキー入力を受け、ウィンドウを操作するもの（2.1） | ファイルやデータの処理 |
| VBA | Officeの中で動くもの（2.1） | Officeを外から操作するもの（3章・4章） |
| シェルスクリプト（bash） | Linux・macOS・CIで動かすもの | Windowsだけで動かすもの |
| WSH（VBScript・JScript） | 既存のものを直すときだけ | 新しく作るもの |

WSHで新しく書かないのは、次の理由によります。

- VBScriptは、Microsoftが非推奨とし、Windowsから段階的に取り除くと告知しています（10章）。
- WSHの `.vbs`・`.js` はUTF-8のファイルを読めないなど、文字コードの制約が多くあります（[改行コード・文字コード規約](line-endings-and-encoding.md)の4章）。
- できることは、PowerShellやPythonで置き換えられます。

## 7. 1つの処理で複数の言語を使うとき

- 中心にする言語を1つ決め、ほかの言語は、呼び出される薄い部分にします。
- 言語の間は、外部コマンドの呼び出しでつなぎます。
  受け渡すのは、引数・標準入出力（構造があるならJSON）・終了コードにします。
  ほかの言語のコードを直接読み込む仕組み（Pythonの中でJavaScriptを動かすなど）は使いません。
- 呼び出す側は、呼び出した先の終了コードを確かめ、0でなければ止めます。
- READMEに、要る実行環境（Python・Node.js・PowerShellの版）を書きます。

例：Officeの操作だけをPowerShellで書き、Pythonから `subprocess` で呼ぶ。
Node.jsで書いた共通の整形処理を、CLIとしてPythonから呼ぶ。

## 8. 既存のスクリプト

本書だけを理由に、既存のスクリプトを別の言語へ書き直しません。
次のときに、2章の順番で言語を選び直します。

- 大幅に作り直すとき。[Python CLI開発基本方針](python-cli-development.md)の2.2で、CLIへの移行を考えるときを含みます。
- PowerShellのスクリプトが、4章の「Pythonに移す」に当たったとき。
- 同じ処理を、2つの言語で持ってしまったとき（2.2）。

## 9. まだ決まっていない点

次の点は、実例で確かめてから決めます。

- PowerShellからPythonへ移す目安（100行）が合っているか。
- Node.jsで外部パッケージを使うとき、`node_modules` の置き場をどう決めるか。
- TypeScriptを使うか。使うなら、どの場面で使うか。

## 10. 根拠

2026-11-13に、Windows 11（Node.js v24.15.0、uv 0.11.8、Python 3.13.13、Windows PowerShell 5.1.26100、PowerShell 7.6.9）で確かめました。

- Node.jsの拡張子：`"type": "module"` の `package.json` があるフォルダで、`require()` を使う `.js` は `ReferenceError: require is not defined in ES module scope` で止まり、同じ内容の `.cjs` は動きました。`import` を使う `.mjs` も動きました。
  Node.jsのドキュメント [Determining module system](https://nodejs.org/api/packages.html#determining-module-system) も、`.cjs` は常にCommonJS、`.mjs` は常にES Modulesとし、`.js` は `package.json` の `"type"` などで決まる、としています。
- uvのinline script metadata：3章の例の先頭を書いたスクリプトを `uv run` で実行すると、uvがopenpyxl 3.1.5を入れた環境で動かしました。
  uvのドキュメント [Declaring script dependencies](https://docs.astral.sh/uv/guides/scripts/#declaring-script-dependencies) と [PEP 723](https://peps.python.org/pep-0723/) に、この書き方があります。
- PowerShellの版：このPCでは、Windows PowerShell 5.1（`powershell.exe`）とPowerShell 7（`pwsh.exe`）が別々に入っています。
  先頭に `#Requires -Version 7` を書いたスクリプトは、Windows PowerShell 5.1では「Windows PowerShell 7.0を要る」という旨のエラーで止まり、PowerShell 7では動きました。
  Microsoftのドキュメント [Differences between Windows PowerShell 5.1 and PowerShell 7.x](https://learn.microsoft.com/powershell/scripting/whats-new/differences-from-windows-powershell) に、2つの版の違いがまとめられています。
- VBScript：Microsoftは、2024年5月の [VBScript deprecation: Timelines and next steps](https://techcommunity.microsoft.com/blog/windows-itpro-blog/vbscript-deprecation-timelines-and-next-steps/4148301) で、Windows 11 24H2からVBScriptをオンデマンド機能（初めは有効）にし、2027年ごろに既定で無効にし、その後のWindowsから取り除く、と告知しています。
