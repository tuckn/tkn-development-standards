# 改行コード・文字コード規約

テキストファイルの改行コードは **LF** に揃えます。Windowsの実行環境やOfficeが読み込む形式（BAT・CMD、PowerShell、AutoHotkey、WSH、VBAのエクスポート）だけを **CRLF** とし、文字コードもその実行環境が読める形式にします。人や生成AIが編集しても、改行コード・文字コードの違いで動作や差分が変わらないようにするための共通規約です。

## 1. 位置付けと適用範囲

本書は、リポジトリで管理するテキストファイル（コード、文書、設定、テストデータ）に、実装言語を問わず適用します。人による編集と、プログラムや生成AIによる生成・編集の両方を対象とします。

改行コードと文字コードの規則は本書を正本とします。[YAML記述規約](yaml-style.md)と[Python CLI開発基本方針](python-cli-development.md)は、本書を参照します。

外部サービスの入出力や、既存の利用者が依存するファイルなど、別の改行コード・文字コードが求められる場合は、その仕様を優先します。例外は `.gitattributes` と対象プロジェクトの文書に記載します。

## 2. 基本はLF・BOMなしのUTF-8

- 新規のテキストファイルは、改行コードをLF、文字コードをBOMなしのUTF-8とします。
- 1つのファイルの中で改行コードを混在させません。最終行も改行で終えます。
- Markdown・YAML・JSON・Python・Node.jsやブラウザ用のJavaScript・TypeScript・シェルスクリプトなど、3章に挙げない形式は、すべてこの基本に従います。
- Linux・macOSで実行するシェルスクリプトや、shebang（`#!`）で起動するファイルは、必ずLFにします。shebang行の末尾にCRが残ると、インタープリター名の一部として扱われ、起動できません。

## 3. CRLFにする形式

Windowsの実行環境やOfficeが読み込む次の形式は、CRLFにします。文字コードは、各実行環境が読める形式にします（4章）。

| 形式 | 拡張子 | 文字コード | LFにしたときの確認結果 |
| --- | --- | --- | --- |
| BAT・CMD | `.bat`、`.cmd` | ASCIIを基本とし、日本語を含めるならCP932。BOMを付けない | ラベルの位置によって、`call`・`goto` がラベルを見つけられない |
| VBAのクラス・フォーム | `.cls`、`.frm` | ANSI（日本語環境ではCP932） | クラスモジュール・フォームにならず、ヘッダーがコードに残った標準モジュールとして読み込まれる |
| VBAの標準モジュール | `.bas` | ANSI（日本語環境ではCP932） | 読み込めた |
| PowerShell | `.ps1`、`.psm1`、`.psd1` | 非ASCII文字を含むならBOM付きUTF-8 | Windows PowerShell 5.1・PowerShell 7とも動いた。here-stringの改行がLFになる |
| AutoHotkey | `.ahk` | 非ASCII文字を含むならBOM付きUTF-8 | v2で動いた |
| WSH | `.vbs`、`.wsf`、WSH用の `.js` | ASCIIを基本とし、日本語を含めるならCP932 | 動いた |

VBAの `.frm` と対で書き出される `.frx` はバイナリファイルです。テキストとして変換しません。

### 3.1. CRLFにする理由

LFにして壊れたのは、BAT・CMDと、VBAの `.cls`・`.frm` です。この2つでは、CRLFが動作の条件です。BAT・CMDの不具合はラベルの位置によって起き、ラベルの位置を100通り変えた確認では、LFで1通りだけ失敗しました。少ない確認で動いても、LFを使いません。

PowerShell・AutoHotkey・WSHと、VBAの `.bas` は、LFでも動きました。それでもCRLFに揃えるのは、次の理由です。

- 拡張子だけで改行コードが決まり、ファイルごとに判断しなくて済みます。人と生成AIが、どちらの改行コードが正しいかを確かめ直す手戻りを防ぎます。
- ExcelのVBEなど、Windowsの標準ツールが書き出す形式と同じになります。書き出し・commit・読み込みを繰り返しても、改行コードだけの差分が出ません。
- PowerShellのhere-stringのように、ファイルの改行コードが実行結果の文字列に入る場合があります。CRLFに揃えると、Windowsのほかのツールに渡す出力が変わりません。

これらの形式が動かない原因になりやすいのは、改行コードより文字コードです（4章）。

### 3.2. PowerShellの版

Windows PowerShell 5.1用とPowerShell 7用のスクリプトは、同じ拡張子を使います。PowerShell 7もCRLFのスクリプトをそのまま実行できるため、版を問わずCRLFにします。

例外として、Linux・macOSでshebangから直接起動するPowerShell 7のスクリプトは、2章に従ってLFにします。対象のパスを `.gitattributes` で指定します。

### 3.3. `.js` の扱い

`.js` は、Node.jsやブラウザ用の既定どおりLFにします。WSH（JScript）用の `.js` は専用のフォルダに置き、そのパスを `.gitattributes` で指定してCRLFにします。拡張子だけで一律にCRLFにしません。

## 4. 文字コードの注意

エディターや生成AIのツールは、BOMなしのUTF-8で保存することが多くあります。3章の形式では、それが動かない原因になります。

| 実行環境 | 読み方と確認結果 | 守ること |
| --- | --- | --- |
| Windows PowerShell 5.1 | BOMなしのファイルをANSI（日本語環境ではCP932）として読む。BOMなしのUTF-8に日本語を含むと、LF・CRLFを問わず構文エラーになった | 非ASCII文字を含むファイルはBOM付きUTF-8にする。PowerShell 7はBOMの有無を問わず読める |
| AutoHotkey v1 | BOMなしのファイルをANSIとして読む（v2はUTF-8として読む） | 非ASCII文字を含むファイルはBOM付きUTF-8にする。v1・v2の両方で正しく読める |
| cmd.exe（BAT・CMD） | コンソールのコードページ（日本語環境ではCP932）で読む。UTF-8の日本語の行で構文エラーになり、BOMを付けると1行目（`@echo off` など）が壊れた | BOMを付けない。日本語を含めるならCP932にする |
| WSH（`.vbs`・`.js`） | UTF-8を読めない。BOMなしでは日本語の行で構文エラーになり、BOM付きではASCIIだけのファイルでも1行目でエラーになった | UTF-8にしない。ASCIIを基本とし、日本語を含めるならCP932にする |
| WSH（`.wsf`） | BOMなしのUTF-8では、日本語の行で構文エラーになった | CP932かBOM付きUTF-8にする |
| VBE（VBAの書き出し・読み込み） | ANSIで書き出し、ANSIとして読む。UTF-8のファイルは日本語が文字化けした | Excelが書き出したCP932のまま保つ |

- WSHはUTF-16LE（BOM付き）も読めます。ただし、`.gitattributes` で `text` を指定したUTF-16LEのファイルは、Gitの改行の変換で壊れます（5章）。そのため、WSHのファイルにUTF-16LEを使いません。
- CP932のファイルは、GitHubなどの差分表示で日本語が文字化けします。表示より、実行環境が読めることを優先します。

## 5. `.gitattributes` で固定する

Git for Windowsでは、`core.autocrlf` の設定によって、checkoutしたファイルの改行コードが環境ごとに変わります。リポジトリのrootに `.gitattributes` を置き、各自の設定に依存しないようにします。`.gitattributes` の `eol` の指定は、`core.autocrlf` より優先されます。

```gitattributes
# Text files: normalize in the repository and check out with LF
* text=auto eol=lf

# Windows-native scripts
*.bat  text eol=crlf
*.cmd  text eol=crlf
*.ps1  text eol=crlf
*.psm1 text eol=crlf
*.psd1 text eol=crlf
*.ahk  text eol=crlf
*.vbs  text eol=crlf
*.wsf  text eol=crlf

# JScript for WSH (other .js files stay LF)
wsh/**/*.js text eol=crlf

# VBA modules exported from Office
*.bas  text eol=crlf
*.cls  text eol=crlf
*.frm  text eol=crlf
*.frx  binary
```

- リポジトリにある形式の行だけを置きます。例えば、VBAのファイルがないリポジトリには `.bas`・`.cls`・`.frm` の行を置きません。LaTeXのクラスファイルなど、別の形式が同じ拡張子を使うためです。
- `wsh/` は例です。WSH用の `.js` を置くフォルダに合わせます。
- Gitは、リポジトリ（index）にはLFで保存し、checkoutのときに `eol` の改行コードにします。作業ツリーに書かれたファイルは、`git add` やcommitをしても変換されません。checkoutし直すまで、書かれたときの改行コードのまま実行されます。CRLFの形式を作成・編集したら、作業ツリーのファイルを確かめます（6章）。
- `.gitattributes` を追加・変更したら、`git add --renormalize .` で既存のファイルをindexに反映し、改行コードだけの変更として、内容の変更と別にcommitします。
- UTF-16LEのファイルに `text` を指定すると、改行の変換でファイルが壊れます。UTF-16LEのファイルを置く場合は、そのパスに `binary` を指定します。
- エディターに同じ規則を伝える場合は、`.editorconfig` の `end_of_line`・`charset` も使えます（任意）。生成AIのツールが `.editorconfig` を読むとは限らないため、6章の確認を省きません。

## 6. 作成・編集するときの確認

生成AIを含め、ファイルを作成・編集するときは次を守ります。

- 編集の前に、対象ファイルの改行コードと文字コード（BOMの有無を含む）を確かめ、編集の後も同じであることを確かめます。
- 規約と違う既存ファイルでも、ほかの目的の編集のついでに改行コード・文字コードを変えません。変える場合は、それだけを目的とする変更にし、内容の変更と別にcommitします。そのファイルに依存する利用者や処理への影響も確かめます。
- `git ls-files --eol <path>` で、index（`i/`）・作業ツリー（`w/`）・`.gitattributes` の指定（`attr/`）の改行コードを確かめられます。`w/` が `attr/` の `eol` と違う場合は、作業ツリーのファイルを直します。
- UTF-8として読み書きするツールで、CP932などのファイルを直接編集しません。日本語が置き換わったまま保存されます。文字コードを指定して読み書きする方法（例：Pythonの `open(path, encoding="cp932", newline="")`）で編集します。
- 3章の形式を作成・変更したら、対象の実行環境で一度実行するか読み込み、動作を確かめます。確かめられなかった場合は、作業結果にその旨を書きます。

## 7. 確認した環境

3章・4章の確認結果は、2026-11-08に、各形式のファイルを改行コードと文字コードを変えて作り、次の環境で実行・読み込みして得たものです。

- Windows 11 Pro（10.0.26300）、システムのコードページ932（日本語）
- cmd.exe 10.0.26100、Windows PowerShell 5.1.26100、PowerShell 7.6.9
- AutoHotkey 2.0.2
- Windows Script Host 10.0（cscript）
- Excel 16.0（build 20527）。VBAの読み込みと書き出しは、COMの `VBComponents.Import`・`Export` で行いました。
- Git 2.54.0（Git for Windows）

AutoHotkey v1の文字コードの扱いは、公式文書の[Scripts（Script File Codepage）](https://www.autohotkey.com/docs/v1/Scripts.htm)によります。v1の動作は確認していません。Linux・macOSでのshebangの扱いも、この環境では確認していません（Git Bashでは、CRLFのshebang行でも起動しました）。
