# Changelog

## Unreleased

- Python CLI開発、README・関連文書の執筆、YAML記述の3規約を独立した正本として収録。
- READMEとYAMLの規約をPythonから独立して適用できるように適用範囲を明確化。
- 作者のCLI prefix（`tkn-`）・設定namespace（`tkn`）・application IDの例・公開済みの参考リポジトリと参考READMEを、各規約の適用例として収録。
- 実環境パス・Vault固有schemaと旧日時移行規則は共通規約から分離。
- YAML記述規約で、`type` に限っていた引用符なしの規則を日付・日時・パス以外の文字列全般へ広げ、5章として独立。UUID・ハッシュなどの識別子の表記と、キー名・未設定値の表記など用途ごとに定める項目を追加。
- README執筆規約に、タイトル（H1）を `<表示名> — <副題>` の形式で書く規則を3.4として追加。表示名はリポジトリ名から作り、リポジトリ名・コマンド名は併記しない。従来の3.4は3.5へ繰り下げ。
- README・CHANGELOG・docsの役割、docs直下への配置、正本と時点を持つ記録の区別を継承。
- README執筆規約4.1で、公開リポジトリの利用者向けのインストール・更新を、clone不要の `uv tool install git+https://…` と `uv tool install --reinstall git+https://…` を基本とする形に変更。cloneからの `uv tool install .` は未公開時の基本手順か開発の説明に置く。
- Python CLI開発基本方針5.1を同じ方針に変更し、公開repositoryの通常のinstallation・更新をGitHub URLの形式にした。14章の受入条件に、公開後のGitHub URLからのinstallationを追加。
- Python CLI開発基本方針6.2で、application IDを「repository名を変更しても安易に変更しない」とした規則を改めた。application IDはGitHubのoriginのrepository名から作り、repository名を変更したときと、名前や役割とかけ離れた・誤解させるときは変更する。変更時はCHANGELOGに旧いdirectory・新しいdirectory・移し方を記載する。例の3行目を `tkn_genai_chat_note` に更新。
- Python CLI開発基本方針7章に、data・state・cacheの区分がXDG Base Directory Specification（0.8）に倣うことと、stateの基準（dataに置くほど重要でないもの）を追加。4.5・13.2とREADME執筆規約3.1で、評価・調査のレポートや実機検証結果を `.local/state` から `.local/data` へ移し、`.local/state` は実行log・run report・進捗やcheckpointなど、失っても作業の成果が残るものに限った。
- Python CLI開発基本方針4.5とREADME執筆規約3.1で、変更前バックアップの置き場を `.local/data` から `.local/backups/<datetime_work-id>/` に分けた。後で必要な資料と、変更を確認した後に削除できる控えを区別するため。data・backups・stateには同じ作業名を使う。
- 改行コード・文字コード規約（`docs/line-endings-and-encoding.md`）を追加。テキストファイルはLF・BOMなしのUTF-8を基本とし、BAT・CMD（`.bat`・`.cmd`）、PowerShell（`.ps1`・`.psm1`・`.psd1`）、AutoHotkey（`.ahk`）、WSH（`.vbs`・`.wsf`・WSH用の `.js`）、VBAのエクスポート（`.bas`・`.cls`・`.frm`）はCRLFとした。各形式の文字コード（BOM付きUTF-8・CP932など）、`.gitattributes` の例、編集時の確認、実機で確かめた環境を収録。PowerShellは版を問わずCRLFとし、Linux・macOSでshebangから起動するスクリプトだけLFとする。`.js` は既定でLFとし、WSH用だけをパスで指定する。
- Python CLI開発基本方針の3.3・4.2・14章とYAML記述規約の6章から、改行コード・文字コード規約を参照するようにした。標準リポジトリ構成に `.gitattributes` を追加。
- コードで描くモーショングラフィックス動画の制作ガイドライン（`docs/motion-graphics-video-production.md`）を追加。1分の紹介動画を3版作った経験から、品質を決めたもの、関門と採点、確かめる道具、時刻だけで描く作り方と決定性の落とし穴、場面のつなぎ・製品の画面・文字の見た目、読む時間の目安、書き出し（別のブラウザーで並べて撮る、動きのぶれ、BT.709の色の印）、音、参照の動画の渡し方、動画用の枠組みを入れるかの判断をまとめた。規約ではなく、実例1本で確かめた目安とした。
