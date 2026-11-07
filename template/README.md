# {{プロジェクト名}}

{{このリポジトリが何かを1行で}}

## セットアップ

### 必要なもの

- {{言語の実行環境と道具}}

### 手順

1. `{{依存を入れるコマンド}}` で、依存を入れる。
2. `{{プロダクトの検査のコマンド}}` を実行し、成功することを確かめる。

## 使い方

{{利用者向けの使い方}}

## 開発

この節は、OpenSpec を使った開発のためにものである。\
OpenSpec の部分を切り離して納品する場合は、納品物からこの説を削除する。

### 開発に必要なもの

- Node.js 22.12 以上と npm(OpenSpec CLI と検査スクリプトに使う)
- git(コミット前のフックで全体検証を実行する)

### OpenSpec のグローバル設定

OpenSpec が生成するワークフローは、マシンごとのグローバル設定で決まる。\
生成されるファイルを全員で一致させるため、各マシンで最初に1回だけ次を実行する。

```sh
npx --no-install openspec config set profile custom
npx --no-install openspec config set workflows '["explore", "ff", "apply", "update", "verify", "sync", "archive"]'
npx --no-install openspec config set delivery skills
```

Windows PowerShell 5.1 では、2行目が `expected array, received string` で失敗することがある。\
その場合は `npx --no-install openspec config profile` を実行し、対話の画面で上と同じワークフローと配信方法(skills)を選ぶ。\
設定は `npx --no-install openspec config list` で確認できる。

### 開発の準備

1. セットアップの手順で、プロダクトの依存を入れる。
2. `npm install` で、OpenSpec を入れる。
3. OpenSpec のグローバル設定を行う。
4. `git config core.hooksPath .githooks` で、コミット前のフックを有効にする。
5. `npm run check` で、全体検証が成功することを確かめる。

全体検証は、プロダクトの検査と OpenSpec の検査をまとめて実行し、コミットの前にフックで自動で実行される。\
AI ツール向けの生成されたファイル(`.github/skills/`、`.claude/skills/`、`.agents/skills/`のうち、使うツールのもの)はコミットし、手で編集しない。\
OpenSpec を更新した後は、`npx --no-install openspec update` で生成し直す。

### 参照する文章

- プロダクトの目的と範囲: [product.md](openspec/product.md)
- 開発の進め方: [AGENTS.md](AGENTS.md)
- 仕様書の一覧: [openspec/specs/README.md](openspec/specs/README.md)