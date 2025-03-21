---
title: "高度なAtlas機能と応用"
---

本章では、Atlasの高度な使い方を紹介します。マイグレーションをさらに柔軟かつ自動化したいユーザー向けの内容です。

## カスタムテンプレートによるマイグレーションのカスタマイズ

Atlasでは、マイグレーションファイルの生成をカスタムテンプレートを使って自由に調整できます。  
独自のテンプレートを使うことでチームの規約やプロジェクト要件に合った形式を簡単に実現できます。

カスタムテンプレートの設定例：

```shell
atlas migrate diff add_feature \
  --from "mysql://user:pass@localhost:3306/db" \
  --to file://schema.hcl \
  --dir file://migrations \
  --format '{{ sql . "  " }} -- executed at {{ now }}'
```

- `--format`オプションでテンプレートを指定します。
- `.sql` は生成されるSQL内容、`{{ now }}`はファイル作成日時などの情報を組み込めます。

## Atlas SDKを活用した自動化方法

AtlasはGoのSDKを提供しており、スキーマ管理やマイグレーションをプログラム的に制御できます。  
自動化ツールやカスタムCLIの開発に便利です。

簡単なスキーマ検査のGoコード例：

```go
package main

import (
  "context"
  "fmt"
  "log"

  "ariga.io/atlas/sql/mysql"
  "ariga.io/atlas/sql/schema"
)

func main() {
  ctx := context.Background()
  dbURL := "mysql://user:pass@localhost:3306/mydb"

  driver, err := mysql.Open(ctx, dbURL)
  if err != nil {
    log.Fatal(err)
  }
  defer driver.Close()

  sch, err := driver.InspectSchema(ctx, "mydb", nil)
  if err != nil {
    log.Fatal(err)
  }

  schema.WalkTables(sch, func(t *schema.Table) error {
    fmt.Printf("Table: %s\n", t.Name)
    return nil
  })
}
```

これにより独自の運用ツールを構築したり、CI/CDとより深く統合したりできます。

## 安全なマイグレーション戦略

本番環境へのマイグレーションには慎重な戦略が求められます。以下の戦略を推奨します。

- **非破壊的（Non-destructive）マイグレーションの徹底**  
  テーブル削除やカラム削除を直接行わず、まずは非推奨（deprecated）にするフローを採用します。

- **ゼロダウンタイムでのマイグレーション**  
  新しいカラム追加 → アプリケーション更新 → 古いカラムの削除の順序で段階的に移行します。

- **マイグレーションのリハーサル**  
  本番DBと同じ構成のステージング環境で必ずリハーサルを実施し、影響範囲を事前に確認します。

- **差分監視とロールバック戦略の確立**  
  マイグレーション適用時のスキーマ差分監視、問題発生時の迅速なロールバック方法を明確にします。

次章ではAtlasを各種ORMと連携させる方法について解説します。
