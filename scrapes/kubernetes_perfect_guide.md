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

###
- ExternalName Service は、通常の Service リソースとは異なり、Service 名の名前解決に対して外部のドメイン宛の CNAME を返す
    - 使用する場面
        - 別の名前を設定したい場合
        - クラスタ内からのエンドポイントを切り替えやすくしたい場合

###
- None-Selector Service では、Service 名で名前解決を行うと自分で指定したメンバに対してロードバランシングを行う
- Kubernetes 内に自由な宛先でここまで説明した形式のロードバランサが作れる機能
    - 基本的には ClusterIP によるクラスタ内 ロードバランサで利用するケースが大半
- イメージ的にはクライアントサイドロードバランシング用のエンドポイントを提供するService
    

###
Kubernetes で環境変数を渡す際には、Pod テンプレートに env または envFrom を指定する
下記の 5 つの情報源から環境変数を埋め込むことが可能
- 静的設定
- Pod の情報
- コンテナの情報
- Secret リソースの機密情報
- ConfigMap リソースの設定値

### 
Secret を定義しているマニフェストは base64 でエンコードされているが、暗号化されているわけではないため、マニフェストを Git リポジトリ上などにアップロードすることはできない
Secret が定義されたマニフェストを暗号化する kubesec / SealedSecret / ExternalSecret といったオープンソースソフトウェアもある

###
一般的なスキーマレスの Secret を作成する場合は、type に Opaque を指定
- kubectl でファイルから値を参照して作成する(--from-file)
- kubectl で envfile から値を参照して作成する(--from-env-file)
- kubectl で直接値を渡して作成する(--from-literal)
- マニフェストから作成する(-f)

###
kubectl を使ってファイルから値を参照して作成する場合には、「--from-file」オプションを指定する
```
 kubectl create secret generic --save-config sample-db-auth \
--from-file=./username --from-file=./password

kubectl get secret sample-db-auth -o json | jq -r .data.username | base64 --decode root
```

###
一つのファイルから一括で作成する場合には、envfile から作成する方法もある
```
kubectl create secret generic --save-config sample-db-auth \
--from-env-file ./env-secret.txt
```

###
kubectl を使って直接値を渡して作成するには、「--from-literal」オプションを使って作成する
```
kubectl create secret generic --save-config sample-db-auth \
--from-literal=username=root --from-literal=password=rootpassword
```

###
マニフェストから作成する場合、base64 でエンコードした値をマニフェストに埋め込む

###
証明書として利用する Secret を作成する場合は、type に tls を指定
```
kubectl create secret tls --save-config tls-sample --key ~/tls.key --cert ~/tls.crt
```

###
Docker レジストリの認証用の Secret を作成する場合は、type に docker-registry を指定
```
kubectl create secret docker-registry --save-config sample-registry-auth \
--docker-server=REGISTRY_SERVER \ 
--docker-username=REGISTRY_USER \ 
--docker-password=REGISTRY_USER_PASSWORD \ 
--docker-email=REGISTRY_USER_EMAIL
```

###
Secret をコンテナから利用する場合、大きく分けて下記の 2 つのパターン
- 環境変数として渡す
    - Secret の特定の Key のみ
    - Secret のすべての Key
- Volume としてマウントする 
    - Secret の特定の Key のみ 
    - Secret のすべての Key

###
ConfigMap とは、設定情報などの Key-Value で保持できるデータを保存しておくリソース
nginx.conf や httpd.conf のような設定ファイル自体も保存可能

ConfigMap をコンテナから利用する場合にも、大きく分けて下記の 2 つのパターン
- 環境変数として渡す
    - ConfigMap の特定の Key のみ
    - ConfigMap のすべての Key
- Volume としてマウントする
    - ConfigMap の特定の Key のみ
    - ConfigMap のすべての Key

###
Secret と ConfigMapの大きな違いは、Secret が機密情報を扱うためのリソース
- Secret のデータは、Kubernetes Master が利用している分散 KVS(Key-Value Store)の etcd に保存されている
- 実際に Secret を利 用する Pod がある場合のみ、etcd から Kubernetes Node にデータを送る
- Kubernetes Node 上には永続的にデータが残らないように、Secret のデータは tmpfs 領域(メモリ上に構築される一時的なファイルシステム)に保持されるようになっている

###
- Volume はあらかじめ用意された利用可能なボリューム(ホストの領域/ NFS / iSCSI / Ceph)などを、マニフェストに直接指定することで利用可能にするもの
- PersistentVolume は、外部の永続ボリュームを提供するシステムと連携して、新規の ボリュームの作成や、既存のボリュームの削除などを行うことが可能
- PersistentVolumeClaim は、その名のとおり作成された PersistentVolume リソースの中からアサインするためのリソース

###
emptyDir は、Pod 用の一時的なディスク領域として利用可能
ホスト上の任意の領域をマウントできるわけではない
ホスト上にあるファイルを参照することもできない

###
hostPath は、Kubernetes Node 上の領域をコンテナにマッピングするプラグイン

###
downwardAPI は、Pod の情報などをファイルとして配置するためのプラグイン

###
projected は、Secret / ConfigMap / downwardAPI / serviceAccountToken のボリュームマウントを 1 箇所のディレクトリに集約するプラグイン

###
PersistentVolume の 種類がわからなくなってしまうため、type / environment / speed などのラベルをつけていくことが推奨

###
アクセスモード
- ReadWriteOnce(RWO)
    -  単一ノードから Read / Write が可能
- ReadOnlyMany(ROX)
    - 複数ノードから Read が可能
- ReadWriteMany(RWX)
    - 複数ノードから Read / Write が可能

###
Reclaim Policy は、PersistentVolume を利用し終わった後の処理方法(破棄するか、再利用するか など)を制御するポリシー
- Delete
    - PersistentVolume の実体(GCP の永続ディスクなど)が削除されます
    - GCP / AWS / OpenStack などで確保される外部ボリュームの Dynamic Provisioning 時に利用されることが多い
- Retain
    - PersistentVolume の実体(GCP の永続ディスクなど)を消さずに保持します
    - 他の PersistentVolumeClaim によって、この PersistentVolume が再度マウントされることはありません
-  Recycle(非推奨)
    - PersistentVolume のデータを削除(rm -rf ./*)し、再利用可能な状態にします
    - 他の PersistentVolumeClaim によって再度マウントされます
    - 将来の Kubernetes にて廃止が検討されているため、Dynamic Propvisioning を利用するようにしてください

###
Dynamic Provisioning を使った PersistentVolumeClaim の場合、PersistentVolumeClaim が発行されたタイミングで動的に PersistentVolume を作成して、割り当てる
- 事前に PersistentVolume を作成する必要がない
- 容量の無駄が生じない

###
StorageClass には volumeBindingMode の設定項目が用意されており、WaitForFirstConsumer を指定することで、実際に Pod にアタッチされる直前のタイミングで PersistentVolume が作成され、 紐づけられるようになる

###
StatefulSet でのワークロードでは、永続化されるデータ領域を利用することが多いため、spec.volumeClaimTemplate の項目がある
volumeClaimTemplate を利用すると、 PersistentVolumeClaim を別途定義する必要がなくなり、自動的に PersistentVolumeClaim を作成していくことが可能

###
subPath は Volume をマウントする際に、特定のディレクトリをルートとしてマウントする機能

###
ResourceQuota を利用することで各 Namespace ごとに、すなわち仮想 Kubernetes クラスタごと に利用可能なリソースを制限することが可能

###
HorizontalPodAutoscaler は、Deployment / ReplicaSet / ReplicationController のレプリカ数 を CPU 負荷などに応じて自動的にスケールさせるリソース
VerticalPodAutoscaler は、コンテ ナに割り当てる CPU /メモリのリソースの割り当てを自動的にスケールさせる機能

###
Liveness Probe は、Pod 内のコンテナが正常に動作していることを確認するためのヘルスチェック
Readiness Probe は、Pod がリクエストを受け付けることができるかを確認するためのヘルスチェック
Startup Probe は、Pod の初回起動が完了しているかどうかを確認するためのヘルスチェック

###
exec - コマンドを実行し、終了コードが 0 でなければ失敗
httpGetHTTP - GET リクエストを実行し、Status Code が 200~399 でなければ失敗 
TCP - セッションが確立できなければ失敗

###
Pod ready++ では、その Pod に対して ReadinessGate という追加のチェック項目 (spec.readinessGates)を設定する
ReadinessGate がパスするまでは、Service の転送先に追加されない
ローリングアップデート時には次の Pod の起動に移らない

###
ReadinessProbe が失敗して いる場合でも Service に紐づけるようにするには、spec.publishNotReadyAddresses を true に設定

###
ReplicaSet の削除を行った場合の挙動
- Background(デフォルト)
    -  ReplicaSet を直ちに削除し、Pod はガベージコレクタがバックグラウンドで非同期に削除
-  Foreground
    - ReplicaSetを直ちに削除はせず、deletionTimestamp,metadata.finalizers=foregroundDeletionに設定
    - ガベージコレクタが各 Pod で blockOwnerDeletion = true のものを削除する ・(false のものに関してはフォアグラウンド削除であってもバックグラウンドで削除) ・ すべて消し終わったあとに ReplicaSet を削除する
- Orphan
    - ReplicaSet 削除時に Pod の削除を行わない

###
ノードを SchedulingDisabled に変更し、スケジューラの候補から外す場合は「kubectl cordon」コ マンドを利用する
SchedulingEnabled に変更し、スケジューリングの候補に戻す場合は「kubectl uncordon」 コマンドを利用する

###
ノード上で実行しているすべての Pod を退避させる排出処理(drain)を行うには、「kubectl drain」コマンドを利用する

###
PodDisruptionBudget はノードが排出処理を行う際に、Pod を停止することのできる数を制限 するリソース
PodDisruptionBudget を設定 しておくことで、条件にマッチする Pod の最小の起動数(spec.minAvailable)または最大の停止数(spec.maxUnavailable)を見ながら、ノード上からの Pod の追い出しを行うことが可能になる

###
- nodeSelector(Simple Node Affinity) 
    - 簡易的な Node Affinity 機能
- Node Affinity 
    - 特定のノード上だけで実行する
- Node Anti-Affinity 
    - 特定のノード以外で実行する
- Inter-Pod Affinity　
    - 特定の Pod がいるドメイン(ノード、ゾーン、etc)上で実行す る
- Inter-Pod Anti-Affinity
    - 特定の Pod がいないドメイン(ノード、ゾーン、etc)上で実行する

###
Node Affinity における必須スケジューリングポリシーと優先スケジューリングポリシー
requiredDuringSchedulingIgnoredDuringExecution 必須のスケジューリングポリシー preferredDuringSchedulingIgnoredDuringExecution 優先的に考慮されるスケジューリングポリシー

###
TopologySpreadConstraints では、指定したラベルを持つ Pod が特定のトポロジに対してどの程度 の偏りの許容を設定することでスケジューリングの制約を設ける

###
Taints / Tolerations
対象のノードを特定の用途に向けた専用ノードにする場合などに利用する
- PreferNoSchedule
    - 可能な限りスケジューリングしない
- NoSchedule
    - スケジューリングしない(すでにスケジューリングされている Pod はそのまま)
- NoExecute
    - 実行を許可しない(すでにスケジューリングされている Pod は停止される)

###
ServiceAccount は Kubernetes の世界だけで完結するもので、Pod で実行されるプロセスのために割り当てるもの

###
imagePullSecrets が指定された ServiceAccount を割り当てた Pod が起動した場合、自動的に Pod の imagePullSecrets として利用する

###
RBAC(Role Based Access Control)は、どういった操作を許可するのかを定めた Role を作成し、 ServiceAccount などの User に対して Role を紐づける(RoleBinding)ことでその権限を付与する

Namespace レベルのリソースとして Role と RoleBinding
Clusterレベルのリソースとして ClusterRole と ClusterRoleBinding がある

###
SecurityContext は、個々のコンテナに対するセキュリティ設定(SecurityContext)

privileged 特権コンテナとして実行
capabilities Capabilities の追加と削除 
allowPrivilegeEscalation コンテナ実行時に親プロセス以上の権限を与えるかどうか 
readOnlyRootFilesystem root ファイルシステムを ReadOnly にするかどうか 
runAsUser 実行するユーザ
runAsGroup 実行するグループ
runAsNonRoot root での実行を拒否する
seLinuxOptions SELinux のオプション

###
PodSecurityContext は、Pod(すべてのコンテナ)に対するセキュリティ設定(SecurityContext)

runAsUser  実行するユーザ
runAsGroup  実行するグループ
runAsNonRoot root での実行を拒否する
supplementalGroups  プライマリ GID に追加で付与する GID のリストを指定 
fsGroup ファイルシステムのグループを指定 
sysctls  上書きするカーネルパラメータを指定(Kubernetes v1.11 以降) 
seLinuxOptions SELinux のオプションを指定

###
PodSecurityPolicy は、Kubernetes クラスタに対してセキュリティポリシーによる制限を行うリソース

###
NetworkPolicy は、Kubernetes クラスタ内で Pod 同士が通信する際のトラフィックルールを規定する

###
Admission Control は、Kubernetes API サーバにリクエスト制御を追加で行う仕組み
Kubernetes では Authentication / Authorization / Admission Control の 3 つのフェーズを通っ てリソースが登録される
Authentication/AuthN(認証)では、正しいユーザであるか どうかをユーザ名とパスワードやトークンといったもので確認する
Authorization/AuthZ(認可) では、そのユーザが行おうとしている要求について、実行可能な権限があるかの制御を行う
Admission Control フェーズでは、別途そのリクエストの内容を許可するかどうかの判断や、受け取ったリソースの改変などを行って登録するなどが可能になる

###
PodPreset は Admission Controller の一つで、Pod を作成する際にデフォルト設定を埋め込むリソース

- 特定のラベルが付与されている Pod に対しては、自動的に環境変数を埋め込む
- 特定のラベルが付与されている Pod に対しては、/var/log/領域に Persistent Volume を割り当
てる

Pod の定義と PodPreset により追加される定義が、どれか1つでも衝突する場合、PodPreset の書き換えは一切行われない

###
kubesec は、Kubernetes の Secret を安全に管理するためのオープンソースソフトウェア
GnuPG / Google Cloud KMS / AWS KMS を利用して Secret の暗号化を行うことが可能
ファイル全体を暗号化するのではなく、Secret の構造(data.*)を保ったまま値だけ暗号化するため、可読性が優れている
