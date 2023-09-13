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
```
# sample-podが正常に起動する(Ready状態になる)まで待機する
$ kubectl wait --for=condition=Ready pod/sample-pod
pod/sample-pod condition met
```
