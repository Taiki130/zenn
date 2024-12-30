---
title: "GitHub ActionsのSecretの値を後から安全に確認する方法"
emoji: "🫐"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["githubactions", "age"]
published: true
---

GitHub ActionsのSecretをTerraformでIaC化しようとして、値を参照しようと試行錯誤したときのメモです。その他のアプローチとして、Terraformでimportしたり、GitHub APIを使用してもSecretの値は確認できませんでした。

手順は以下の通りです。

1. 公開鍵暗号の鍵を作成
2. GitHub Actionsのworkflow上でsecretsを公開鍵で暗号化
3. ローカルで秘密鍵を使い復号

## 1. 公開鍵暗号の鍵を作成

GPG等何でも良いですが、今回は[age](https://github.com/FiloSottile/age)を使用しました。

```shell
% age-keygen -o key.txt
```

## 2. GitHub Actionsのworkflow上でsecretsを公開鍵で暗号化

適当にworkflowを作成し、commitします。

```yaml
name: Load Secrets
on:
  push:

jobs:
  load-secrets:
    runs-on: ubuntu-latest
    steps:
      - name: Setup age
        uses: AnimMouse/setup-age@v1
      - run: |
          echo "${{ toJson(secrets) }}" | age -r ${{ env.AGE_PUBKEY }} -e -o - | base64
        env:
          AGE_PUBKEY: <1で生成した公開鍵>
```

:::message alert
サードパーティアクションを使う場合は、問題ないかどうかソースコードを確認してください。
:::

## 3. ローカルで秘密鍵を使い復号

2のログに出力された値をローカルで復号します。

```shell
% echo '<2のログに出力された値>' | base64 --decode | age -d -i key.txt
```

これでsecretsにあるすべての値がJSONで出力されると思います。
