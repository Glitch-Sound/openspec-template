# OpenSpec テンプレート

OpenSpec を使う開発の共通部分と雛形を管理する。\
要件を日本語の EARS と Gherkin で書き、テスト駆動で実装し、要件とテストの対応を検査スクリプトで確かめる。\
言語やプロジェクトには依存しない。

## 前提条件

- Node.js 22.12 以上と npm(OpenSpec CLI と検査スクリプトに使う)
- git で管理するリポジトリ(コミット前のフックで全体検証を実行する)
- AI ツール: GitHub Copilot(VS Code)、Claude Code、Codex のいずれか
- テストの名前やコメントに、シナリオID(例: `REQ-001-S01`)を書けるテストの仕組み

OpenSpec の版は、[版と更新履歴](#版と更新履歴)の "確認した OpenSpec" を使う。

## 新しいプロジェクトへの導入

次の5つの手順で導入する。\
コマンドは、すべてプロジェクトのルートで実行する。

| 手順 | 内容 |
| --- | --- |
| 1. ファイルと道具を用意する | テンプレートをコピーし、OpenSpec を入れる |
| 2. OpenSpec を初期化する | グローバル設定を行い、AI ツール向けのスキルを生成する |
| 3. プロジェクトに合わせて埋める | 雛形の `{{...}}` を書き換える |
| 4. 検査を動かす | 全体検証を設定し、最初のコミットを行う |
| 5. 開発を始める | VS Code で開き、AI に依頼する |

### 1. ファイルと道具を用意する

1. プロジェクト用のディレクトリを作り、`template/` の中身をルートにコピーする。\
   `.openspec-template`、`.gitignore`、`.gitattributes`、`.githooks/`、`.vscode/` などの隠しファイルも含める。
2. 次のコマンドを実行する。

   ```sh
   git init
   npm init -y
   npm install -D @fission-ai/openspec@1.16.0
   ```

- `npm init -y` は、`package.json` がない場合だけ実行する。\
  `package.json` は、OpenSpec の版を固定するため、言語に関係なく必要である。
- プロジェクトの言語の環境(例: Python の `pyproject.toml`)は、テンプレートとは別に、その言語の道具で用意する。
- テンプレートを OpenSpec より先にコピーしても、手順2の `openspec init` はコピーしたファイルを変更しない。

### 2. OpenSpec を初期化する

1. コピーした `README.md` の "OpenSpec のグローバル設定" にある3つのコマンドを実行する。\
   マシンごとに最初の1回だけ必要で、必ず次の `openspec init` より前に行う。\
   行わないと、ff と verify のスキルが生成されない。
2. AI ツール向けのスキルを生成する。

   ```sh
   npx --no-install openspec init --tools github-copilot,claude,codex
   ```

   `--tools` には、使う AI ツールだけをカンマで区切って指定すればよく、3つすべてを指定する必要はない。\
   `github-copilot` は GitHub Copilot、`claude` は Claude Code、`codex` は Codex を表す(例: GitHub Copilot だけなら `--tools github-copilot`)。\
   実行の後に表示される `Restart your IDE to refresh skills.` は、生成したスキルを AI ツールに読み込ませるための案内である。\
   この時点では再起動せず、手順5で読み込ませればよい。

### 3. プロジェクトに合わせて埋める

次の箇所を、プロジェクトの内容に書き換える。\
ほとんどは `{{...}}` の入力欄で、README.md のセットアップだけは、言語に合わせて例の文を直す。\
それ以外の節には推奨の内容が書いてあるので、必要なときだけ直す。

| ファイル | 書き換える箇所 |
| --- | --- |
| `README.md` | プロジェクト名、1行の説明、セットアップ(言語に合わせる)、使い方 |
| `openspec/product.md` | 目的、対象範囲 |
| `openspec/tech.md` | 主要な技術、構成要素と外部サービス |
| `openspec/structure.md` | プロジェクト固有のディレクトリと責務 |

product.md は、AI が判断するときの基準になるので、開発を始める前に書いておく。

### 4. 検査を動かす

1. `package.json` の `scripts` に、全体検証の `check` を追加する。\
   仕様の検査、トレース検査、台帳(`openspec/traceability.md`)の生成を、この順に実行する。

   ```json
   "check": "npx --no-install openspec validate --specs --strict && node scripts/openspec-trace.mjs check && node scripts/openspec-trace.mjs generate"
   ```

   テストや静的解析は、導入した時点で AI がプロダクトの検査として言語の道具の側にまとめ、`check` の先頭から呼ぶ(例: `"check": "uv run --locked poe check && npx --no-install openspec ..."`)。\
   プロダクトの検査を言語の道具の側に置くのは、OpenSpec の部分を切り離して納品する場合でも、納品物だけで検査を実行できるようにするためである。\
   `npm init -y` が入れた `main` と、`scripts` の `test` は、使わないので削除してよい。\
   `test` は "テストがない" と表示して失敗するだけで、残すと `npm test` で意味のない失敗が出る。

2. テストファイルの場所を指定する場合は、`openspec/trace.json` を作る(書き方は [docs/openspec-trace-spec.md](docs/openspec-trace-spec.md))。\
   指定しない場合は、一般的なテストファイルの名前から探す。
3. フックを有効にし、全体検証が成功することを確かめる。

   ```sh
   git config core.hooksPath .githooks
   git add --chmod=+x .githooks/pre-commit
   npm run check
   ```

   2行目は、他のマシンで clone したときにもフックが動くよう、実行の権限を付けて記録する。\
   macOS と Linux では、2行目の前に `chmod +x .githooks/pre-commit` も実行する。

4. 最初のコミットを行う。

### 5. 開発を始める

1. VS Code でプロジェクトを開く。\
   すでに開いている場合は、スキルと AGENTS.md の設定を読み込ませるため、コマンドパレットの "Developer: Reload Window" でウィンドウを再読み込みする。\
   Claude Code と Codex は、すでに起動している場合は終了して起動し直す。
2. AI に依頼して開発する(例: `/openspec-ff-change ○○を作りたい`)。\
   ID の採番と台帳の作り直しは、AI と全体検証が自動で行うため、人がコマンドを実行する必要はない。

### 既存のリポジトリに導入する場合

手順は同じで、次の6つのファイルだけ扱いを変える。\
同じ名前の既存のファイルを上書きしてしまうため、コピーせず、既存のファイルに中身を手で統合する。

- `README.md`
- `AGENTS.md`
- `CLAUDE.md`
- `.vscode/settings.json`
- `.gitignore`
- `.gitattributes`

pre-commit や husky などですでにフックを管理している場合は、`.githooks/` と `core.hooksPath` を使わず、既存のフックに `npm run check` を追加する。

### 更新の反映

1. 導入済みのプロジェクトの `.openspec-template` で、現在の版を確認する。
2. 共通のファイルを上書きし、一部共通のファイルは共通部分を置き換える。
3. 雛形は、更新履歴と差分を確認して手で反映する。
4. 全体検証を実行し、成功を確認する。

共通、一部共通、雛形の区別は、[内容](#内容)の表にある。

## 使い方

導入した後の作業は、AI に依頼して進める。\
AI は AGENTS.md に従い、change を作るか直接変更するかを判断する。

- 振る舞いを変える場合: ff、apply、verify、archive の順に進める。\
  要件が曖昧な場合は、ff の前に explore で整理する。
- 振る舞いを変えない場合(文書、リファクタリング、既存のシナリオどおりに動かないバグの修正など): change を作らずに直接変更する。

人が行うのは、次の3つである。

1. ff の後に、proposal とシナリオをレビューし、未決事項に答える。
2. verify の指摘を確認し、対応を決める。
3. 必要に応じて、manual-checks.md の実機確認を行い、結果を記録する。

何をどこに書くかは、次のとおりである。

| 書くもの | 置き場所 |
| --- | --- |
| プロダクトの目的、範囲、技術や構成の方針 | `openspec/product.md`、`tech.md`、`structure.md` |
| 要件とシナリオ | `openspec/specs/`(change の差分仕様から archive で反映する) |
| 今回の変更の理由、設計、作業 | `openspec/changes/<変更名>/` の proposal、design、tasks |
| 実機での確認 | `openspec/manual-checks.md` |

詳しい規則は、AI 向けの指示である AGENTS.md と `openspec/config.yaml`、成果物の作り方である `openspec/schemas/ears-bdd/schema.yaml`、検査の内容である [docs/openspec-trace-spec.md](docs/openspec-trace-spec.md) にある。

### OpenSpec の部分を切り離して納品する場合

OpenSpec の部分を含めずに納品したい場合は、`.gitattributes` の `export-ignore` で、OpenSpec を使った開発のためのファイルを納品物から除く。\
OpenSpec の部分も含めて納品する場合は、この手順は不要である。\
除くのは、`openspec/`、`AGENTS.md`、`CLAUDE.md`、AI ツールのスキル、`.githooks/`、検査スクリプト、`package.json` などである。\
コミットした状態から、次のコマンドで納品物を作る。

```sh
git archive HEAD -o <納品物の名前>.zip
```

作った後に、納品物の側で次を直す。

- README.md の "開発" の節を削除する。
- テストに書いたシナリオIDは、動作に影響しないので残してよい。\
  渡さない仕様書の番号が気になる場合は、納品物の側で消す。
- `.gitattributes` と `.gitignore` に OpenSpec のための行が残るが、動作に影響しないので残してよい。

TypeScript など、プロダクトでも `package.json` を使う場合は、`package.json` と `package-lock.json` を除かない。\
`.gitattributes` からこの2行を消しておき、納品物の側で `devDependencies` の OpenSpec と `check` を消して、`npm install` で `package-lock.json` を作り直す。

最後に、納品物を展開し、README.md のセットアップの手順でプロダクトの検査が成功することを確かめる。

## 内容

### このリポジトリ

| パス | 内容 |
| --- | --- |
| `template/` | プロジェクトにコピーする中身 |
| `.gitattributes` | `template/.githooks/` の改行を LF に固定する設定 |

### template/ の中身

| パス(template/ から) | 種類 | 内容 |
| --- | --- | --- |
| `AGENTS.md` | 共通 | すべての作業に共通する AI 向けの指示 |
| `CLAUDE.md` | 共通 | Claude Code に AGENTS.md を読み込ませる1行 |
| `.openspec-template` | 共通 | 導入したテンプレートの版 |
| `.githooks/pre-commit` | 共通 | コミット前に全体検証を実行するフック |
| `openspec/schemas/ears-bdd/` | 共通 | 共通 schema とテンプレート |
| `scripts/openspec-trace.mjs` | 共通 | トレース検査、ID 採番、台帳生成のスクリプト |
| `openspec/config.yaml` | 一部共通 | rules 以外は共通、rules はプロジェクト固有 |
| `.vscode/settings.json` | 雛形 | Copilot に AGENTS.md を読ませる設定 |
| `openspec/product.md` | 雛形 | プロダクトの前提 |
| `openspec/tech.md` | 雛形 | 技術の前提 |
| `openspec/structure.md` | 雛形 | 構成の前提 |
| `openspec/manual-checks.md` | 雛形 | 実機確認の管理表 |
| `openspec/specs/README.md` | 雛形 | 仕様書の一覧 |
| `README.md` | 雛形 | プロジェクトの説明とセットアップ(開発の節は、OpenSpec を切り離して納品する場合に消す) |
| `.gitignore` | 雛形 | テンプレートが使うもののうち、git で管理しないもの |
| `.gitattributes` | 雛形 | フックの改行の固定と、OpenSpec を切り離して納品する場合に除くファイルの設定 |

- 共通: プロジェクトで編集しない。\
  テンプレートの更新時は上書きする。
- 一部共通: コメントで示した共通部分だけを、テンプレートの更新時に置き換える。
- 雛形: コピーした後にプロジェクトで編集する。\
  テンプレートの更新時は、差分を確認して手で反映する。
