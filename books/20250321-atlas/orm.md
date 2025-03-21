---
title: "AtlasとORMとの連携"
---

本章では、AtlasとORM（Object Relational Mapper）の連携方法を詳しく解説します。ORMを利用する開発現場でもAtlasを効果的に活用できます。

## GORMやentとの併用方法

GORMやentなどのORMツールを利用する際にも、Atlasを併用してマイグレーション管理が可能です。

### GORMとの併用例

GORMのモデル定義からスキーマを生成し、Atlasで差分管理をする流れの例です。

```go
package models

type User struct {
  ID    uint   `gorm:"primaryKey"`
  Name  string `gorm:"size:255;not null"`
  Email string `gorm:"size:255;unique;not null"`
}
```

このモデルからスキーマを生成:

```shell
# GORMでスキーマ生成（MySQLの例）
go run main.go auto-migrate
```

Atlasで差分検出・管理:

```shell
atlas migrate diff gorm_changes \
  --from "mysql://user:pass@localhost:3306/current_db" \
  --to "mysql://user:pass@localhost:3306/gorm_db" \
  --dir "file://migrations"
```

### entとの併用例

entのスキーマをAtlasのHCLに出力して管理する例です。

```shell
# entのスキーマをAtlasのHCL形式にエクスポート
go run entgo.io/ent/cmd/ent generate ./ent/schema
atlas schema inspect --url "sqlite://file?mode=memory&_fk=1" > schema.hcl
```

## ORMとAtlas間のスキーマ同期方法

ORMとAtlas間でスキーマの差異が出ないように、定期的にスキーマを同期するフローを設けます。

同期フロー例:

1. ORM側でスキーマ変更
2. ORMからスキーマを生成・更新したDBを作成
3. Atlasのdiff機能でORMのスキーマをHCL化
4. マイグレーションファイルとして管理し、Atlasで適用

これにより、常にスキーマを一元管理できます。

## ORM利用時の注意点とアンチパターン

ORMとAtlasを組み合わせる際の注意点を解説します。

- **ORMの自動マイグレーション依存の回避**  
  ORMの自動マイグレーション機能を本番環境で使うと意図しない変更が起きるリスクがあります。必ずAtlasで明示的な差分管理を推奨します。

- **スキーマ定義の重複管理を避ける**  
  ORMとAtlas両方でスキーマを手動管理すると差分の競合が起きやすいため、どちらかを主体として決めます。

- **過度な抽象化の回避**  
  ORM側で過度に抽象化されたスキーマを定義するとAtlasで管理しづらくなります。DBの制約などは明示的に定義することが推奨されます。

以上のポイントを守ることで、AtlasとORMの併用を効率的かつ安全に行えます。

次章では、実際の運用時のトラブルシューティングや安全なロールバック方法を解説します。
