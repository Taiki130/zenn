---
title: "dyffとgithub-commentでkustomizeの差分をコメントするGitHub Actions workflowを作成しました"
emoji: "👓"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [kustomize, kubernetes, githubcomment, githubactions, aqua]
published: true
---

GitHubでKustomizeの差分を簡単に確認できるよう、**dyff**と**github-comment**を使って、Kustomizeリソースの変更内容を自動的にコメントするワークフローを作成しました。本記事では、このワークフローのセットアップと実行方法について解説します。

## 背景

Kustomizeを使っていると、リソースの差分をレビューするのが面倒になることがあります。特に、レビュー中に差分がGitHub上に表示されない場合、手作業で確認するのは効率が悪くなりがちです。
そこで、**dyff**で差分を確認し、PullRequestにコメントとして出力する方法を模索しました。これにより、レビュー効率が大幅に向上します。

## 使用ツールの紹介

### [**dyff**](https://github.com/homeport/dyff)

YAMLファイルの差分を視覚的に表示するツールで、Kustomizeリソースの差分確認に便利です。

### [**github-comment**:](https://github.com/suzuki-shunsuke/github-comment)

CLIを使ってGitHubのPull RequestやIssueにコメントを投稿できるツールです。

### [**aqua**](https://github.com/aquaproj/aqua)

CLIのバージョン管理ツールです。curl等でインストールすることもできるので、aquaが使えない方はそちらの方法で試していただけます。

## サンプルコード

```yaml:aqua.yaml
---
# aqua - Declarative CLI Version Manager
# https://aquaproj.github.io/
# checksum:
#   enabled: true
#   require_checksum: true
#   supported_envs:
#   - all
registries:
- type: standard
  ref: v4.239.0 # renovate: depName=aquaproj/aqua-registry
packages:
- name: suzuki-shunsuke/github-comment@v6.3.0
- name: homeport/dyff@v1.9.2
```

```yaml:.github/.github-comment.yaml
templates:
  diff_title: "## :white_check_mark: manifest diff ({{.Vars.app}})"
  diff_content: |
    {{template "link" .}}
    {{template "join_command" .}}
    Diff {{if eq .ExitCode 1}}found{{end}} :eyes:
    {{template "hidden_combined_output" .}}
  diff_content_for_too_long: |
    {{template "link" .}}
    {{template "join_command" .}}
    Diff {{if eq .ExitCode 0}}found{{else}}failed{{end}} :eyes:
    comment is too long
exec:
  diff:
    - when: ExitCode == 1
      template: |
        {{template "diff_title" .}}
        {{template "diff_content" .}}
      template_for_too_long: |
        {{template "diff_title" .}}
      embedded_var_names: [app]
hide:
  diff: 'Comment.HasMeta && Comment.Meta.TemplateKey == "diff" && Comment.Meta.Vars.app == Vars.app'
```

github-commentの設定ファイルで、コメントする条件と内容を設定します。今回実現したい内容としては、

- 差分がある場合のみコメント
- 新しいコミットが追加されるとコメント前にhideする

詳しくは[こちら](https://suzuki-shunsuke.github.io/github-comment/config)を参照してください。

```yaml:.github/workflows/kustomize_diff.yaml
name: Kustomize Diff
on:
  pull_request:
permissions: {}
jobs:
  test:
    runs-on: ubuntu-latest
    permissions:
      contents: read # checkout
      pull-requests: write # add comment using github-comment
    strategy:
      matrix:
        app:
          - service-a
          - service-b
    steps:
      - name: Checkout
        uses: actions/checkout@a5ac7e51b41094c92402da3b24376905380afc29 #v4.1.6
      - uses: actions/cache@0c45773b623bea8c8e75f6c82b208c3cf94ea4f9 # v4.0.2
        with:
          path: ~/.local/share/aquaproj-aqua
          key: v1-aqua-installer-${{runner.os}}-${{runner.arch}}-${{hashFiles('aqua.yaml')}}
          restore-keys: |
            v1-aqua-installer-${{runner.os}}-${{runner.arch}}-
      - uses: aquaproj/aqua-installer@6ce1f8848ec8e61f14d57bd5d7597057a6dd187c # v3.0.1
        with:
          aqua_version: v2.29.0
      - name: diff
        run: |
          git config --global url."https://${{ github.actor }}:${{ github.token }}@github.com".insteadOf ${{ github.server_url }}
          github-comment hide -k diff -pr ${{ github.event.pull_request.number }} --config .github/.github-comment.yaml -var 'app:${{ matrix.app }}'
          github-comment exec -k diff -pr ${{ github.event.pull_request.number }} --config .github/.github-comment.yaml -var 'app:${{ matrix.app }}' -- dyff --omit-header --set-exit-code bw <(kustomize build ${{ github.server_url }}/${{ github.repository }}//${{ env.PROD_DIR }}?ref=main) <(kustomize build ${{ env.PROD_DIR }})
        continue-on-error: true
        env:
          GITHUB_TOKEN: ${{ github.token }}
    env:
      PROD_DIR: overlays/prod/${{ matrix.app }}/
```

diffのところが肝心なので、解説します。

```yaml
github-comment hide -k diff -pr ${{ github.event.pull_request.number }} --config .github/.github-comment.yaml -var 'app:${{ matrix.app }}'
```

すでにPRに同じサービスのコメントがある場合はコメントをhideするようにしています。commitするたびに差分がコメントされるとかなり見づらくなってしまうためです。

`-var`で渡しているのは、hideの条件として同一サービスかどうか？を判断するために利用しています。

```yaml
github-comment exec -k diff -pr ${{ github.event.pull_request.number }} --config .github/.github-comment.yaml -var 'app:${{ matrix.app }}' -- dyff --omit-header --set-exit-code bw <(kustomize build ${{ github.server_url }}/${{ github.repository }}//${{ env.PROD_DIR }}?ref=main) <(kustomize build ${{ env.PROD_DIR }})
```

dyffの実行で、mainブランチとの差分がある場合にコメントします。`--omit-header`はdyff実行時に出力される不要なヘッダを削除。`—set-exit-code`は差分がある場合に異常終了するようにするオプションです。

異常終了している場合のみgithub-commentを実行することで、差分がある場合だけコメントする、という機構を実現してます。

## 余談

ローカルでマニフェストを書きながら確認したい場合もあると思います。そのため自分は以下のようなスクリプトを用意して確認できるようにしています。

```shell
#!/bin/bash
set -euC

function usage() {
  echo "Usage: $0 <env> <namespace>"
  echo "<env>: (prod|stg)"
  echo "<namespace>: (sercice-a|sercice-b|...)"
  exit 1
}

if [[ $# -ne 2 ]]; then
  usage
fi

declare -r env="$1"
declare -r namespace="$2"

echo "Diffing $env/$namespace"

GITHUB_TOKEN=$(gh auth token) \
  dyff bw <(kustomize build git@github.com/OWNER/REPO//"overlays/$env/$namespace"?ref=main) \
  <(kustomize build "overlays/$env/$namespace")
```

## おわりに

本記事では、**aqua**を活用して必要なツールをインストールし、**dyff**と**github-comment**でKustomizeリソースの差分をGitHubコメントに出力するワークフローを作成する方法を解説しました。Pull
Request上での差分確認が効率化され、レビューに役立つワークフローとなっています。ぜひご活用ください！
