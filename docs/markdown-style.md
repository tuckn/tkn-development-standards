# Markdown記述規約

見出し・リスト・太字の使い分けを定めます。
人と生成AIが、リポジトリの文書やエディターで書くノートの構造を、同じ考え方で組み立てるための共通規約です。

## 1. 位置付けと適用範囲

本書は、人と生成AIが書くMarkdown（README・CHANGELOG・docs、Obsidianなどのエディターで書くノート）に、実装言語を問わず適用します。
記法そのものは、CommonMarkとGitHub Flavored Markdown（GFM）に従います。
CommonMark・GFM・Obsidianが定めているのは記法で、見出しにするかリストにするかといった使い分けは定めていません。
本書は、その使い分けを、markdownlintのルールと、主なパーサー・エディターの動き（5章）を根拠に定めます。

READMEの章構成と見出しの名前の付け方は [CLIリポジトリREADME執筆規約](readme-writing.md) の3.5に、Markdownのソースの読みやすさ（空行、1文1行など）は同じく5.3に従います。

## 2. 見出し・リスト・太字の役割

| 書き方 | 示すもの | エディターで畳める | アウトラインに出る・リンクの先になる |
| --- | --- | --- | --- |
| 見出し（`#`〜`######`） | 文書の階層（節） | はい | はい |
| リスト（入れ子を含む） | 文書の階層（項目と、その下の項目） | はい | いいえ |
| 太字 | 文の中の要点（強調） | いいえ | いいえ |

- 見出しとリストは、どちらも文書の階層を示します。
- Obsidianなどのエディターは、見出しの節も、子の項目を持つリストの項目も畳めます。
- 見出しは、さらにアウトライン（目次）に出て、リンクの先になります（GitHubの見出しのアンカー、Obsidianの `[[ノート#見出し]]`）。
- 太字は強調で、階層を示しません。

## 3. 見出しにするかを決める

- 「そこへリンクしたいか」「アウトラインからそこへ飛びたいか」を目安にします。
  どちらかに当たる節を、見出しにします。
- どちらにも当たらないもの（数個の項目をまとめる分類の印など）は、見出しにせず、リストの項目にして、中身をその下に入れ子にします。

  ```markdown
  - **Background**:
    - <内容>
    - <内容>
  ```

- 小さなまとまりまで見出しにすると、中身より見出しのほうが重くなります。
  アウトラインも長くなり、探したい節が見つけにくくなります。

## 4. 太字の1行を見出しの代わりにしない

- 節を分けるのに、太字だけの1行を使いません。
  道具が文書の構造を読み取れなくなるためです（markdownlintのMD036）。
  見出しにするか、3章のようにリストの項目にします。
- 太字のラベルの次の行からリストを書く形（`**Background**:` の直後に `- …`）も使いません。
  - 空行がないと、パーサーによってはリストとして読まれません。
    pandocのMarkdownは、ラベルとリストを1つの段落として読みます（5章）。
    kramdownなども、前後に空行のないリストを読みません（markdownlintのMD032）。
  - 空行を入れると、どの項目がどのラベルの下かは並び順でしか分からず、エディターでラベルごとに畳めません。
- MD036は、末尾が `:` などの句読点で終わる段落を、既定で対象外にします。
  lintを通っても、太字の1行が見出しの代わりになっていれば、この章に当たります。

## 5. 根拠

2026-11-13に確かめました。

- markdownlintの [MD036（no-emphasis-as-heading）](https://github.com/DavidAnson/markdownlint/blob/main/doc/md036.md)：節を分けるのに強調を使うと、道具が文書の構造を推し量れない、としています。
  既定の `punctuation` は `.,;:!?。，；：！？` です。
- markdownlintの [MD032（blanks-around-lists）](https://github.com/DavidAnson/markdownlint/blob/main/doc/md032.md)：kramdownなど一部のパーサーは、前後に空行のないリストを読まない、としています。
- pandoc 3.9.0.2：`**Background**:` の次の行に `- item one` を書いた入力は、`-f markdown` では `<p><strong>Background</strong>: - item one …</p>` という1つの段落になり、`-f gfm` ではリストになりました。
  pandocのMarkdownでは、拡張 `lists_without_preceding_blankline` が既定で無効です。
- Obsidianのヘルプ [Folding](https://obsidian.md/help/Editing+and+formatting/Folding)：見出しと、字下げしたリストを畳めます。
- [Google Markdown style guide](https://google.github.io/styleguide/docguide/style.html)：太字を見出しの代わりにすることには触れていません。
  リストと小見出しで同じ情報を示せることがある、としています。
