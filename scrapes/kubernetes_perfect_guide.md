kubernetes完全ガイド 読書メモ
###

リソース作成にも kubectl apply を使うべき理由
- 作成時には「kubectl create」を使い、更新時には「kubectl apply」を使うといった使い分けが不要になる
- 「kubectl create」と「kubectl apply」を混在して使用すると、「kubectl apply」実行時に差分を検出しきれないことがあるため

###
Server-sideapply
- 様々なシステムコンポーネントがリソースの フィールドを自動的に書き換えることがある
- Server-side apply を利用することでこれらの衝突を検知することが可能

### 
Pod の再起動
- Pod の起動時の処理 を再実行したい時
- Secret リソースで参照されている環境変数を更新したい時
```
% kubectl rollout restart deployment <name>
```

###
リソースの状態のチェックと待機(wait)
- kubectl コマンドを連続的に実行してリソースを操作していく際に、次のコマンドを実行する前にそ れまでに操作したリソースが意図する状態になってから次のコマンドを実行したいケース
- 「--for」オプションで指定する状態になるまで kubectl コマンドが最大「--timeout」オプションに指定する時間(デフォ ルト値は 30 秒)まで終了せずに待機し続ける
```
# sample-podが正常に起動する(Ready状態になる)まで待機する
$ kubectl wait --for=condition=Ready pod/sample-pod
pod/sample-pod condition met
```

###
マニフェストファイルの設計
*1 つのマニフェストファイルの中に複数のリソースを記述する*
- 典型的な利用事例としては、「Pod を起動する Workloads APIs カテゴリのリソース」と「外部公開 する Service APIs カテゴリのリソース」をマニフェストにまとめて記述するという使い方
- 共通で利用する設定ファイル(ConfigMap リソース)やパスワード(Secret リソース)な どは様々なリソースから利用される場合
    - こういった共通で利用されるリソース は別のマニフェストに分割するべき

*複数のマニフェストファイルを同時に適用する*
- 複数のマニフェストファイルを一度に適用するには、ディレクトリ配下に適用したい複数のマニフェストファイルを配置しておき、「kubectl apply」コマンドを実行する際にディレクトリを指定する

*マニフェストファイルの設計指針*
- 規模が大きくない場合
    - システム全体を構成するための すべてのマイクロサービスのマニフェストを 1 つのディレクトリにまとめる
- 1 つのディレクトリにファイルを配置しきれないほど巨大なシステムの場合
    - サブシステムや部署ごとにディレクトリを分ける
    - マイクロサービスごとにディレクトリを切るパターン
        - 可読性が高まるというメリットがある一方で、適用の順序制御が難しくなる

###
アノテーションは、主に下記の 3 種類の用途で利用されている
- システムコンポーネントのためにデータを保存する
- すべての環境では利用できない設定を行う
- 正式に組み込まれる前の機能の設定を行う

ラベルは主に下記の 2 種類の用途で利用されている
- 開発者が利用するラベル
- システムが利用するラベル

###
CI/CD パイプラインでは更新されたマニフェストに対して「kubectl apply --prune」コマンドを実行し続けるだけで、削除されたリソースも 自動的に削除することが可能になる

###
Pod定義を編集(環境変数EDITORに定義されたエディターが起動)
kubectl edit pod sample-pod

###
マニフェストファイルを更新せずとも、一部の設定値(また、この設定値を持つ特定のリソース)に 限り「kubectl set」コマンドを使用して簡単に動作状態を変更することが可能
```
kubectl set image リソース種類 リソース名 コンテナ名=コンテナイメージ指定(:コンテナイメージのタグ指定)
```

###
```
# クラスタの登録情報とマニフェストの差分を確認 
$ kubectl diff -f sample-pod.yaml
```
###
「kubectl debug」コマンドで、Pod に追加の一時的なコンテナ(Ephemeral Container) を起動し、そのコンテナを使ってデバッグやトラブルシューティングを行える

###
「kubectl port-forward」コマンドでkubectl を実行している手元のマシンから特定の Pod へトラフィックを転送する

###
- リソースなどを作成した際の HTTP Request / Response の概要を表示する場合は、「-v=6」以降のログレベルで出力
- Request Body / Response Body まで確認する場合は、「-v=8」以降のログレベルで出力

###
Pod が起動しない場合のデバッグ
- 「kubectl logs」コマンドを使って、コンテナが出力しているログを確認する
    - 主にアプリケーション側に問題がある場合
- 「kubectl describe」コマンドで表示される Events の項目を確認する
    - 主に Kubernetes の設定やリソースの設定に関して問題がある場合
- 「kubectl run」コマンドを使って、実際にコンテナのシェル上で確認する
    - 主にコンテナ内の環境やアプリケーションに問題がある場合

###
ホストのネットワークを利用する設定(spec.hostNetwork)を有効化することで、ホスト上でただプロセスを起動するのと同等のネットワーク構成(IP アドレス、DNS 設定、hosts 設定など)で Pod を起動させることが可能

###
deploymentで変更履歴を確認するには、「kubectl rollout history」コマンドを利用
該当リビジョンのより詳細な情報を取得するには、「--revision」オプションを指定
```
kubectl rollout history deployment sample-deployment --revision 1
```
ロールバックするには、「kubectl rollout undo」を利用
```
kubectl rollout undo deployment sample-deployment --to-revision 1
```
実際にはこのロールバック機能を使用するケースは多くない。CI/CD のパイプラインからロールバックする場合、「kubectl rollout」コマンドよりも、古いマニフェストを再度「kubectl apply」コマンドを実行して適用したほうが相性が良いから

###
StatefulSet では、spec.volumeClaimTemplates を指定することができる。これにより、StatefulSet によって作成される各 Pod に対して、PersistentVolumeClaim(PersistentVolume 要求)を設定できる

###
jobにおける重要なパラメータ
completions: 5 parallelism: 3 backoffLimit: 5
上記は3 並列で実行/成功数が 5 になったら終了/失敗数が 5 に なったら終了という意味

###
Job は終了後に削除されず残り続けてしまう
「spec.ttlSecondsAfterFinished」を設定することで、Job が終了後に一定秒数経過で削除されるように設定することが可能

###
Cronjobの一時停止
```
kubectl patch cronjob sample-cronjob -p '{"spec":{"suspend":true}}'
```

###
CronJob を任意のタイミングで実行
```
kubectl create job sample-job-from-cronjob --from cronjob/sample-cronjob
```

###
同時実行に関するポリシーは spec.concurrencyPolicy に指定
```
Allow(デフォルト) 同時実行に対して制限を行わない
Forbid 前の Job が終了していない場合、次の Job は実行しない(同時実行を行わない)
Replace 前の Job をキャンセルし、Job を開始する
```

###
Pod は Service を利用せずとも Pod 間通信を行うことが可能
しかし、Service を利用することによって得られる大きなメリットが 2つある
- Pod 宛トラフィックのロードバランシング
- サービスディスカバリとクラスタ内 DNS
