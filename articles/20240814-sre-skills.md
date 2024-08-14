---
title: "2024年版 SREに求められるスキルを募集要項から調査してみた"
emoji: "😺"
type: "idea" # tech: 技術記事 / idea: アイデア
topics: [sre, 採用]
published: true
---

# はじめに

SREに求められるスキルを調べていたら、以下の記事を見つけました。2024年現時点ではどのように変わっているのか知りたかったため、各社の募集要項からSREに求められるスキルを調べてみました。

[**各社の募集要項からSREに求められるスキルをまとめた**](https://qiita.com/mogulla3/items/7bf8db3b18980277db52)

Gitやチーム開発などの一般的にエンジニア求められるスキルは除いています。

# 調査した企業

Findyの求人でSREで絞り込んで人気上位5社を対象にしました。

- [株式会社IVRy](https://findy-code.io/companies/1610)
- [株式会社ROBOT PAYMENT](https://findy-code.io/companies/1673)
- [株式会社バニッシュ・スタンダード](https://findy-code.io/companies/533)
- [RIZAPグループ株式会社](https://findy-code.io/companies/885)
- [ファインディ株式会社](https://findy-code.io/companies/164)

# 求められているスキル

## パブリッククラウドに関する知見

AWSでの構築・運用が多そう。

> - AWS, GCP, Azure といったパブリッククラウドでのWebサービス開発・運用の実務経験 （株式会社IVRy/必須）
> - AWS, GCPなどのクラウドサービスを利用した開発経験(3年以上) （株式会社ROBOT PAYMENT/必須）
> - AWS環境の構築、運用経験(目安3年以上) （株式会社バニッシュ・スタンダード/必須）
> - AWS等クラウドインフラ構築／運用経験（目安：3年以上） （RIZAPグループ株式会社/必須）
> - AWSやGCPなどのパブリッククラウドサービスの利用経験 （ファインディ株式会社/必須）

## 障害検知やモニタリング環境の構築

> - 障害検知やキャパシティプランニングのためのモニタリング環境構築の経験 （株式会社IVRy/必須）
> - 障害検知やキャパシティプランニングの為のモニタリング環境構築／運用経験 （RIZAPグループ株式会社/歓迎）

## パフォーマンス改善やチューニング経験

> - アプリケーション、ミドルウェアのパフォーマンス改善、OS、各ミドルウェア、データベースの設定のチューニング経験 （株式会社IVRy/必須）
> - サービスのパフォーマンス改善の可視化／分析／提案経験 （RIZAPグループ株式会社/歓迎）
> - MySQL, Redis, Elasticsearch等のパフォーマンスチューニング経験 （ファインディ株式会社/歓迎）

## Infrastructure as Code（IaC）の経験

Terraformが多そう。

> - Terraform 等の IaCの経験 （株式会社IVRy/歓迎）
> - Terraformなどの構成管理ツールの業務利用経験 （株式会社ROBOT PAYMENT/歓迎）
> - Terraform, Ansible等の構成管理ツールを利用した構築／運用経験 （RIZAPグループ株式会社/必須）
> - IaC/terraform/Ansibleなどの活用経験 （株式会社バニッシュ・スタンダード/歓迎）
> - TerraformやCloudFormationなどのIaCに関する知識 （ファインディ株式会社/必須）

## コンテナ技術の経験

> - 本番環境でのECS、Fargate、Kubernates等を用いたDockerコンテナ環境の構築運用経験 （株式会社ROBOT PAYMENT/歓迎）
> - Dockerを用いたサーバ環境設計／構築経験 （RIZAPグループ株式会社/歓迎）
> - Dockerなどのコンテナに関する知識 （ファインディ株式会社/必須）

## Webサービス開発・運用経験

> - Webアプリケーションサービス開発や運用に関わる業務経験(3年以上) （株式会社ROBOT PAYMENT/必須）
> - Webサービスのフロントエンドおよびバックエンドの開発・運用経験 （ファインディ株式会社/歓迎）

## 大規模Webサービスの構築・運用経験

> - 大規模Webサービスの運用経験 （株式会社ROBOT PAYMENT/歓迎）
> - 大規模Webサービスの構築、運用経験 （RIZAPグループ株式会社/歓迎）

## デプロイやCIの改善・自動化経験

> - デプロイ改善、開発環境改善、CI改善などの開発業務効率化 （株式会社IVRy/歓迎）
> - デプロイや各種オペレーション自動化ツールの開発／運用経験 （RIZAPグループ株式会社/歓迎）
> - CI環境や自動化テストによる品質保証に取り組んだ経験 （ファインディ株式会社/歓迎）

## SREに関する知識

> - SREやDevOpsに関する知識 （株式会社ROBOT PAYMENT/歓迎）
> - SRE組織でのマネジメント/リーダー経験 （株式会社バニッシュ・スタンダード/歓迎）

# 最後に

パフォーマンス改善やチューニング経験、Webサービス開発・運用経験が求められている点から、
大前提としてコンピュータサイエンスの基礎知識・コーディングスキルなど求められているように思えます。

求められているスキルは2016年時点とそこまで変わってない印象を受けました。

# 参考
- https://findy-code.io/companies/1610/jobs/Cx0gdkU6Vd_x7
- https://findy-code.io/companies/1673/jobs/JcXGoJBzkmIY_
- https://findy-code.io/companies/533/jobs/xEVYXi_HG5NaD
- https://findy-code.io/companies/885/jobs/0UM_csce9F0ss
- https://findy-code.io/companies/164/jobs/flUdVucXpfxcu
