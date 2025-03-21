---
title: "スキーマ管理とマイグレーションの基本"
---

## HCLおよびSQLを使ったスキーマ定義方法

Atlasではスキーマ定義をSQLだけでなく、HCL (HashiCorp Configuration Language) でも行うことができます。HCLを使うとコードとしてスキーマを管理できるため、スキーマ変更が明確で追いやすくなります。

### SQLでの定義例
以下は、SQLによる簡単なスキーマ定義例です。

```sql
CREATE TABLE users (
  id INT PRIMARY KEY AUTO_INCREMENT,
  name VARCHAR(255) NOT NULL,
  email VARCHAR(255) UNIQUE NOT NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

### HCLでの定義例
同じスキーマをHCLで書くと以下のようになります。

```hcl
table "users" {
  schema = schema.main
  column "id" {
    type = int
    auto_increment = true
  }
  column "name" {
    type = varchar(255)
    null = false
  }
  column "email" {
    type = varchar(255)
    null = false
    unique = true
  }
  column "created_at" {
    type = timestamp
    default = sql("CURRENT_TIMESTAMP")
  }
  primary_key {
    columns = ["id"]
  }
}
```

HCLを使うことで、スキーマ定義を構造的かつ直感的に管理できます。

## マイグレーションファイルの作成と適用手順
Atlasでマイグレーションファイルを作成する基本的な手順は以下の通りです。

1. スキーマ差分検出とマイグレーションファイルの生成

```shell
atlas migrate diff create_users \
  --from "mysql://user:pass@localhost:3306/mydb" \
  --to file://schema.hcl \
  --dir "file://migrations"
```

これによりmigrationsディレクトリに差分をマイグレーションファイルとして生成します。

2. マイグレーションの適用
生成したマイグレーションファイルをDBに適用します。

```shell
atlas migrate apply \
  --url "mysql://user:pass@localhost:3306/mydb" \
  --dir "file://migrations"
```

これでマイグレーションが実行されます。

## 実際のスキーマ運用例
実践的なスキーマ運用の例として、以下のようなフローが推奨されます。

### 開発フロー例
1. HCLまたはSQLでスキーマ定義を変更
2. Atlas CLIで差分検出を実行してマイグレーションファイルを生成
3. マイグレーションファイルをGitにコミットし、Pull Requestを作成
4. レビュー後、マージしてCI/CDで自動的にDBに適用
5. これにより、チームで安全かつ効率的にスキーマを管理できます。

次章では、より詳細な差分検出や自動マイグレーションのテクニックを紹介します。
