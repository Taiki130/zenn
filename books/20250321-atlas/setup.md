---
title: "Atlasのセットアップと導入"
---

この章では、Atlasを利用するための環境構築と基本的な操作方法を紹介します。

## Atlas CLIのインストール手順

Atlas CLIは、公式ページやパッケージマネージャを使って簡単にインストールできます。

### Mac（Homebrew）の場合

```shell
brew install ariga/tap/atlas
```

### LinuxやWindowsの場合
公式リリースページからバイナリをダウンロードし、パスの通った場所に配置します。

- [Releases · ariga/atlas](https://github.com/ariga/atlas/releases)

例（Linux）:

```shell
curl -LO https://release.ariga.io/atlas/atlas-linux-amd64-latest
chmod +x atlas-linux-amd64-latest
sudo mv atlas-linux-amd64-latest /usr/local/bin/atlas
```

インストール後、CLIが正しく動作するか確認します。

```shell
atlas version
```

## Dockerを使用したAtlas環境構築

Dockerを使うことでAtlasを簡単に利用できます。

DockerイメージをPullします。

```shell
docker pull arigaio/atlas:latest
```

Dockerコンテナ内でAtlasを実行する方法:

```shell
docker run --rm arigaio/atlas:latest version
```

ローカルスキーマをDockerで差分検出する例:

```shell
docker run --rm -v $(pwd):/workspace arigaio/atlas:latest migrate diff example \
  --from "mysql://user:pass@host:port/dbname" \
  --to file:///workspace/schema.hcl \
  --dir file:///workspace/migrations
```

## 基本的なAtlasのコマンド体系

Atlas CLIには、主に以下のようなコマンドがあります。

- `atlas migrate diff`  
  DBスキーマ間の差分を検出し、マイグレーションファイルを生成します。

- `atlas migrate apply`  
  作成したマイグレーションファイルをDBに適用します。

- `atlas schema inspect`  
  現在のDBスキーマをHCLやSQL形式で出力します。

具体的な利用例:

```shell
# 差分を生成
atlas migrate diff init \
  --from "mysql://user:pass@localhost:3306/mydb" \
  --to file://schema.hcl \
  --dir "file://migrations"

# マイグレーション適用
atlas migrate apply \
  --url "mysql://user:pass@localhost:3306/mydb" \
  --dir "file://migrations"

# DBスキーマを検査
atlas schema inspect \
  --url "mysql://user:pass@localhost:3306/mydb"
```

以上でAtlasの基本的なセットアップと操作が完了します。次章では、具体的なスキーマ管理とマイグレーションの基本を学んでいきましょう。
