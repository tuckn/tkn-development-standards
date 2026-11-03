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
