# Python CLI開発基本方針

## 1. この文書の目的

この文書は、生成AIに新しいスクリプトの作成または既存スクリプトの大幅な再設計を
依頼するときの基本方針を定義する。

主な対象は、Windows PCで日常的または定期的に使用するPython CLI、
特にファイルの収集、変換、検証、要約、同期などを行うデータパイプラインである。
Windowsを主な実行環境としつつ、OS固有機能を必要としない部分はLinuxでも
動作しやすい構成にする。

この文書を生成AIへ渡した場合、生成AIは個別依頼に明記された要件と本仕様を
組み合わせて設計・実装する。個別依頼と本仕様が矛盾する場合は個別依頼を優先し、
逸脱内容と理由を作業結果に明記する。

## 2. 適用範囲と適用の強さ

### 2.1 主な適用対象

- 独立したGitリポジトリとして管理するPythonアプリケーション
- `uv`で開発、依存関係管理、インストールを行うCLI
- ローカルファイルを入力または出力とするデータパイプライン
- Windows Task Schedulerやcronなどから定期実行できる処理
- 生成AIや外部CLIを一部のstageで呼び出す処理

### 2.2 小規模スクリプトへの適用

一度だけ使う短い調査コードや、数十行程度の補助スクリプトに、独立パッケージ、
補助言語のREADME、設定階層などを一律に要求しない。

ただし、次のいずれかに該当した時点で、本仕様に沿ったCLIアプリケーションへの
移行を検討する。

- 繰り返し使用する
- 定期実行する
- 他の作業やデータが依存する
- 設定値や入力元が複数ある
- ファイルの移動、削除、上書きを行う
- 処理結果の再現、監査、障害復旧が必要になる

### 2.3 規範用語

- **必須**: 原則として満たす。満たせない場合は理由を明記する。
- **原則**: 通常は採用する。対象の性質に応じた例外を認める。
- **任意**: 必要な場合に採用する。

## 3. 設計原則

### 3.1 CLIを正式な利用インターフェースにする

- アプリケーションは、原則としてインストール可能なCLIとして提供する。
- 日常利用では、リポジトリ内の`.py`ファイルやPython実行ファイルの場所を
  直接指定しなくても実行できるようにする。
- CLI名、command、option、終了コード、標準出力の形式を公開インターフェースとして
  扱い、変更時は後方互換性または移行方法を検討する。
- CLIの処理は薄く保ち、設定解決、business logic、I/O、validationを
  import可能なmoduleへ分離する。

### 3.2 安全性と再現性を既定にする

- 同じ入力と設定から、同じ意味の出力を再生成できるようにする。
- 同じ処理を再実行しても、不要な重複や破損を発生させない。
- 変更を伴う処理は、可能な限り事前確認、検証、原子的な確定を行う。
- 元データ、処理済みデータ、実行状態、cache、一時ファイルを役割ごとに分離する。
- 破壊的な動作を、曖昧な既定値や暗黙の副作用にしない。

### 3.3 Windowsを主対象とし、移植しやすくする

- パス操作には`pathlib.Path`を使用し、文字列連結や区切り文字の決め打ちを避ける。
- home directoryは`Path.home()`、一時領域は`tempfile`などの標準機能で解決する。
- ファイルの読み書きは、原則としてUTF-8を明示する。
- subprocessは引数のlistで起動し、shell固有のquoteやcommand chainへの依存を避ける。
- Windows固有機能が必要な場合は、その境界をmoduleまたは補助スクリプトへ分離し、
  READMEに対応OSと代替手段を記載する。
- WindowsとLinuxで意味が変わるファイル名、大文字小文字、改行、timezone、
  permissionの差をテストまたは設計レビューで考慮する。

### 3.4 依存関係と複雑性を最小化する

- Python標準ライブラリで十分な機能に、不要な外部dependencyを追加しない。
- 外部dependencyを追加する場合は、用途、保守状況、ライセンス、
  platform compatibilityを確認する。
- application固有の処理と、外部service、生成AI、OS固有機能のadapterを分離する。
- 将来使うかもしれない機能を先回りして実装せず、現在の要件を満たす最小の構成から
  始める。

## 4. 標準技術構成

### 4.1 Pythonとuv

- Python 3.11以上を基本とし、実際の最低versionを`pyproject.toml`へ明記する。
- project管理、virtual environment、dependency lock、実行、buildには`uv`を使う。
- `pyproject.toml`と`uv.lock`をversion controlへ含める。
- packageは`src` layoutにする。
- CLI entry pointは`pyproject.toml`の`[project.scripts]`で定義する。
- build backendは、特別な要件がなければHatchlingなどの標準的で小さなbackendを
  使用する。

### 4.2 標準リポジトリ構成

新規リポジトリは、原則として次の構成にする。

```text
<repository-root>/
├── src/
│   └── <python_package>/
│       ├── __init__.py
│       ├── __main__.py
│       ├── cli.py
│       ├── config.py
│       ├── resources/
│       │   └── config.example.yaml
│       └── ...
├── tests/
│   ├── fixtures/
│   └── ...
├── scripts/
├── docs/
├── .gitignore
├── LICENSE
├── pyproject.toml
├── README.md
├── CHANGELOG.md
└── uv.lock
```

- READMEの言語は11.1に従う。`README.md`は英語を基本とし、主要な対象ユーザーが
  日本語話者であることが明確な場合は日本語にできる。補助言語版の`README_ja.md`または
  `README_en.md`は必要に応じて追加する。
- `<repository-name>`は、Git repositoryの名前である。
- `src/`はimport対象のcodeをrepository rootから分けるための置き場であり、
  `<python_package>`は`import <python_package>`で使うPython package名である。
  repository名、配布名、CLI名と一致する必要はない。codeを`src/`直下に並べて
  `<python_package>/`の代わりにはしない。
- `<python_package>`とapplicationの内部識別子には、原則として
  `snake_case`を使用する。
- CLIの実行名は、shellで入力しやすいkebab-caseを原則とする。
- 複数のCLIをまとめてTab補完できるよう、所有者や製品群の共通prefixを採用してよい。
  prefixの具体値は対象projectで定める。
- 既存の公開CLI名に利用者や自動処理が依存している場合は、互換性と移行costを評価し、
  既存名を維持する例外を認める。
- `scripts/`は、Task Scheduler登録など、main CLIへ含める必要がない
  platform固有の補助処理がある場合だけ使用する。
- `docs/`は詳細文書が必要な場合に使い、内容が分かるfile名で直下へ置くことを基本にする。
  文書の役割、正本、subfolderへ分ける判断は11.6に従う。空の分類folderは作らない。
- runtime data、大きなfixture、credential、実設定はrepositoryへ含めない。

### 4.3 責務の分離

規模に応じて、少なくとも次の責務を分離する。

- `cli`: 引数解析、command dispatch、終了コード
- `config`: default、設定ファイル、CLI optionの統合とvalidation
- `paths`: application directory、cache、一時領域の解決
- `models`または`schema`: 入出力とmanifestの構造
- `pipeline`または用途別module: 処理stageとbusiness logic
- `io`: 原子的書き込み、hash、serializationなどの共通I/O
- `validation`: 入力、出力、整合性の検証
- 外部連携adapter: 生成AI、外部CLI、Web API、OS固有機能

小規模なprojectでは、空のmoduleを形式的に増やさず、責務が増えた時点で分割する。

### 4.4 非code resourceの分離と所有権

- 利用者または開発者が内容を読んで編集するYAML、JSON、Markdown、template、
  promptなどは、原則としてPython codeへ長い文字列として埋め込まず、独立したfileへ
  分離する。
- Pythonの処理を変更せずに内容だけを更新できるMarkdownやJSONなどは、codeの変更を
  必要としない外部resourceとして管理する。内容の改訂と処理logicの変更を分離し、
  resource file単体の差分としてreviewできる構成にする。
- 少なくとも、`config.example.yaml`、生成AIへのsemantic instruction、structured
  output schema、最終出力template、長い定型文は外出しを検討する。
- Python codeは、resourceの読込、validation、version管理、control flow、安全なI/O、
  error処理を担当する。外部fileへ分離した内容をPython側へ重複して定義しない。
- resourceに定義した見出し、field、section順、semantic instructionなどを、Python codeの
  条件分岐や文字列定数として独立にhard-codeしない。code側のvalidationが必要な場合も、
  resourceから読み込んだ契約を使用し、同じ内容を二重管理しない。
- 外出しは、利用者による任意のcustomizationを意味しない。applicationが品質と互換性を
  管理するresourceは、application-ownedなpackage resourceとして扱ってよい。
- prompt、output schema、Markdown templateなど、相互に依存して1つの出力契約を
  構成するresourceは、profile directoryへまとめる。言語や用途ごとにprofileを追加する
  場合も、個別fileを無関係な設定として扱わない。
- resourceのID、version、hashを必要に応じて検証・記録し、prompt、schema、template、
  rendererの変更影響を一体としてtestする。
- installed applicationが実行時に必要とするresourceは`src/<python_package>/`以下へ置き、
  wheelとsource distributionへ収録する。repository rootだけに存在するfileへ実行時に
  依存しない。
- 同一resourceの手編集可能なcopyを複数置かない。やむを得ず生成物または配布用copyを
  持つ場合は、source of truthを1つに定め、buildまたはtestで一致を保証する。
- 短い定数、型、algorithm、validation ruleまで形式的に外出ししない。codeとdataの
  どちらが責務を明確にできるかで判断する。

### 4.5 ローカルに保持する開発資料と一時ファイル

- Gitへcommitしないが継続して保持する開発資料は、repository rootの`.local/`へ置く。
  評価・調査レポート、実機検証結果、応答記録、正解メモ、再現用の補助scriptなどを対象とする。
- 再開に必要な作業資料・保持する生成物・変更前バックアップは `.local/data/<datetime_work-id>/`、
  レポート・ログ・進捗や再開用stateは `.local/state/<datetime_work-id>/` へ置く。
  作業開始時のローカル日時を先頭に付ける（例: `20261012T130200_vision-evaluation`）。
  dataとstateには同じ名前を使い、再開時もそのフォルダを使用する。
  作業名は既定でkebab-case、Pythonのリポジトリ名を使う場合はsnake_case、PowerShell・Windows系のフォルダ名はUpperCamelCaseとする。
- 開発支援ツールの再取得・再生成可能なcacheは、そのツール用のcache領域へ置く。唯一の原本や人が編集した資料を置かない。
  開発支援ツールの実行中だけ必要なscratchはOS標準一時領域の実行専用ディレクトリを使う。
- `.local/`は一時領域やcacheとして扱わず、作業完了、再起動、通常のcleanupを理由に
  一括削除しない。削除する場合は、保持の必要性と参照・依存関係を確認し、明示的な指示に従う。
- 本当に一時的で、処理終了後に失われても問題ないscratchは一時領域へ置く。
  test用一時領域は13.2に従い、repository内に置く場合は`.tmp/`を使用する。
  保存する評価結果や人が編集した資料を、一時ファイルと混在させない。
- `.local/`と`.tmp/`を使用するrepositoryでは、`/.local/`と`/.tmp/`を
  `.gitignore`へ追加する。Git管理外であることは、削除してよいことを意味しない。
- 既存の`.tmp/`から保存資料を移す場合は、根拠となる記録や再現用scriptも確認し、
  関連するlink・path・保存先の説明を更新する。cacheなど本当に一時的なfileは移さなくてよい。
- applicationが正式に管理するdata・state・cacheの保存先は7章に従う。
  `.local/`は開発資料の置き場であり、それらの保存先を置き換えない。

## 5. インストールと実行

### 5.1 通常のinstallationを基本にする

通常の個人利用では、次の形式でrepository rootへ移動してからinstallする。
例示している`C:\path\to\<repository-name>`は、対象repositoryの実際のfolder pathへ
置き換える。

```shell
cd "C:\path\to\<repository-name>"
uv tool install .
<cli-name> --help
```

READMEのinstallation例では`cd`を省略しない。利用者が想定current working directoryを
推測せずに済み、`.`が対象repositoryを意味することをcommand列だけで確認できるようにする。

日常利用するCLIのPython dependencyは、原則として`uv tool install .`だけで
揃える。一部のcommandでしか使わないdependencyでも、install時間や容量への
影響が許容範囲なら通常のdependencyに含める。`[context]`などのextraを
使い分ける手間とREADMEで説明する負担も、dependencyを分けるかの判断に含める。
OS固有のdependencyは、必要に応じてenvironment markerで対象OSに限定する。

download容量やinstall時間が大きい、dependencyが競合する、対応platformが
限られるなど、通常のdependencyに含める実質的な問題がある場合は、機能別の
extraを設けてよい。その場合は、通常のinstallationで利用できる機能と
追加installationの手順をREADMEに明記する。

`uv`が管理するtoolの実行ファイルは、既定ではユーザーのtool bin directory
（通常は`~/.local/bin`）から使用する。スクリプト本体を手作業でそこへcopyしない。

この通常のinstallationでは、install時点のcode、package resource、dependencyを
`uv`が管理するtool環境へinstallする。repository側の変更は自動では反映されない。
`git pull`などでrepositoryを更新するたびに、更新後の内容を反映するため、
次の形式で再インストールする。

```shell
cd "C:\path\to\<repository-name>"
uv tool install . --reinstall
<cli-name> --help
```

repositoryのcode、package resource、dependencyを確実にtool環境へ反映する場合は、
すべてのpackageを再インストールし、cacheもrefreshする`--reinstall`を使用する。
`--force`は既存のtool環境を再作成し、実行ファイルdirectoryにある同名のentry pointを
置き換えるoptionである。通常のrepository更新では、packageの再インストールという意図を
明示する`--reinstall`を使用し、tool環境の強制再作成やentry pointの競合解消が必要な場合に
限って`--force`を使用する。

開発時にrepository内のsource code変更をすぐCLIへ反映する必要がある場合は、
editable installationを選択してよい。

```shell
cd "C:\path\to\<repository-name>"
uv tool install -e . --reinstall
```

editable installationでは、installされたcommandがrepository内のsource codeを
直接参照するため、source codeだけの変更は再インストールせずに反映される。
ただし、tool環境のdependencyとinstallation metadataは自動更新されないため、
次の場合は同じeditable installation commandを`--reinstall`付きで再実行する。

- dependencyを追加、削除、またはversion変更し、
  `pyproject.toml`または`uv.lock`を更新した
- `pyproject.toml`のpackage metadataまたはentry pointを変更した
- repository folderを移動またはrenameした
- editable installationの参照先が古い可能性がある

repository-localな`./.<namespace>/config.yaml`は例外的に元repositoryへ結び付くのではなく、
実行時のcurrent working directoryを基準に検索される。installation方式にかかわらず、
repository rootから実行した場合だけ、そのrepositoryのoverrideが通常の優先順位で
読み込まれる。

### 5.2 インストール後の確認

READMEには、installation直後に実行する確認commandを記載する。最低限、
次のいずれかを実行して、entry pointと設定解決を確認できるようにする。

```shell
<cli-name> --help
<cli-name> config list
<cli-name> --version
```

## 6. 設定

### 6.1 形式

本書の `<namespace>` は、対象projectが決める設定・データ保存先の名前空間を表す
placeholderである。例として `example-tools` を選ぶと、
`~/.<namespace>/<application_id>/` は `~/.example-tools/<application_id>/`、
`./.<namespace>/config.yaml` は `./.example-tools/config.yaml` となる。
namespaceとapplication IDは具体値をREADMEへ記載し、placeholderをそのまま保存先にしない。
既存projectの保存先は移行条件を定めずに変更しない。

- 設定ファイルは、原則としてYAMLの`config.yaml`に統一する。
- YAMLの日時・引用符・型・パス表記は、[YAML記述規約](yaml-style.md)に従う。
- built-in defaultsを持ち、最低限の設定不足は具体的なerrorとして示す。
- repositoryには、秘密情報と個人固有pathを含まないpackage resource
  `src/<python_package>/resources/config.example.yaml`を置く。
- `config.example.yaml`はrepositoryの`.<namespace>/`へ置かない。wheel installation後も
  repositoryに依存せず利用できるよう、Python packageの一部として配布する。
- installed applicationでも初期設定を作成できるよう、`config init`または同等のcommandを
  提供する。exampleを`~/.<namespace>/<application_id>/config.yaml`へ作成し、作成結果と絶対pathを
  表示する。
- 利用者にwheelやtool環境のinstallation directoryを探させない。設定の作成・所在確認は
  `config init`と`config list`で完結させる。
- `config init`は、同一内容の既存fileを`unchanged`として扱い、編集済み設定を暗黙に
  上書きしない。置換を許可する場合は、明示的な`--force`とbackupまたは十分な警告を
  必要とする。
- repositoryの`.<namespace>/`はcurrent working directory固有の実設定またはoverride用であり、
  package resource、sample、templateの保管場所として前提にしない。
- 実設定の`./.<namespace>/config.yaml`はversion controlへ含めない。
- unknown key、型の不一致、未対応schema versionは、可能な限り黙って無視せず
  errorにする。

#### 6.1.1 config schema version

- applicationが管理する各`config.yaml`は、top-levelに`schema_version`を必須で持つ。
  値はSemVerに準じた`"MAJOR.MINOR.PATCH"`形式の文字列とし、常に3要素を記載する。
  YAMLで数値として解釈されないよう引用符で囲み、pre-release identifierとbuild metadataは
  原則として使用しない。
- `config.example.yaml`と`config init`が生成する設定では、YAML document markerと
  冒頭commentを除く最初のmapping keyに`schema_version`を配置する。これは人が互換性を
  最初に確認できるようにするserialization規則であり、loaderはkey順序に依存しない。
- config schema versionはapplication versionから独立した契約とする。各部分は次の意味で
  更新する。
  - Major: keyのrenameまたは削除、型や意味の変更など、既存configをそのまま安全に
    解釈できない変更
  - Minor: optional keyや既定値の追加など、新しいapplicationが同じMajorの古いconfigを
    読み込める後方互換な変更
  - Patch: accepted structureやkeyの意味を変更しないvalidation、説明、実装上の修正
- 後方互換性は、原則として新しいapplicationが同じMajorの古いMinorまたはPatchの
  configを読み込めることを意味する。古いapplicationが新しいMinorのconfigを読めることは
  保証しない。
- loaderは、対応中の同じMajorに属する古いversionを原則として受け入れ、不足するoptional
  keyをbuilt-in defaultsで補う。古いMajorも、明示的でtest済みのmigration経路がある場合は
  受け入れてよい。versionが異なるという理由だけで、古いconfigを一律にerrorにしない。
- 現在のapplicationより新しい未対応MajorまたはMinor、migration経路のない古いMajor、
  不正なversion形式は、推測して読み込まず、対応versionと必要なactionを示してerrorにする。
  Patchはstructureを変更しない契約であるため、同じMajorとMinorでは新しいPatchも
  読み込み可能とする。
- `schema_version`がないlegacy configを受け入れる場合は、対応するlegacy version、warning、
  migration経路を明示する。新規CLIまたはlegacy互換性が不要なCLIでは、欠落をerrorにする。
- migrationは、読込時に現在の内部modelへ変換するin-memory migrationと、利用者の
  `config.yaml`を更新する永続migrationを区別する。読込だけでconfigを暗黙に書き換えない。
  永続migrationは`config migrate`などの明示commandで行い、`--dry-run`、backup、validation、
  atomic replacementを備える。
- `config list`では、secretを除く解決後の設定に加え、各設定sourceから読み込んだ
  schema version、内部で使用するeffective schema version、migrationの有無を確認できるように
  する。

例:

```yaml
schema_version: "1.3.4"

raw_data_root: 'C:\path\to\raw'
request_timeout_seconds: 30
```

#### 6.1.2 config property名

- applicationが管理する設定propertyは、top-levelとnested mappingの両方で`snake_case`を
  既定とする。`schema_version`もこの規則に従う正式なkey名である。
- CLI optionは一般的な`kebab-case`を使用してよい。同じ設定を表す場合は、例えば
  `--run-records-root`と`run_records_root`のように、機械的に対応できる名前にする。
- Pythonの設定modelと内部fieldも、特別な理由がなければconfigと同じ`snake_case`を使用する。
- 外部API、外部schema、既存の公開済みconfigとの互換性により別の命名を維持する場合は、
  adapterまたはaliasで境界を明示し、内部へ異なる命名規則を拡散させない。
- 既存keyのrenameはbreaking changeとして扱い、Major version、旧keyの扱い、migration方法、
  deprecation期間を定義する。
- この命名規則はapplication-ownedな設定propertyに適用する。Markdown Frontmatter、manifest、
  外部service payloadなど、別の公開済みartifact契約を自動的なrename対象にしない。

### 6.2 application ID

設定とapplication dataのdirectory名には、安定した`snake_case`の
`<application_id>`を使用する。repository名のprefixが所有者の識別だけを目的とする場合、
application IDから省略してよい。

例:

| Repository | Application ID | CLI |
| --- | --- | --- |
| `example_video_note` | `video_note` | `example-video-note` |
| `example_receipt_pipeline` | `receipt_pipeline` | `example-receipt` |
| `example_context_pipeline` | `context_pipeline` | `example-context` |

一度公開したapplication IDは、repository名を変更しても安易に変更しない。

### 6.3 設定の優先順位

設定は次の順で読み込み、後の値が前の値を上書きする。

1. built-in defaults
2. `~/.<namespace>/<application_id>/config.yaml`
3. `./.<namespace>/config.yaml`
4. `--config`で指定した設定ファイル
5. 個別のCLI option

この優先順位はREADMEとtestの両方へ記載する。

- `~/.<namespace>/<application_id>/config.yaml`は、ユーザーの通常設定である。
- `./.<namespace>/config.yaml`は、current working directory固有のoverrideである。
- `--config`は、特定の実行で使う設定ファイルを明示する。
- 個別CLI optionは、その1回の実行だけを最終的に上書きする。
- 各設定ファイルはmerge前に個別にschema version、unknown key、型を検証し、必要な
  in-memory migrationを適用する。検証前のmappingをそのままmergeしない。
- `schema_version`は設定sourceのmetadataであり、通常の設定値として上位sourceや
  CLI optionで上書きしない。schema検証後の設定propertyだけを優先順位に従ってmergeする。
- 各値がどのsourceから決まったかを、`config list`で確認できるようにする。
- secretは、`config list`やlogへ平文で表示しない。

#### 6.3.1 自動読込とprofile選択の区別

- `--config`や`--profile`を省略しても、user-globalとCWDの既定の設定ファイルを
  自動で確認し、存在するものを6.3の優先順位に従って読み込む。CWDに設定がないことを
  理由にuser-global設定の読込を省略しない。CWDは実行時の作業フォルダであり、
  installation先やsource codeの場所ではない。
- この規則は「最初に見つかった1ファイルだけを選択する」方式ではなく、存在する
  設定sourceを統合する方式である。上位sourceで省略された値は下位sourceから引き継ぐ。
- 自動探索先にファイルがない場合は読み飛ばす。明示した`--config`のファイルが存在しない、
  または読み込む設定が不正な場合は、別の設定で黙って実行せず、該当pathと理由を示す。
- profileを持つCLIでは、設定ファイルの探索とprofileの選択を別の処理として定義する。
  `--profile`は統合後の設定内のprofile名を選択し、通常はファイルpathを指定しない。
  省略時は`default_profile`などの文書化した既定選択を使い、通常利用で毎回の引数指定を
  必須にしない。
- 選択されたprofile名が存在しない場合は、別のprofileや全profileを推測して実行しない。
  errorには、指定名、その値を決めたsource、利用可能なprofile名、読み込んだ設定path、
  修正方法を示す。設定未作成の場合は`config init`などの初期化方法と保存先を示す。
- profile mappingを名前単位でmergeする場合、異なる名前は別profileとして残ることを
  文書化する。重複検証のerrorには、対象の名前とそれぞれの設定元を示す。
- 生成AIが検証用設定を作る場合、自動読込される`./.<namespace>/config.yaml`を安易に使用しない。
  実行専用の一時領域や検証記録用directoryへ置き、`--config`で明示する。
  CWD固有の実運用設定を作成した場合は、そのpath・目的・自動読込への影響を報告する。

### 6.4 パス値

- `~`をhome directoryとして展開する。
- 絶対pathはそのまま使用する。
- 新規projectでは、相対pathをcurrent working directory基準で解決することを
  既定とする。
- config file基準など別の規則を採用する場合は、全設定sourceで一貫させ、
  READMEとtestへ明記する。
- 実在確認、directory作成、read/write permission確認は、実際に必要になるstageで
  行い、`config list`のようなread-only commandが不要な変更を行わないようにする。

## 7. ユーザー領域とruntime file

application自身が管理するfileは、原則として次のように配置する。

```text
~/.<namespace>/<application_id>/
├── config.yaml
├── data/
└── state/

~/.cache/<application_id>/

<platform-temporary-directory>/
```

- `config.yaml`: ユーザー単位の設定
- `data/`: raw capture、registry、生成物など、長期保持するapplication data
- `state/`: checkpoint、manifest、run report、logなど、再起動後も必要な運用状態
- `~/.cache/<application_id>/`: 削除しても再取得または再生成できるcache
- platform標準一時領域: その実行中だけ必要なscratch

次の方針も守る。

- repository rootやcurrent working directoryへ、暗黙にruntime fileを作らない。
- output先が利用者の管理する外部data rootである場合は、configで明示する。
- 一時生成物は、成功時に確定先へ移し、失敗時に残すか削除する条件を定義する。
- atomic replacementに同一filesystemが必要な場合は、staging fileを確定先と
  同じdirectoryへ作る。
- directory layoutを変更する場合は、既存dataの自動移動、手動移行、旧pathの扱いを
  明記し、意図せず二重に読み込まない。

## 8. CLIの振る舞い

### 8.1 command設計

- command名は、処理対象と動作が分かる動詞または動詞句にする。
- 変更を表すcommand名は、optionなしで実行したときに、その名前が表す処理を
  実際に行うものとする。read-onlyな`plan`、`validate`、`verify`などは、
  command名から変更しないことが分かるようにする。
- 複数stageを持つpipelineでは、全stageを実行するcommandと、各stageを
  個別に再実行できるcommandの両方を検討する。
- `config list`または同等のcommandを設け、解決後の非secret設定と
  読み込んだ設定sourceを表示する。
- artifactを持つpipelineでは、`validate`または`verify` commandを設ける。
- `--help`には、optionの意味だけでなく、既定値や副作用が判断に必要な場合は
  それも記載する。

### 8.2 CLIログ

- CLIのconsole logには、原則としてPython標準の`logging` moduleを使用する。
- console logの1行は、`[<level>] <message>`形式とする。
- `<level>`は大文字で表記し、基本的に次のlevelを使用する。

```text
[DEBUG] 通常利用では不要な詳細診断
[INFO] 通常の進捗や処理状況
[SUCCESS] 利用者にとって意味のある処理または検証の正常完了
[WARNING] 処理は継続できるが、利用者の確認が必要な状態
[ERROR] commandまたは対象の処理を正常完了できないerror
[CRITICAL] application全体が継続できない重大なerror
```

- `SUCCESS`はPython標準のlevelではないため、数値`25`として追加する。
  これは`INFO`の`20`より高く、`WARNING`の`30`より低い。
- `[SUCCESS]`は、ingest全体の完了やartifact validation成功など、利用者が
  最終結果として認識すべき箇所で使用する。通常のstage開始、件数、進捗には
  `[INFO]`を使用し、細かな途中経過をすべて`[SUCCESS]`にしない。
- `config list`のようなread-onlyな設定確認は`[INFO]`で知らせる。
  設定の列挙だけで`[SUCCESS] success`を出さず、`[INFO] Showing resolved configuration`
  のように何を表示しているか具体的に記す。
- `[CRITICAL]`は通常の入力errorには使用せず、applicationの初期化不能など、
  処理全体を継続できない状態に限定する。
- console logには、既定ではtimestamp、logger名、module名を付けず、人が実行中に
  読みやすい短い形式にする。
- messageには、可能な限り現在のstage、対象、件数、次に取るべきactionを含める。
- 既定では`[INFO]`、`[SUCCESS]`、`[WARNING]`、`[ERROR]`、`[CRITICAL]`を
  表示する。
- `-q` / `--quiet`では進捗を省略し、`[ERROR]`と`[CRITICAL]`だけを表示する。
- `-v` / `--verbose`では、既定のlevelに加えて`[DEBUG]`を表示する。
- `-q` / `--quiet`と`-v` / `--verbose`は同時指定できないようにする。
- 予期された入力errorや設定errorは、簡潔で対応可能な`[ERROR]` messageとして
  表示する。tracebackは通常表示せず、debug時または想定外errorの診断で使用する。
- Windows consoleを含め、標準出力と標準エラーはUTF-8で日本語を扱えるようにする。

#### Console color

- interactive terminalでは、`[SUCCESS]`を含む成功行全体を緑、
  `[ERROR]`と`[CRITICAL]`を含む行全体を赤で表示する。
- `[DEBUG]`、`[INFO]`、`[WARNING]`には、既定では色を付けない。
- 色は意味を補助する表示に限定し、色がなくてもlevel prefixとmessageだけで
  状態を判断できるようにする。
- ANSI terminal colorを使用し、PowerShell固有の実装にしない。
- 色を付けるのは、出力先がTTYであり、ANSI colorを安全に使用できる場合だけとする。
- pipe、redirect、test captureなど、出力先がTTYでない場合は自動的に無色にする。
- `NO_COLOR` environment variableが存在する場合、値にかかわらず色を無効にする。
- `TERM=dumb`の場合は色を無効にする。
- Windowsでは、consoleの
  `ENABLE_VIRTUAL_TERMINAL_PROCESSING`を確認または有効化する。有効化できない場合は
  errorにせず、ANSI escape sequenceを含まない無色表示へfallbackする。
- 色付きmessageの末尾にはANSI resetを付け、後続のconsole表示へ色を漏らさない。
- 標準出力へ出すJSONなどのmachine-readable outputには色を付けない。

#### 実装の基本形

```python
import logging
import sys


SUCCESS = 25
logging.addLevelName(SUCCESS, "SUCCESS")

RESET = "\x1b[0m"
LEVEL_COLORS = {
    SUCCESS: "\x1b[32m",
    logging.ERROR: "\x1b[31m",
    logging.CRITICAL: "\x1b[31m",
}


def log_success(logger: logging.Logger, message: str, *args: object) -> None:
    logger.log(SUCCESS, message, *args)


class ColorFormatter(logging.Formatter):
    def __init__(self, fmt: str, *, use_color: bool) -> None:
        super().__init__(fmt)
        self.use_color = use_color

    def format(self, record: logging.LogRecord) -> str:
        rendered = super().format(record)
        color = LEVEL_COLORS.get(record.levelno) if self.use_color else None
        return f"{color}{rendered}{RESET}" if color else rendered


level = logging.DEBUG if verbose else logging.ERROR if quiet else logging.INFO
handler = logging.StreamHandler(sys.stderr)
handler.setFormatter(
    ColorFormatter(
        "[%(levelname)s] %(message)s",
        use_color=supports_color(sys.stderr),
    )
)
logging.basicConfig(level=level, handlers=[handler], force=True)
```

`supports_color`は、TTY、`NO_COLOR`、`TERM=dumb`、Windows virtual terminalの
条件を上記の規則どおりに判定する。Windows APIやTTY判定に失敗した場合は
例外でCLIを停止せず、`False`を返して無色表示へfallbackする。

console logについて、少なくとも次を自動testで確認する。

- `SUCCESS`のlevel名と、INFOより高くWARNINGより低いlevel順
- level prefixとmessage
- 既定、quiet、verboseのlevel filtering
- 標準エラーへのrouting
- 色付き`[SUCCESS]`が緑になること
- 色付き`[ERROR]`と`[CRITICAL]`が赤になること
- 無色時にANSI escape sequenceが含まれないこと
- TTYでない場合、`NO_COLOR`、`TERM=dumb`、Windows VT非対応時の無色fallback

長期保存するfile logが必要なapplicationでは、console logとは別のhandlerを使用して
よい。file logはUTF-8で`~/.<namespace>/<application_id>/state/logs/`へ保存し、調査に必要な
timezone付きtimestamp、level、messageを記録する。consoleとfileのhandlerを
併用する場合は、同じmessageの重複出力に注意する。

### 8.3 標準出力、標準エラー、終了コード

本節は、設定一覧だけでなく、処理、検証、設定操作などのCLI command全般の結果表示に
適用する。`--help`と`--version`は、それぞれの用途に応じた表示とする。

- 既定の標準出力は、人が読み、必要な値やpathをcopyして使える一覧表示とする。
  機械処理用JSONは、利用者または呼出元が`--json`を指定した場合に標準出力へ出す。
- 人が見る進捗、warning、diagnosticは、出力形式にかかわらず標準エラーへ出力する。
  標準出力へlogや進捗文を混在させない。
- 正常終了は`0`、validation error、処理失敗、一部失敗は非`0`にする。
  出力形式によって成功・失敗の判定や終了コードを変えない。
- 長時間処理では、現在のstage、対象、件数が分かるprogressを出す。
- `--quiet`と`--verbose`は標準エラーのlog量を制御し、最終結果の出力形式や有無を
  変えない。
- secret、credentialは通常の一覧とJSONのどちらにも含めず、logにも出さない。
  logへ個人情報や長大なsource本文を不必要に出力しない。

#### 通常の一覧表示

- `git config --list`のように、1行に1つの`key=value`を表示する形式を基本とする。
  nested mappingは`.`で、配列要素は`[0]`などのindexで表し、空の配列・mappingは
  それぞれ`[]`・`{}`と表示する。booleanは`true`/`false`、未設定値は`null`とする。
- 文字列はJSONの引用符で囲まず、Windows pathのbackslashを二重にescapeしない。
  pathを省略・切り詰めず、利用者がそのまま選択・copyできる表示にする。
  改行など1行表示を壊す制御文字はescapeする。
- 最終status、処理件数、失敗・要確認の対象と理由など、次の判断に必要な情報を表示する。
  対象pathと理由は別の項目にし、error本文からpathを抜き出さなくても使えるようにする。
- 大量の変更なし・skipの詳細は、既定では件数にまとめてよい。失敗・要確認の対象は
  確認できるようにし、省略した詳細がある場合は、その確認方法を示す。
- `--full-output`などの詳細表示optionを設ける場合は、表示する情報量を制御する。
  出力形式の選択とは分離し、通常の一覧表示と`--json`の両方で意味を揃える。

#### 機械処理用JSON

- `--json`指定時は、人向け一覧の代わりに、機械処理用の妥当なJSONを標準出力へ出す。
  通常実行、dry-run、検証、設定操作、失敗時も、選択された出力形式を守る。
- 1回の実行結果を1つのJSON documentとして出力し、人向けの見出し、log、進捗文、
  ANSI escape sequenceを混在させない。
- JSONのkey、型、status、件数、pathなどを機械処理用の契約として扱い、READMEまたは
  仕様書へ記載する。通常の一覧とJSONは同じ処理結果を表し、出力形式の選択によって
  処理内容、変更対象、副作用を変えない。
- AI、shell script、Task Scheduler、subprocessなどの呼出元が結果を解析して
  後続処理に使う場合は、`--json`を明示する。人向け一覧の文字列解析に依存しない。
- TTYかどうか、pipeやredirectの有無だけで、通常の一覧とJSONを自動切替しない。
  出力形式は`--json`の指定で決定し、色の有無は8.2の規則に従う。

#### 設定一覧の出力

- `config list`は設定を読み取って解決するread-only commandとし、設定file、
  state、cache、reportを作成・更新しない。
- 通常の一覧表示と`--json`の共通規則を適用する。
- 解決後の設定に加え、読み込んだ設定source、各値の決定元、選択中のprofile、
  sourceとeffectiveのschema version、migrationの有無など、該当する情報を表示する。

#### 文書化と既存CLIの移行

- `--help`とREADMEに、既定の一覧表示、`--json`、標準出力と標準エラーの役割、
  終了コードを記載し、人が実行する例と結果を解析する呼出例を示す。
- 既存CLIの標準出力をJSONから通常の一覧へ変更する場合は、出力契約のbreaking change
  として扱う。`--json`を用意し、既存の呼出元を確認・更新したうえで既定表示を変更する。
  version境界と移行手順を明示する。

### 8.4 変更を伴うcommand

#### 8.4.1 通常の変更command

- `build`、`fetch`、`ingest`、`summarize`、`transcribe`など、変更を表す通常の
  commandは、optionなしで実行した場合に、その名前が表す処理を実際に行う。
- 模擬実行は、利用者が`--dry-run`を明示した場合だけ有効にする。新規CLIでは、
  通常の変更commandを既定でdry-runにし、`--apply`または`--write`を付けた場合だけ
  実行する形式を採用しない。
- 変更対象または副作用を事前に計算できる処理には、原則として`--dry-run`を設ける。
  read-onlyな`list`、`show`、`validate`、`verify`などに、形式的な`--dry-run`を
  追加する必要はない。
- `--help`とREADMEには、optionなしの実行が変更を行うことと、`--dry-run`時に
  実行または省略する処理を明記する。

#### 8.4.2 dry-runの契約

- dry-runは、対象data、設定、state、cache、reportなど、applicationが管理する
  永続fileを作成、更新、削除しない。処理終了後に一時fileを残さず、外部systemも
  変更しない。
- 変更予定を正確に計算するために必要な場合は、local fileや既存stateの読取、認証、
  read-onlyなnetwork access、取得結果のmemory上でのvalidationを行ってよい。
  network access、認証、download、外部service利用の有無はREADMEと`--help`へ
  明記する。
- 生成AIの呼出など、課金または大きな計算資源を消費する処理は、dry-runでは原則として
  行わない。意味のあるpreviewに不可欠な場合は、費用と副作用を明記し、別の明示optionで
  利用者の承認を得る。
- dry-runでは、作成、更新、置換、削除、skipの予定件数と、必要に応じて対象pathや
  理由を表示する。通常実行と同じ設定解決、入力validation、安全条件の確認を可能な限り
  行い、本実行で失敗することが分かっている状態を成功扱いにしない。
- dry-run後から本実行までの間にfileや外部stateが変化する可能性があるため、結果の
  完全一致を保証しない。本実行では、確定直前に入力、競合、保護条件を再検証する。

#### 8.4.3 高リスク処理

- 削除、不可逆な上書き、外部公開、credential変更、複数の正本間の同期や競合解決、
  大規模migration、review済みの手動編集への変更は、高リスク処理として通常の変更と
  区別する。
- 事前reviewと実行対象の固定が重要な場合は、`plan`と`apply`を別commandとして設ける。
  `plan`は変更内容を表示し、必要に応じて利用者が明示したpathへplan artifactを保存する。
  `apply`はreview済みplanだけを適用し、入力hash、対象state、plan version、有効期限などを
  再検証して、古いplanや別の状態に対するplanを適用しない。
- plan artifactを使わない高リスク処理では、`delete`、`cleanup`、`publish`など、
  副作用が分かる専用commandと対話確認を使用する。automationで対話確認を省略する場合は
  `--yes`などの明示optionを要求し、そのoptionは確認省略以外の意味を持たせない。
- 削除を伴う場合は、copyまたは出力の検証完了後にだけ削除する。review済みの手動編集を
  上書きする可能性がある場合は、対象と保護解除条件を実行前に明示する。
- `--force`、`--overwrite`、`--refresh`、`--delete`は意味を分け、何が保護解除、置換、
  再取得、削除されるかをREADMEと`--help`へ記載する。これらを通常実行を有効にする
  一般的なwrite switchとして使用しない。
- `--write-*`は、複数の出力先から実際に書き込む対象を選択する必要がある場合だけ使用する。
  通常実行を有効にする確認optionとしては使用しない。

#### 8.4.4 既存CLIの移行

- 既存CLIが既定dry-runと`--apply`または`--write`を公開済みの場合、optionなしの意味を
  突然反転させない。Task Scheduler、cron、shell script、利用者の手順への影響を確認し、
  breaking changeとして段階的に移行する。
- 移行時は、`--dry-run`の追加、旧形式へのdeprecation warning、READMEとautomationの
  更新、version境界の明示、旧optionの削除または互換期間を計画する。重要dataを扱う
  既存CLIでは、必要に応じて`plan` / `apply`型へ再設計する。

### 8.5 冪等性

同じ入力、設定、generator条件で再実行した場合は、次のいずれかを明確に返す。

- `created`: 新規作成した
- `updated`または`replaced`: 内容を更新した
- `unchanged`: 同一のため変更しなかった
- `skipped`: 条件により対象外とした
- `failed`: 処理できなかった

出力が既存内容と異なる場合、暗黙に上書きせず、明示的な更新規則または
force optionを要求する。

## 9. データパイプラインの追加要件

### 9.1 stageとdata layer

対象に応じて、次のstageを分離する。

1. acquireまたはimport: sourceを取得し、rawとして保存する
2. validate: sourceの完全性とschemaを検証する
3. transformまたはbuild: 決定論的な変換を行う
4. generate: 必要な場合だけ生成AIなどの非決定的処理を行う
5. verify: outputとsourceの対応を検証する
6. commitまたはpublish: 検証済みoutputを確定する

生成AIを使うstageと決定論的なstageを分離し、READMEでどのcommandが
生成AI、network、外部processを使用するかを示す。

### 9.2 raw dataとprovenance

- 取得したraw dataは、原則として不変のcaptureとして保存する。
- source URL、取得日時、tool version、選択条件、file size、SHA-256など、
  再現と追跡に必要なprovenanceをmanifestへ記録する。
- source取得に失敗または不完全性がある場合、完全なoutputを装って後続stageを
  続行しない。failure reportまたはmanifestを残す。
- 入力に異常値や内部不整合がある場合、根拠となるraw dataを捨てず、
  warningと採用した処理規則を記録する。

### 9.3 書き込みと検証

- outputは一時fileへ完全に生成し、schemaと内容を検証してから原子的に確定する。
- fileのcopyまたはmoveでは、必要に応じてcopy後のSHA-256を照合する。
- 複数fileを1回の処理で変更する場合は、途中失敗時の状態、rollback、
  resume方法を設計する。
- 大量処理または長時間処理では、checkpointと再開可能なworking manifestを
  検討する。
- 削除は、保存先、derived output、manifestの確定後にのみ行う。
- 現在書き込み中のsource、当日分、lock中のfileなどを除外すべき処理では、
  対象外条件を明示してtestする。

### 9.4 schemaと互換性

- 長期保存するJSON、YAML、Markdown Frontmatter、manifestには、
  必要に応じてschema versionを持たせる。
- `config.yaml`のschema versionとproperty命名は、6.1.1と6.1.2の契約に従う。
  他のartifactはそれぞれの公開契約に従い、同じkey名やversion形式を一律に要求しない。
- 古いschemaの読み取り、migration、再生成のどれを行うかを定義する。
- 現在の実装より新しい未対応schemaを、黙って上書きしない。
- 設定、source、generator、rendererなど、outputへ影響する条件を
  fingerprintへ含めることを検討する。

### 9.5 時刻

- machine-readableな日時は、原則としてtimezone offset付きISO 8601を使用する。
- 日付partitionの基準、timezone、当日分の判定方法を設定または仕様で明示する。
- filesystem timestampだけをsource event timeの代わりに使わない。

## 10. 生成AIまたは外部processを使う場合

- 外部process呼び出しをadapterへ分離し、timeout、終了コード、標準出力、
  標準エラーを扱う。
- 実行ファイルはconfigまたは明確な探索規則で解決し、Windowsのshim、
  PATH、実行account、permission差を考慮する。
- 生成AIへ渡したsource、model、promptまたはprompt version、
  structured output schemaを追跡できるようにする。
- 生成AIへのsemantic instruction、structured output schema、最終出力templateは、
  Python codeへ分散して重複定義せず、application-owned resourceとして外出しする。
- 相互依存するprompt、schema、templateは1つのprofileとして読み込む。日本語・英語や
  別用途の出力を提供する場合は、`default-ja`、`default-en`など、目的が分かる
  curated profileとして分離し、configまたはCLIでprofile単位に選択する。
- 個別resourceを利用者が自由に組み替えられる仕組みは、契約理解と品質保証が必要なため、
  明確なuse caseがない限り提供しない。
- 有効なmodel名を取得できる場合は、outputまたはmanifestへ記録する。
- 生成AIの出力は信頼せず、schemaとsourceとの対応をdeterministic codeで検証する。
- 生成AIを呼び出せない場合に、内容を推測して成功扱いにしない。
- retryは、同じ処理の重複や意図しない課金を防ぐ条件と回数上限を持たせる。
- credentialの作成、変更、表示をapplicationの暗黙の責務にしない。

## 11. README

README の新規作成・大幅な改訂では、[CLIリポジトリREADME執筆規約](readme-writing.md)を併読する。
日常の修正では、同規約の関連する節と完了前の確認項目を適用する。
本章はREADME、CHANGELOG、関連文書の配置・役割・更新に関する必須要件を定め、詳しい構成例・書き方・改訂手順はREADME執筆規約を正本とする。
これらの文書を、人がCLIの成果物・動作・設計を理解し、変更の妥当性を確認するための入口として維持する。
人と生成AIのどちらが作業する場合も、参照先の本文を実際に確認する。
同じガイドラインの全文コピーを各repositoryへ配置する必要はない。

### 11.1 主言語と補助言語

`README.md`を主文書とし、英語で作成することを基本とする。
日本語の補助版`README_ja.md`は、対象ユーザーや運用上の必要性に応じて用意する。
主要な対象ユーザーが日本語話者であることが明確な場合は、`README.md`を日本語にできる。
この場合、英語の補助版が必要なら`README_en.md`を用意する。
補助言語版は任意とし、どちらの構成でも`README.md`だけでよい。
複数の言語版を維持する場合は、先頭付近に相互linkを置き、command、設定、前提、手順、警告、制約、図、開発手順の意味を同期する。
各言語で自然に読める文章にし、具体的な選択・同期方法は、専用ガイドラインの「主言語と補助言語」を参照する。

### 11.2 読者と構成

README は、初めてCLIを使う利用者が、用途・入力・得られる結果を理解し、最初の処理と結果確認まで進める入口にする。
既存利用者は、目的別のcommand一覧やlinkから必要な詳細を直接参照できるようにする。

#### 11.2.1 基本構成

概要、短い出力例、必要な対象範囲・用語・全体像を示し、セットアップ、初回実行、結果確認、日常利用を前半へ置く。
設定・処理・保存構造・制限・保守・移行・開発などの詳細は、必要時に参照できるよう後半や関連文書へ置く。
章数や見出し名は固定せず、対象に存在しない機能の空の章は作らない。
構成例と各章の目的は、専用ガイドラインの「情報の配置と章構成」に従う。

#### 11.2.2 「使い方」に含める内容

必要環境、作業folder、通常のinstallation、最小限の設定、初回処理、日常の更新、結果の保存先と確認方法をつなげる。
通常のinstallationは5.1、CLIの動作とdry-runは8章に従い、未実装の操作を文書上だけで追加しない。
自動で実行される補助処理と、利用者が別途実行する操作、任意の下流処理を区別する。
具体的な手順の書き方は、専用ガイドラインの「手順と仕様の説明」に従う。

#### 11.2.3 詳細説明の配置と保持

有用な仕様、制約、安全条件、既存利用者に必要な移行説明を保持して再構成する。
重複する詳細は1か所へ集約し、概要・手順・command一覧から案内する。
文字数だけを理由に説明を削除・分割せず、読み方と探しやすさで配置を判断する。

### 11.3 図とdiagram

構造や処理の関係が理解しやすくなる場合は、原則としてGitHub互換のMermaidを使う。
図だけに情報を閉じず、導入文と必要な条件・注意を添える。
複数の言語版がある場合は、図の構成要素と関係を一致させる。
図・表・実例の使い分けと構文上の注意は、専用ガイドラインの「実例・表・図・リンクの使い分け」に従う。

### 11.4 command説明の必須内容

主要commandについて、次の内容をREADMEと直接参照できる詳細文書から判断できるようにする。

- 目的、入力、処理対象、必要な設定・前提
- 作成・更新・削除するoutputと保存先
- 通信、外部送信、費用、上書きなどの副作用と保護条件
- 再実行、skip、中断・再開の条件
- 代表的な実行例と成功を確認する方法
- 通常の一覧表示と`--json`の使い分け、標準出力・標準エラー・終了コード

#### 共通optionとdry-runの説明

通常実行とdry-runの違いは、最初の実行前に短く示す。
詳細は8.4の契約と現在の実装に一致させ、共通説明へ集約する。
操作前に必要な費用・外部送信・削除・上書きの注意は、その操作の近くにも要点を残す。
共通optionとcommand固有の例外の書き分けは、専用ガイドラインの「コマンドは動作と結果を説明する」に従う。

#### command例の表記

READMEと関連文書では、特定のshellに依存しないcommand例のコードフェンスに、原則として`shell`を指定する。
実行場所は「ターミナル」などの汎用的な表現にし、未検証のOSやshellでの動作を保証しない。
shell固有の構文が必要な例は対象shellを明記し、適切な言語指定を使う。
採用理由、`console`との使い分け、実行条件の書き方は、[README執筆規約の「6.3. コード例は実行条件まで含める」](readme-writing.md#63-コード例は実行条件まで含める)に従う。

### 11.5 README更新時の確認

初見の利用者が最初の結果まで進めること、既存利用者が必要な詳細を探せること、有用な説明が改訂後も残ることを確認する。
記述した事実、command・設定例、link、Markdown、図を確認し、未確認事項を確認済みとして扱わない。
複数の言語版がある場合は、言語間の意味の整合も確認する。
確認手順と完了条件は、専用ガイドラインの「執筆・更新の手順」「完了前の確認」に従う。

### 11.6 文書の配置と役割

`README.md`と`CHANGELOG.md`はrepository rootに置き、projectの詳細文書は原則として`docs/`直下へまとめる。
`configuration.md`や`cli-output.md`など、内容を判断できるfile名を使う。`reference/`、`specs/`、`guides/`などの分類folderは必須にしない。
文書が増え、関連文書をまとめた方が探索・更新しやすい場合だけ、用途と格納範囲を定めてsubfolderを追加する。
project自身の詳細文書を置くために、`reference/`をrepository rootで`docs/`と並列に新設しない。

- READMEから、各文書を読む目的と説明の正本へ到達できるようにする。詳細の正本は1か所に決め、他の文書には必要な要約とlinkを置く。
- 仕様・設定・運用・設計理由・検証の説明を、読者が確認したい内容に応じて用意する。すべてを独立fileにする必要はない。
- 意図する契約、実装済みの動作、未実装の提案を区別する。検証結果には対象versionまたはcommit、実施日、条件、確認した範囲と未検証事項を示す。文書・実装・結果の不一致を隠さない。
- 現在の仕様・手順と、特定時点の判断・検証・引き継ぎの記録を区別する。現在の利用に必要な説明を引き継ぎ文書だけに残さない。私的な作業記録は4.5に従う。
- 動作を変える作業では、関係するREADME・詳細文書・CHANGELOGを同じ作業で更新し、複数の言語版がある場合は対応する説明と参照先の整合も保つ。
- 文書を移動するときは有用な内容を保持し、文書・生成処理・検証処理からの参照を更新する。既存構成の一括移行は要求せず、対象と影響を確認し、依頼された範囲で適用する。

文書名の例、reference等の用語、正本と分割、時点を持つ記録の扱いは、専用ガイドラインの「文書の役割を決める」に従う。
事実と検証根拠の書き分けは、同ガイドラインの「正確さと説明の根拠」に従う。

### 11.7 CHANGELOG

標準的なCLI repositoryには、repository rootに`CHANGELOG.md`を作成し、
利用者から見た変更履歴を継続して更新する。READMEの入口からlinkする。
新規projectでは最初のversionの主な機能を記録する。既存projectでは、
確認できるGit履歴とversion情報に基づいて履歴を整理し、根拠のない公開日や変更点を補わない。

- 未リリースの変更は`Unreleased`に記録し、正式なversionと公開日が確定した時点で
  そのversionの節へ移す。開発中の変更に未決定のversion番号や公開日を付けない。
- 機能追加、動作変更、bug修正、非互換変更を、利用者への影響が分かる言葉で記す。
  CLI option、設定schema、出力形式、保存先、依存環境が変わる場合は、必要な
  migration、再installation、automation修正も明記する。
- READMEには現在の使い方と必要な注意を保ち、変更の経緯はCHANGELOGに記す。
  commit messageの羅列や内部的な作業記録で代用しない。
- 実装と同じ作業で該当する変更履歴を更新し、version更新やreleaseの前に内容と日付を確認する。
  根拠が不十分な過去versionは推測で埋めず、確認できた範囲を明示する。

## 12. Public repository hygiene

- README、sample config、test、fixture、commentへ、個人の実pathを含めない。
- Windows pathの例には、`C:\path\to\...`または `C:\Users\ExampleUser\...`を使用する。
- secret、token、cookie、credential、private URL、個人データをcommitしない。
- `.gitignore`には、少なくとも`.venv/`、`.local/`、testやlintのcache、build artifact、実設定の`.<namespace>/config.yaml`、runtime dataを含める。
- packageに含める`resources/config.example.yaml`、prompt、schema、templateなどの非code resourceは除外せずcommitする。
- test fixtureは最小化、匿名化し、実データの意味を残す必要がある場合は公開可能性を確認する。
- licenseを明記し、dependencyのlicenseと配布条件を確認する。

## 13. Testと品質確認

### 13.1 最低限の自動test

次のうち、applicationに該当するものをtestする。

- configの5段階の優先順位
- `config init`の新規作成、同一fileの`unchanged`、編集済みfileの保護
- `config.example.yaml`と`config init`が、引用したSemVer形式の`schema_version`を
  最初のmapping keyとして生成すること
- 同じMajorの古いversion、同じMajor / Minorの新しいPatch、migration可能な古いMajor、
  version欠落、不正形式、未対応の新しいMajor / Minorの扱い
- 複数のconfig sourceをmerge前に個別検証し、`schema_version`を設定値として
  上書きしないこと
- config propertyがnested mappingを含めて`snake_case`であり、対応するCLI optionの
  `kebab-case`と一貫していること
- unknown key、不正な型、未対応schemaを黙って無視しないこと
- Windows形式とPOSIX形式のpath
- CLI help、引数、終了コード、既定の一覧表示と`--json`の機械処理用JSON
- 通常・dry-run・成功・一部失敗・失敗時の出力形式と終了コード、stdout/stderrの分離
- pathをそのままcopyでき、TTY・pipe・redirectで出力形式が自動切替されないこと
- 出力形式の選択が処理内容・副作用を変えず、quiet/verboseが最終結果を抑制しないこと
- CLI logのlevel、SUCCESS、quiet、verbose、color、無色fallback
- 通常の変更commandがoptionなしで名前どおりの処理を行い、`--dry-run`指定時だけ
  模擬実行になること
- dry-runがlocalと外部systemのdata、設定、state、cache、reportを変更せず、
  課金を伴う処理を既定で行わないこと
- 高リスク処理がplan / apply、対話確認、`--yes`など、定義した安全機構を守ること
- 同一入力の再実行が`unchanged`になること
- 異なる既存outputを暗黙に上書きしないこと
- atomic writeと途中失敗
- hash、manifest、provenance
- deletionが全検証完了後だけ行われること
- timezone、日付境界、当日分除外
- 外部processのtimeout、failure、invalid output
- wheelとsource distributionに必要なpackage resourceが収録され、repository外から
  読み込めること
- profileを持つ場合は、各profileのresource一式、選択、validation、識別情報

### 13.2 Test用一時領域

- 実行中だけ必要なscratchは、test frameworkが管理する一時領域またはplatform標準の
  一時領域を優先する。
- 標準一時領域を使用できない場合は、書き込み可能な外部一時領域に、その実行専用の
  一意なdirectoryを作成する。
- Windows ACLや生成AIのsandboxにより外部一時領域を使用できない場合に限り、
  repository内の`.tmp/<tool>/<run-id>/`をfallbackとして使用できる。このdirectoryは
  `.gitignore`で除外し、用途と実行単位をpathから識別できるようにする。
- `pytest --basetemp`など実行前に指定先を消去する機能には、必ず新規の専用子directoryを
  指定する。repository root、`.tmp`自体、共有cache、成果物directoryを指定しない。
- 一時directoryはtest終了後に削除する。削除に失敗した場合は正確なpathと理由を報告し、
  別名の`.test-tmp-*`などを繰り返し作成しない。
- test fixtureはGit管理、再利用するcacheやbuild artifactはtoolごとの既定directory、
  確認・保存するtest結果は専用のresultsまたはreports directoryとし、一時領域と区別する。
  Git管理外でローカルに保持する開発時の結果は、4.5に従い`.local/state/<datetime_work-id>/`へ置く。
- 人が実行する場合と生成AIが実行する場合で基本原則は変えない。生成AIにはfallbackの順序、
  許可するrepository内path、cleanup、再試行の上限を明示し、sandbox userと通常userの
  permissionまたは所有権の差を考慮する。

### 13.3 開発時の標準確認

projectの採用toolに応じて、READMEへ次のようなcommandを記載する。

```shell
uv sync --locked
uv run pytest
uv run ruff check .
uv run mypy src
uv build
```

- `ruff`と`pytest`を基本とする。
- structured modelや複数moduleを持つapplicationでは、原則として`mypy`の
  strict modeを検討する。
- lock fileどおりの環境でtestし、wheelとsource distributionをbuildする。
- 少なくとも1つの代表fixtureで、CLI entry pointからoutput検証までの
  end-to-end testを行う。
- bug修正では、原因を再現するregression testを追加する。
- 外部serviceや実データに依存する修正では、自動testに加えて、許可された
  代表データによる実動作確認を行う。実動作確認を省略した場合は明記する。

### 13.4 完了時の報告

生成AIは、実装完了時に少なくとも次を報告する。

- 作成または変更した主なfile
- 利用者から見た変更点
- 実行したtestと結果
- 実行しなかったtestと理由
- migration、再installation、設定変更など利用者に必要な作業
- 本仕様からの逸脱と理由

## 14. 新規作成時の受入条件

新しい標準的なPython CLIは、次をすべて満たした時点で基本的な完成とする。

- `uv`でdependencyを解決し、lock fileを再現できる
- `uv tool install .`でCLIをinstallできる
- repository外のworking directoryからCLIを実行できる
- `--help`と`config list`が動作する
- `config init`がuser-global設定のpathを表示し、編集済み設定を保護する
- 設定の5段階の優先順位が実装され、testされている
- 各config fileが引用したSemVer形式の`schema_version`を持ち、互換version、未対応version、
  legacy config、migrationの扱いが実装・testされている
- config propertyはnested mappingを含めて`snake_case`を既定とし、CLI optionとの対応と
  既存契約に対する例外が定義されている
- user data、state、cache、一時fileの保存先が分離されている
- 通常の変更commandはoptionなしで実行され、`--dry-run`指定時だけ模擬実行になる
- dry-runのread-only access、課金処理、永続file、外部systemに関する境界が定義されている
- 高リスク処理のplan / apply、対話確認、automation時の確認省略、上書き規則が
  必要に応じて定義されている
- 変更処理が冪等であり、既存outputとの競合規則が定義されている
- outputを検証できるcommandまたは処理がある
- CLI全般で既定の人向け一覧と`--json`の機械処理用JSONが実装・testされている
- console logのlevel、SUCCESS、stderr/stdout分離、color fallbackが実装・testされている
- repository rootに`CHANGELOG.md`があり、最初のversionの変更点を記録し、READMEから参照できる
- READMEの主言語と任意の補助言語版の構成が11.1に従っている
- 複数の言語版がある場合は、相互linkと意味の整合が保たれている
- READMEの用途・入力・出力例と、前半の利用手順から、
  必要環境、installation、最小限の設定、初回処理、日常の更新、結果確認まで理解できる
- command一覧から詳細へ移動でき、設定・command・保守・移行・仕様・開発の詳細が
  初回利用の流れを中断せず、既存の有用な説明も保持されている
- 詳細文書の役割と正本が明確で、READMEから仕様・設計理由・検証の説明へ到達できる。
  意図する契約、実装状況、提案、検証済みの範囲を区別し、必要な説明が引き継ぎ記録だけに残っていない
- dry-runなどの共通説明が集約され、初回と日常の対象範囲、自動処理と個別実行、
  各commandの副作用・安全条件を判断できる
- 複数componentの関係や処理flowを図示する場合、配置依存のアスキーアートではなく
  GitHub互換のMermaidを使用し、複数の言語版がある場合は図の意味も同期している
- sample configにprivate pathやsecretがない
- 実行時に必要な非code resourceがpackageへ収録され、repository外から利用できる
- unit test、CLI test、代表的なend-to-end testが成功する
- lint、type check（採用時）、package buildが成功する
- Windowsで実行確認し、Linux非対応部分がある場合は明記されている

## 15. 生成AIが実装前に確認すること

生成AIは、実装を始める前に次を確認する。

1. repository内の`AGENTS.md`、仕様書、既存README、`CHANGELOG.md`、`pyproject.toml`
2. 現在のdirectory構成、CLI entry point、設定解決、保存先
3. 未commit変更と、利用者がすでに行っている作業
4. source、output、削除、上書きの安全条件
5. Windows固有要件とLinuxで共有できる範囲
6. 既存CLIを変更する場合の互換性とmigration

既存projectへ本仕様を適用する場合、無関係な全面rewriteを行わない。まず差分を
確認し、目的に必要な範囲で段階的に整合させる。

## 16. 参考実装

参考実装を使う場合は、対象projectに必要な性質を備えた実装を選び、参照版と確認範囲を記録する。
参照先にアクセスできない場合は、作業者が読める資料を用意する。
参考実装が本仕様のすべてを同じ方法で実装しているとは限らない。
個別の実装をそのままcopyするのではなく、本仕様と対象の要件を優先する。

## 17. 生成AIへの依頼テンプレート

新しいscriptまたはpipelineの作成依頼では、この文書をcontextとして渡したうえで、
次のtemplateを使用できる。
READMEを作成・改訂する作業では、11章から参照する専用ガイドラインも渡すか、作業者が読める参照先を指定する。

```md
添付した「Python CLI開発基本方針」に準拠して、次のPython CLIを
設計・実装してください。

## Goal

<達成したい状態>

## Input

<入力元、形式、量、更新頻度>

## Output

<出力先、形式、利用方法>

## Processing

<必要な処理、変換、生成AIや外部serviceの利用>

## Safety requirements

<通常実行とdry-runの境界、削除、上書き、plan / applyまたは確認、保持すべきraw data、停止条件>

## Operation

<手動実行、定期実行、想定OS、所要時間>

## README language

<主要な対象ユーザーとREADME.mdの言語。英語を基本とし、日本語話者が主対象と明確な場合は日本語も可>
<補助言語版の要否。README_ja.mdまたはREADME_en.mdは任意>

既存repositoryがある場合は、現在の構成と未commit変更を確認し、方針との差分を
説明してから必要な変更を実装してください。完了時は、利用方法、test結果、
再installationやmigrationの要否、方針からの逸脱を報告してください。
```

個別依頼で未指定の項目は、生成AIが安全で小さな既定案を採用してよい。
ただし、削除、不可逆な上書き、外部公開、credential変更など、利用者の権限や
重要なdataへ影響する判断は推測で実行しない。

## 18. 今後の更新

この仕様は、参考repositoryの改善、新しい失敗事例、WindowsとLinuxでの運用経験を反映して更新する。

更新時は、個別project固有の要件を無条件に全projectへ一般化せず、次を区別する。

- すべての標準CLIに必要な基本要件
- データパイプラインにだけ必要な安全要件
- 生成AIや外部processを使う場合だけ必要な要件
- 特定projectにだけ必要な要件

project固有の詳細が増えた場合は、この基本方針へ詰め込まず、各repositoryの仕様書またはREADMEへ分離する。
