---
title: "Atlasを使った差分検出と自動マイグレーション"
---

## 差分検出（diff）の基本と実践

AtlasはDBの現状スキーマとターゲットとなるスキーマの差分を自動的に検出し、必要なマイグレーションファイルを生成します。

差分検出を行う基本的なコマンドは次の通りです。

```shell
atlas migrate diff add_new_column \
  --from "mysql://user:pass@localhost:3306/current_db" \
  --to file://schema.hcl \
  --dir "file://migrations"
```

- `--from` は現在のデータベーススキーマの指定
- `--to` は新しいスキーマ定義（HCLファイルなど）の指定
- `--dir` はマイグレーションファイルの出力先ディレクトリの指定

これにより、Atlasがスキーマの差分を検出して、自動でマイグレーションファイルを作成します。

## マイグレーションファイルの自動生成手順

Atlasが生成したマイグレーションファイルは以下のようなフォーマットで保存されます。

```
migrations/
├── 20240321010101_add_new_column.sql
└── atlas.sum
```

`20240321010101_add_new_column.sql` は自動的に生成されたマイグレーションSQLファイルです。

内容例:

```sql
ALTER TABLE users ADD COLUMN age INT NULL;
```

`atlas.sum` はAtlasが生成する管理ファイルで、マイグレーションファイルの一貫性を管理します。

## マイグレーション運用の実践例

実際の運用では、以下のようなプロセスでマイグレーションを管理します。

### 1. HCLやSQLによるスキーマ定義の変更
チームメンバーが新しい機能開発やバグ修正に伴ってスキーマを変更します。

### 2. 差分検出によるマイグレーションファイル生成
変更内容をAtlasで検出し、自動でファイルを生成します。

```shell
atlas migrate diff add_feature_x \
  --from "mysql://user:pass@localhost:3306/current_db" \
  --to file://schema.hcl \
  --dir "file://migrations"
```

### 3. コードレビューとCI/CDパイプライン
生成されたファイルはGitにコミットし、Pull Requestでコードレビューします。  
レビュー後、CI/CDパイプラインを通じて自動的に適用されます。

実際のCI/CD適用例 (GitHub Actions):

```yaml
name: Atlas Migration

on:
  push:
    branches:
      - main

jobs:
  migrate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: ariga/setup-atlas@v0
      - run: |
          atlas migrate apply \
            --url "mysql://user:pass@prod-db:3306/production" \
            --dir "file://migrations"
```

これにより安全で確実なスキーマ変更を行い、開発サイクルの効率化を図ることが可能です。

次の章では、より実践的なCI/CDパイプライン構築について詳しく解説します。
