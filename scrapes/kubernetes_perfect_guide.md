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
**1 つのマニフェストファイルの中に複数のリソースを記述する**
- 典型的な利用事例としては、「Pod を起動する Workloads APIs カテゴリのリソース」と「外部公開 する Service APIs カテゴリのリソース」をマニフェストにまとめて記述するという使い方
- 共通で利用する設定ファイル(ConfigMap リソース)やパスワード(Secret リソース)な どは様々なリソースから利用される場合
    - こういった共通で利用されるリソース は別のマニフェストに分割するべき

**複数のマニフェストファイルを同時に適用する**
- 複数のマニフェストファイルを一度に適用するには、ディレクトリ配下に適用したい複数のマニフェストファイルを配置しておき、「kubectl apply」コマンドを実行する際にディレクトリを指定する

**マニフェストファイルの設計指針**
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

###
ClusterIP はクラスタ内でのみ利用可能な仮想 IP を持つエンドポイントを提供するロードバランサを構成

一つの Service に複数の Port を割り当てることも可能

###
サービスディスカバリ
Kubernetes の文脈では、Service に属する Pod を列挙したり、Service の名前からエンドポイントの情報を返すことを指す
主に次の 3 種類が用意されている
- 環境変数を利用したサービスディスカバリ
    - Pod 内からは、環境変数でも同じ Namespace のサービスが確認できるようになっている
- DNS A レコードを利用したサービスディスカバリ
    - 基本的に自動的に払い出された IP アドレスに紐づく DNS 名を使用するのが望ましい
    - 実際に登録されている正式な FQDN は、`[Service 名].[Namespace 名].svc.cluster.local` となってる
    - 同一 Namespace である場合には、「sample-clusterip」のように Service 名だけで名前解決ができる
    - 異なる Namespace の場合には「sample-clusterip.default」のように Namespace 名をつけて名前解決 を行う
- DNS SRV レコードを利用したサービスディスカバリ
    - SRV レコードは、Port 名と Protocol を利用することで、サービスを 提供している Port 番号を含めたエンドポイントを DNS で解決する仕組み
    - `[_Service の Port 名].[_Protocol].[Service 名].[Namespace 名].svc.cluster.local`

###
通常クラスタ内 DNS に対して名前解決のリクエストを送りますが、大規模なクラスタでのパフォー マンスを向上させるために各ノードのローカル上に DNS キャッシュサーバを用意する仕組みがある

###
「type: ClusterIP」を指定することで、ClusterIP Service を作成することが可能
spec.ports[].port には ClusterIP で受け付ける Port 番号を、spec.ports[].targetPort は転送先の コンテナの Port 番号を指定

###
- ExternalIP Service は、特定の Kubernetes Node の IP アドレス:Port で受信したトラフィックを、 コンテナに転送する形で外部疎通性を確立する Service
    - 特別な事情がない場合は、 ExternalIP Service の代わりに後述する NodePort Service の利用を検討する
- ExternalIP は「type: ExternalIP」を指定しない
- spec.externalIPs には Kubernetes Node の IP(10.240.0.7 など)を、
spec.ports[].port には Kubernetes Node の IP および ClusterIP で受け付ける Port 番号を、
spec.ports[].targetPort は転送先のコンテナの Port 番号を指定
- ExternalIP に利用可能な IP アドレスは、ノードの情報から確認することが可能

```
kubectl get nodes -o custom-columns="NAME:{metadata.name},IP:{status.addresses[].address}"
```

###
NodePort は、すべての Kubernetes Node の IP アドレス:Port で受信したトラフィックをコンテナに転送する形で、外部疎通性を確立する
- spec.ports[].port には ClusterIP で受け付ける Port 番号
- spec.ports[].targetPort は転送先のコンテナの Port 番号
- spec.ports[].nodePort には 全 Kubernetes Node で受け付ける Port 番号

###
LoadBalancer Service は、外部のロードバランサを利用するため、Kubernetes クラスタが構築されているインフラがこの仕組みに対応している必要がある

###
Service にはセッションアフィニティを有効化することができる

###
NodePort Service と LoadBalancer Service では Kubernetes Node 上に到達したリクエストは、 さらにノードをまたいだ Pod へもロードバランシングされるようになっており、不必要な二段階ロードバランシングが行われてしまう
- 均一にリクエストが分散しやすい
- 不要なレイテンシのオーバーヘッドが生じてしまう
- バランシングが行われる際に NAT されるため送信元 IP アドレスが消失する

spec.externalTrafficPolicy が Cluster（デフォルト） の場合には、LoadBalancer Service と NodePort Service の 両方とも外部からそのノードに到達したリクエストは、3 つの Pod にほぼ均等に転送される
spec.externalTrafficPolicy が Local の場合には、LoadBalancer Service と NodePort Service の両方とも外部からそのノードに到達したリクエストは、そのノード上にある Pod にのみ転送される

###
externalTrafficPolicy を利用することでノード間の通信を防ぐことが可能なため、一部同等の問題 を解決することが可能ですが下記のような問題が存在する
- ClusterIP では利用できない
- 同一ノード内にしか転送範囲を絞ることができない
- 同一ノードに Pod が起動していない場合はタイムアウト待ちになる

Topology-aware Service Routing にはどのトポロジの範囲に転送を試みるかを優先度順に指定していく
例えば、第一優先度「同一ノード」、第二優先度「同一ゾーン」、第三優先度「いずれかの Pod」といったように設定する
「kubernetes.io/hostname」 「topology.kubernetes.io/zone」「topology.kubernetes.io/region」

###
Headless Service は、対象となる個々の Pod の IP アドレスが直接返ってくる Service
Headless Service を作成するには、下記の 2 つの条件を満たす必要がある
- Service の spec.type が ClusterIP であること
- Service の spec.clusterIP が None であること

StatefulSetと組み合わせて利用する場合、特定の条件下で Pod 名で名前解決を行うこともできる
- [オプション] Service の metadata.name が StatefulSet の spec.serviceName と同じであること

